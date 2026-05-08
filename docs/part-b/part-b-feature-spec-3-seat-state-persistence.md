# Feature Spec 3: Session State Persistence for Seat Selection

## Problem Statement

Users select specific seats or berth preferences but selections are cleared when navigating between form pages. The booking form resets to default values (random seat assignment, standard berth) during multi-passenger bookings, especially on mobile. The root cause: session state is not persisted across page navigation; each page reload reinitializes the form with default values. This affects 70% of family bookings (2+ passengers) and 50% of mobile users.

**Impact from Part A**: 70% of multi-person bookings require re-selection; mobile users experience 50x more resets than desktop users; cumulative booking time loss of 2-3 minutes per affected user.

---

## Proposed Solution

Implement persistent session state with three-layer caching:

1. **Client-side Memory State** (Redux): Primary state for current session
2. **Browser Local Storage**: Fallback if page refreshes
3. **Server-side Session Storage**: Sync point for multi-device continuity

When a user selects seats, the selection is immediately:
- Stored in Redux (instant UI update)
- Saved to browser local storage (survives page refresh)
- Synced to server session (survives browser close/reopen)

Additionally, implement a "Session Recovery" feature that auto-restores seat selections if a page reload occurs unexpectedly.

### User Experience (Proposed)

```
Ideal Seat Selection Flow (Multi-Passenger Booking):

Step 1: Select Train
   ↓ [User chooses Delhi-Agra Express]
Step 2: Select Seats
   ↓ [User selects lower berth for Passenger 1 in Coach C]
   ↓ (Redux saves: {passenger1: {coach: "C", berth: "lower", position: "L1"}})
   ↓ (LocalStorage saves same)
   ↓ [User selects lower berth for Passenger 2 in Coach C (adjacent)]
   ↓ (Redux saves: {passenger1: {...}, passenger2: {coach: "C", berth: "lower", position: "L2"}})
   ↓ (LocalStorage saves)
   ↓ [User clicks "Next: Passenger Details"]
   ↓ (Page navigates, but session state is restored from localStorage)
Step 3: Passenger Details Form
   ↓ [Name, age, gender, ID fields appear]
   ↓ (Seat selections still visible in sidebar: "Coach C: L1, L2")
   ↓ [User enters passenger details]
   ↓ [User clicks "Next: Review Booking"]
Step 4: Review & Payment
   ↓ [Seats shown: "Coach C, Lower Berths 1-2"]
   ↓ [Passengers shown: "Raj Kumar, Age 45" + "Priya Kumar, Age 42"]
   ↓ [Payment processed]
   ↓ [Confirmation: "Booking Successful, PNR: 1234567"]

---

On Mobile (With Keyboard Interference):

Step 2: Select Seats (Mobile)
   ↓ [User taps lower berth for Passenger 1]
   ↓ (Redux + LocalStorage update immediately)
   ↓ [Virtual keyboard appears, covers lower half of screen]
   ↓ (State still preserved in Redux)
   ↓ [User dismisses keyboard]
   ↓ (Seat selection still shows in UI - state not lost!)
   ↓ [User scrolls seat map to find berth for Passenger 2]
   ↓ (Redux remembers Passenger 1 selection - not cleared by scroll)
   ↓ [User taps lower berth for Passenger 2]
   ↓ (Both selections saved)
   ↓ [Proceed to next step with confidence]
```

---

## Technical Implementation Plan

### Frontend State Management

