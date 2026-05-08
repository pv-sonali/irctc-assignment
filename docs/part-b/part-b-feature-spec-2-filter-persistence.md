# Feature Spec 2: Smart Filter Persistence & Client-State Caching

## Problem Statement

Search filters on IRCTC exhibit inconsistent behavior. Users select criteria (train type, departure time, duration) but filters either don't apply, reset unexpectedly, or display cached results from previous searches. The fundamental issue: filters are applied client-side while results are served from a server-side cache not invalidated when filter selections change. Users must retry filters 2-4 times per search cycle, wasting 15-30 minutes daily for heavy users.

**Impact from Part A**: 60% of filter operations fail on first attempt; 30% of searchers affected; more severe on 3G/4G networks.

---

## Proposed Solution

Implement client-side filter state management with cache invalidation. When a user applies a filter, the client:
1. Stores filter state in browser memory (Redux/Context API)
2. Sends new filter parameters to backend
3. Backend returns fresh results (bypasses stale cache)
4. Client caches results locally with filter fingerprint
5. Subsequent searches reuse cached results **only if filters haven't changed**

Additionally, add a "Save Filter Set" feature so users can name and reuse filter combinations (e.g., "AC Sleeper Early Morning").

### User Experience (Proposed)

```
Current (Broken) Filter Flow:
─────────────────────────────
User selects "AC Only" → Backend returns 45 trains (old cache)
User re-selects "AC Only" → Backend returns 12 trains ✓
User adds "Depart 7-10 PM" → Shows trains from PREVIOUS search (still cached)

Proposed (Fixed) Filter Flow:
─────────────────────────────
[Search for Mumbai→Delhi]
    ↓
[Results: 45 trains shown]
    ↓
Apply Filter: "AC Only"
    ↓ (Client detects filter state change)
Send: {route, filters: {coach: "AC"}} → Backend ignores cache, queries DB
    ↓
[Results: 12 AC trains]  ← Fresh results, not cached
    ↓
Apply Filter: "Depart 7-10 PM"
    ↓ (Client detects second filter added)
Send: {route, filters: {coach: "AC", departTime: "19:00-22:00"}} → New query
    ↓
[Results: 4 trains matching both filters] ← Correct results
    ↓
Clear "Depart 7-10 PM" filter
    ↓
[Results: 12 AC trains] ← Retrieved from cache (filter fingerprint matches)
```

---

## Technical Implementation Plan

### Frontend State Management

**Redux Store Structure:**
```javascript
{
  search: {
    currentSearch: {
      route: {from: "Mumbai", to: "Delhi"},
      date: "2026-05-10",
      filterFingerprint: "coach:AC|speed:express|duration:<20h", // Hash of active filters
      appliedFilters: {
        coach: ["AC"],
        speed: ["express"],
        duration: {max: 20},
      }
    },
    resultCache: {
      "route:Mumbai-Delhi|date:2026-05-10|filters:coach:AC": {
        trains: [...],
        timestamp: 1620000000,
        expiresAt: 1620003600, // 1-hour TTL
      }
    },
    savedFilterSets: [ // NEW: Save filter presets
      {id: 1, name: "AC Sleeper Early Morning", filters: {...}},
      {id: 2, name: "Non-AC Budget", filters: {...}},
    ]
  }
}
```

**Cache Invalidation Logic:**
```javascript
function updateFilter(newFilter) {
  const oldFingerprint = generateFilterHash(state.appliedFilters);
  const updatedFilters = {...state.appliedFilters, ...newFilter};
  const newFingerprint = generateFilterHash(updatedFilters);
  
  // If fingerprint changed, invalidate cache for this search
  if (oldFingerprint !== newFingerprint) {
    invalidateResultCache(currentRoute, newFingerprint);
    fetchNewResults(updatedFilters); // Bypass cache
  }
  
  dispatch(setAppliedFilters(updatedFilters));
}
```

**New Components:**
- `FilterPanel.jsx` - Checkboxes/toggles for coach, speed, duration, price
- `SaveFilterModal.jsx` - Save current filter set with name
- `SavedFilterPresets.jsx` - Quick access to saved filters
- `FilterStatusIndicator.jsx` - Shows active filters with removable tags

### Backend Changes

**Cache Busting Strategy:**
```javascript
// Before: Always check cache first (WRONG)
GET /search/trains → Check cache(route, date) → Return cached results

// After: Check filter parameters (CORRECT)
GET /search/trains?filters=coach:AC,speed:express
  → Compute filter hash
  → Check cache(route, date, filterHash) 
  → If miss: Query DB with filters
  → Cache result with filterHash
  → Return results
```

**New Query Endpoint:**
```
GET /search/trains
  ?from=Mumbai
  &to=Delhi
  &date=2026-05-10
  &filters=coach:AC|duration:<20h|speed:express
  &bypassCache=true (optional, for force-refresh)

Response:
{
  results: [...],
  filterApplied: {coach: "AC", duration: {max: 20}, speed: "express"},
  cacheHit: false,
  resultCount: 4,
  timestamp: "2026-05-08T10:30:00Z"
}
```

### API Changes

| Method | Endpoint | Purpose | Request | Response |
|--------|----------|---------|---------|----------|
| GET | `/search/trains` | Search with filters | `?from=x&to=y&date=z&filters=x:y` | `{results[], filterApplied, cacheHit}` |
| POST | `/filters/save-preset` | Save filter set | `{name, filters}` | `{presetId, name}` |
| GET | `/filters/my-presets` | Get saved filter sets | - | `{presets: [...]}` |
| DELETE | `/filters/preset/:id` | Delete saved preset | - | `{success}` |
| POST | `/search/apply-preset/:id` | Use saved filter set | - | `{results[], filters}` |

### Database Schema

**New Table: `user_filter_presets`**
```sql
CREATE TABLE user_filter_presets (
  preset_id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT NOT NULL,
  name VARCHAR(100),
  filters JSON, -- {coach: ["AC"], speed: ["express"], duration: {...}}
  usage_count INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  last_used_at TIMESTAMP NULL,
  FOREIGN KEY (user_id) REFERENCES users(user_id),
  INDEX idx_user (user_id)
);
```

**Modified Table: `search_cache`**
```sql
ALTER TABLE search_cache ADD COLUMN 
  filter_hash VARCHAR(255), -- Hash of applied filters
  UNIQUE INDEX idx_route_date_filters (route_id, search_date, filter_hash);
```

### Third-Party Services

- **None required** (client-side caching uses browser storage)

### Client-Side Caching Strategy

**Storage**: IndexedDB (5-50MB limit on most browsers)
- Store: Last 100 search results with filter hash
- TTL: 1 hour per cached result
- Eviction: LRU (Least Recently Used) when capacity exceeded

**Fallback**: If IndexedDB unavailable → LocalStorage (simpler, smaller 5MB limit)

---

## Success Metrics

| Metric | Current | Target | Measurement |
|--------|---------|--------|------------|
| Filter success rate (first attempt) | 40% | 95% | % of filter applications that show correct results |
| User retry count per search | 2-4 times | 1 time | Session logs, filter event tracking |
| Time spent in search flow | 2-5 min | <1 min | User session duration analytics |
| Filter preset adoption | N/A (new feature) | 20% of power users | Usage tracking |
| Mobile filter success rate | 20% (poor) | 90% | Mobile-specific filter event logs |
| Cache hit rate | 0% (broken) | 60% (useful) | Monitoring: cache_hits / total_requests |

---

## Edge Cases & Constraints

### Edge Cases

1. **User Applies 5+ Filters**: Performance degrades
   - **Handling**: Limit to 5 active filters; show warning after 3 filters
   - **Why**: Complex queries slow down backend; user cognitive load increases

2. **Cached Results Expire But User Hasn't Navigated Away**: Results are stale
   - **Handling**: Refresh cache silently in background; notify user if results changed
   - **TTL**: 1 hour for most searches; 30 min for Tatkal searches

3. **Filter Preset Contains Outdated Train Classes** (e.g., "2AC" class removed)
   - **Handling**: Validate preset against current train inventory; show warning if filters invalid
   - **Recovery**: Suggest alternative filter or save updated version

4. **Network Disconnection While Filter Applied**: User offline before results load
   - **Handling**: Show cached results from last successful search; indicate offline mode
   - **Notification**: "Using cached results. Some filters may not apply."

5. **User Saves Identical Filter Presets With Different Names**
   - **Handling**: Warn user: "You already saved this filter set as 'AC Sleeper Early Morning'. Proceed?"

### IRCTC-Specific Constraints

- **Filter Combinations**: Some filters are mutually exclusive (e.g., Tatkal + Advance booking)
  - Validate filter combinations before sending to backend
- **Train Class Inventory**: Changes dynamically; validate cached results against current inventory
- **Mobile Network**: Minimize payload size for filter requests (<5KB per request)
- **Filter Persistence Across Sessions**: Save filters in browser storage so user can resume search

---

## Rollout Plan

**Phase 1 (Week 1)**: Deploy new cache invalidation logic (backend only)
- No UI changes; filters work correctly but no visual improvements
- Monitor: cache hit/miss rates, backend query load
- Rollback trigger: >10% increase in backend errors

**Phase 2 (Week 2)**: Deploy client-side state management (frontend)
- Add Redux filter state management
- A/B test: 10% of users get new filter logic
- Measure: filter success rate improvement

**Phase 3 (Week 3)**: Deploy saved filter presets (new feature)
- 50% rollout; monitor adoption
- Enable for all users if adoption >5%

**Full Rollout**: Week 4 (100% of users)

---

## Related Problems Solved

- **Problem 2 (Filter Issues)**: ✅ Directly solves
- **Problem 5 (Mobile Form)**: Partial; faster search reduces time on form