**Redux Store Structure:**
```javascript
{
  booking: {
    currentBooking: {
      trainId: "12345",
      route: {from: "Delhi", to: "Agra"},
      date: "2026-05-10",
      
      selectedSeats: {
        passenger_1: {
          passengerId: "p1",
          passengerName: "Raj Kumar",
          coach: "C",
          berth: "lower",
          seatPosition: "L1",
          berthType: "lower",
          selectedAt: 1620000000,
        },
        passenger_2: {
          passengerId: "p2",
          passengerName: "Priya Kumar",
          coach: "C",
          berth: "lower",
          seatPosition: "L2",
          berthType: "lower",
          selectedAt: 1620000030,
        },
      },
      
      passengerDetails: {
        passenger_1: {
          name: "Raj Kumar",
          age: 45,
          gender: "M",
          idType: "Aadhar",
          idNumber: "xxxx1234",
        },
        passenger_2: {
          name: "Priya Kumar",
          age: 42,
          gender: "F",
          idType: "Aadhar",
          idNumber: "xxxx5678",
        },
      },
      
      bookingMetadata: {
        createdAt: 1620000000,
        lastModifiedAt: 1620000100,
        sessionId: "sess_abc123xyz",
        stateVersion: 2, // For backward compatibility
      }
    }
  }
}
```

**Persistence Layer:**
```javascript
// Middleware that syncs Redux → LocalStorage
export const persistenceMiddleware = store => next => action => {
  const result = next(action);
  
  // Only persist booking-related actions
  if (action.type.startsWith('booking/')) {
    const state = store.getState().booking.currentBooking;
    localStorage.setItem('irctc_booking_state', JSON.stringify(state));
    
    // Also sync to server for multi-device sync
    syncBookingStateToServer(state);
  }
  
  return result;
};

// On app load: Restore from localStorage
function restoreBookingState() {
  const saved = localStorage.getItem('irctc_booking_state');
  if (saved) {
    try {
      return JSON.parse(saved);
    } catch (e) {
      console.error('Failed to restore booking state:', e);
      return null;
    }
  }
  return null;
}

// On unload: Save current state (backup)
window.addEventListener('beforeunload', () => {
  const state = store.getState().booking.currentBooking;
  sessionStorage.setItem('irctc_booking_emergency_backup', JSON.stringify(state));
});
```

**Recovery on Page Reload:**
```javascript
// On mount: Try to recover previous booking session
useEffect(() => {
  const saved = restoreBookingState();
  if (saved) {
    dispatch(restoreBookingState(saved));
    showToast('Session restored. Your seat selections are preserved.');
  }
}, []);
```

### New Components

- `SeatSelectionSidebar.jsx` - Shows selected seats in real-time while user scrolls seat map
- `SessionRecoveryBanner.jsx` - "We recovered your seat selections from 5 minutes ago" message
- `BookingProgressIndicator.jsx` - Multi-step progress with session checkpoint indicators

### Backend Changes

**New Session Endpoint for State Sync:**
```
POST /booking/session/sync
  Request: {
    sessionId,
    bookingState: {
      selectedSeats: {...},
      passengerDetails: {...},
      ...
    }
  }
  Response: {
    success: true,
    sessionId,
    lastSyncAt: "2026-05-08T10:30:00Z"
  }
```

**Recovery Endpoint:**
```
GET /booking/session/:sessionId/restore
  Response: {
    bookingState: {...},
    lastModifiedAt: "2026-05-08T10:30:00Z",
    expiresAt: "2026-05-08T11:30:00Z"
  }
```

### API Changes

| Method | Endpoint | Purpose | Request | Response |
|--------|----------|---------|---------|----------|
| POST | `/booking/session/sync` | Sync state to server | `{sessionId, bookingState}` | `{success, sessionId}` |
| GET | `/booking/session/:sessionId/restore` | Recover session state | - | `{bookingState, lastModifiedAt}` |
| DELETE | `/booking/session/:sessionId` | Clear abandoned session | - | `{success}` |

### Database Schema

**New Table: `booking_sessions`**
```sql
CREATE TABLE booking_sessions (
  session_id VARCHAR(50) PRIMARY KEY,
  user_id INT NOT NULL,
  booking_state JSON, -- Serialized Redux state
  seat_selections JSON,
  passenger_details JSON,
  created_at TIMESTAMP DEFAULT NOW(),
  last_modified_at TIMESTAMP,
  expires_at TIMESTAMP, -- Auto-delete after 2 hours of inactivity
  status ENUM('active', 'abandoned', 'booked', 'expired'),
  FOREIGN KEY (user_id) REFERENCES users(user_id),
  INDEX idx_user_id_status (user_id, status),
  INDEX idx_expires_at (expires_at)
);
```

### Third-Party Services

- **None required** (uses browser localStorage + server session storage)

### Storage Strategy

| Layer | Storage Type | Capacity | TTL | Use Case |
|-------|--------------|----------|-----|----------|
| L1 | Redux Memory | ~10MB | Until app close | Instant UI updates |
| L2 | Browser LocalStorage | ~5MB | 2 hours | Survive page refresh |
| L3 | Server Sessions | Database | 2 hours | Multi-device sync |

**Sync Frequency**: Every 5 seconds (or on user action) to server

---

## Success Metrics

| Metric | Current | Target | Measurement |
|--------|---------|--------|-------------|
| Seat selection persistence | 30% retain (broken) | 95% retain | % bookings where seats persist across pages |
| Mobile re-selection rate | 50% of users must re-select | 5% must re-select | Session logs, booking flow tracking |
| Session recovery success | N/A (new feature) | 85% recovery success | % of recovered sessions that lead to booking |
| Booking completion rate (family) | 65% | 85% | Multi-passenger booking conversions |
| Time in seat selection step | 3-5 min | 1-2 min | Step duration analytics |
| Mobile booking completion rate | 38% → | 65% | Mobile-specific conversion tracking |

---

## Edge Cases & Constraints

### Edge Cases

1. **Session Expires During Booking**: User takes >2 hours to complete booking
   - **Handling**: Show warning at 1.5 hours; offer to extend session or save as draft
   - **TTL**: 2 hours per session; extendable to 4 hours if user is active

2. **Seat Becomes Unavailable**: Another user books the same seat while current user fills form
   - **Handling**: On step 4 (review), check seat availability; if unavailable, show "Seat Booked" with alternative suggestions
   - **Recovery**: Allow user to pick new seat and continue

3. **LocalStorage Full** (rare): Browser storage quota exceeded
   - **Handling**: Fallback to in-memory Redux only; show warning "Session saved in memory only. Do not close tab."
   - **Mitigation**: Server session backup still active

4. **User Opens Booking on Two Devices Simultaneously**: Same session state on Desktop + Mobile
   - **Handling**: Last-write-wins; show "Session active elsewhere" warning on second device
   - **Recovery**: Sync button to refresh current state from server

5. **Browser Session Cleared** (user clears cache): LocalStorage deleted
   - **Handling**: Server session still active; prompt user to restore: "We have your booking from 10 minutes ago. Restore?"
   - **Restore Duration**: Available for 2 hours

### IRCTC-Specific Constraints

- **Railway Backend Integration**: Seat availability must be re-checked at payment time (not cached)
- **State Version Management**: If app updates state schema, old sessions must be migrated or discarded gracefully
- **Privacy**: Session data contains PII (passenger names, IDs); encrypt in transit and at rest
- **Mobile Network**: Sync to server uses <1KB bandwidth; works on 2G connections

---

## Rollout Plan

**Phase 1 (Week 1)**: Deploy server session storage (silent launch)
- Users don't see UI changes; backend is ready for state sync
- No risk to existing flows

**Phase 2 (Week 2)**: Deploy Redux + LocalStorage (L1 + L2 caching)
- A/B test with 10% of users
- Measure: session persistence improvement
- Rollback trigger: >5% increase in booking errors

**Phase 3 (Week 3)**: Deploy recovery UI + full sync
- Show "Session Restored" banner when recovery works
- 50% rollout; monitor adoption
- Enable for all users if >90% sessions persist correctly

**Full Rollout**: Week 4 (100% of users)

---

## Related Problems Solved

- **Problem 3 (Seat Selection Resets)**: ✅ Directly solves
- **Problem 5 (Mobile Form Difficulty)**: Partial; reduces form friction
