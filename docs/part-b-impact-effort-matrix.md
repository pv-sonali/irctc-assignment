# Part B Deliverable: 2×2 Impact vs Effort Matrix & Roadmap

## Overview

This 2×2 matrix prioritizes the 6 solutions based on business impact and engineering effort. It helps leadership decide what to build first, what to defer, and what to reconsider.

---

## Scoring Framework

### Impact Score (1-5)
- **Users affected**: How many IRCTC users benefit?
- **Core flow impact**: Does it improve core booking flow (high priority)?
- **Problem severity**: Critical vs. medium vs. nice-to-have?
- **Consequence**: Lost bookings, refunds, or just frustration?

**Total Impact Range**: 3-15 (lower = less impact, higher = more impact)

### Effort Score (1-5)
- **Frontend complexity**: Lines of code, new components?
- **Backend work**: New APIs, database schema, infrastructure?
- **Third-party services**: Need new external integrations?
- **Risk**: Likelihood of breaking existing flows?
- **Testing burden**: QA effort and manual testing?

**Total Effort Range**: 4-20 (lower = less effort, higher = more effort)

---

## Individual Solution Scoring

### Problem 1: Tatkal Virtual Queue System

| Dimension | Score | Notes |
|-----------|-------|-------|
| **IMPACT** |
| Users affected (1-5) | 5 | 500K daily users during peak; 60-70% failure rate |
| Core booking impact (1-5) | 5 | Solves complete system collapse at 10 AM |
| Problem severity (1-5) | 5 | Critical - blocks millions of bookings |
| Consequence (1-5) | 5 | Missed trips, refunds, platform loses credibility |
| **IMPACT TOTAL** | **20** | **Highest impact** |
| | |
| **EFFORT** |
| Frontend complexity (1-5) | 2 | Queue waiting room (not complex UI) |
| Backend/infrastructure (1-5) | 5 | Redis cluster, WebSocket server, new APIs |
| Third-party services (1-5) | 3 | Requires Socket.io, possibly Kafka |
| Risk to existing flows (1-5) | 4 | Large system change; careful rollout needed |
| Testing burden (1-5) | 5 | Load testing at 2M concurrent users required |
| **EFFORT TOTAL** | **19** | **Very high effort** |
| | |
| **MATRIX PLACEMENT** | **High Impact / High Effort (Top Right)** | **→ MAJOR PROJECT** |

---

### Problem 2: Smart Filter Persistence

| Dimension | Score | Notes |
|-----------|-------|-------|
| **IMPACT** |
| Users affected (1-5) | 4 | 30% of searchers; intermittent issue |
| Core booking impact (1-5) | 4 | Affects search-to-booking flow; not core checkout |
| Problem severity (1-5) | 4 | High frustration; wasted 2-5 min per search |
| Consequence (1-5) | 3 | Abandoned searches, not lost bookings |
| **IMPACT TOTAL** | **15** | **High-Medium impact** |
| | |
| **EFFORT** |
| Frontend complexity (1-5) | 3 | Redux state + IndexedDB caching |
| Backend/infrastructure (1-5) | 2 | Modify cache invalidation; minimal DB changes |
| Third-party services (1-5) | 1 | None (client-side cache) |
| Risk to existing flows (1-5) | 2 | Low risk; backward compatible |
| Testing burden (1-5) | 2 | Test cache hit/miss scenarios |
| **EFFORT TOTAL** | **10** | **Low-Medium effort** |
| | |
| **MATRIX PLACEMENT** | **High Impact / Low Effort (Top Left)** | **→ QUICK WIN** |

---

### Problem 3: Session State Persistence for Seats

| Dimension | Score | Notes |
|-----------|-------|-------|
| **IMPACT** |
| Users affected (1-5) | 4 | 70% of multi-passenger bookings (20% of all) |
| Core booking impact (1-5) | 4 | Affects seat selection step; major pain point |
| Problem severity (1-5) | 4 | High frustration on mobile; lost time |
| Consequence (1-5) | 3 | Abandoned bookings if time expires (Tatkal) |
| **IMPACT TOTAL** | **15** | **High-Medium impact** |
| | |
| **EFFORT** |
| Frontend complexity (1-5) | 3 | Redux + LocalStorage + Recovery UI |
| Backend/infrastructure (1-5) | 2 | New session table; simple sync endpoint |
| Third-party services (1-5) | 1 | None |
| Risk to existing flows (1-5) | 2 | Low risk; additive feature |
| Testing burden (1-5) | 3 | Test state recovery across devices |
| **EFFORT TOTAL** | **11** | **Low-Medium effort** |
| | |
| **MATRIX PLACEMENT** | **High Impact / Low-Medium Effort (Top Left)** | **→ QUICK WIN** |

---

### Problem 4: Multi-Channel Waitlist Notifications

| Dimension | Score | Notes |
|-----------|-------|-------|
| **IMPACT** |
| Users affected (1-5) | 5 | 300K waitlist bookings daily; 1B annual |
| Core booking impact (1-5) | 3 | Doesn't affect booking flow; post-booking |
| Problem severity (1-5) | 4 | High anxiety; poor UX; excessive checking |
| Consequence (1-5) | 3 | Frustration, support burden, not lost bookings |
| **IMPACT TOTAL** | **15** | **High impact overall** |
| | |
| **EFFORT** |
| Frontend complexity (1-5) | 2 | Notification UI, preference settings |
| Backend/infrastructure (1-5) | 4 | Notification service, rules engine, Kafka queue |
| Third-party services (1-5) | 4 | SendGrid, Twilio/SNS, Firebase, Kafka |
| Risk to existing flows (1-5) | 2 | Low risk; independent system |
| Testing burden (1-5) | 3 | Test each channel; rate limiting |
| **EFFORT TOTAL** | **15** | **High effort** |
| | |
| **MATRIX PLACEMENT** | **High Impact / High Effort (Top Right)** | **→ MAJOR PROJECT** |

---

### Problem 5: Mobile-First Responsive Booking Form

| Dimension | Score | Notes |
|-----------|-------|-------|
| **IMPACT** |
| Users affected (1-5) | 5 | 68-72% of all IRCTC users on mobile |
| Core booking impact (1-5) | 5 | Directly improves booking form UX |
| Problem severity (1-5) | 5 | Critical - 27-point booking completion gap |
| Consequence (1-5) | 5 | 27% of mobile users abandon at form |
| **IMPACT TOTAL** | **20** | **Highest impact** |
| | |
| **EFFORT** |
| Frontend complexity (1-5) | 4 | Bottom sheet, keyboard handling, new components |
| Backend/infrastructure (1-5) | 1 | No backend changes required |
| Third-party services (1-5) | 1 | Only CSS/JavaScript libraries |
| Risk to existing flows (1-5) | 3 | Medium risk; affects critical path |
| Testing burden (1-5) | 4 | Must test on 50+ device/OS combinations |
| **EFFORT TOTAL** | **13** | **Medium-High effort** |
| | |
| **MATRIX PLACEMENT** | **Highest Impact / Medium-High Effort (Top Right)** | **→ MAJOR PROJECT (highest priority)** |

---

### Problem 6: Unified Refund & TDR Dashboard

| Dimension | Score | Notes |
|-----------|-------|-------|
| **IMPACT** |
| Users affected (1-5) | 3 | 5-8% of bookings (cancellations); periodic need |
| Core booking impact (1-5) | 2 | Doesn't affect booking; affects post-booking |
| Problem severity (1-5) | 4 | Confusing, generates support tickets |
| Consequence (1-5) | 2 | Frustration, support burden; not lost bookings |
| **IMPACT TOTAL** | **11** | **Medium impact** |
| | |
| **EFFORT** |
| Frontend complexity (1-5) | 3 | Dashboard UI, timeline component, filters |
| Backend/infrastructure (1-5) | 3 | New refund tracking tables, status APIs |
| Third-party services (1-5) | 2 | Optional bank API for verification |
| Risk to existing flows (1-5) | 1 | Low risk; read-only page |
| Testing burden (1-5) | 2 | Test different refund scenarios |
| **EFFORT TOTAL** | **11** | **Low-Medium effort** |
| | |
| **MATRIX PLACEMENT** | **Medium Impact / Low-Medium Effort (Middle)** | **→ FILL-IN** |

---

## Matrix Visualization

```
          LOW EFFORT ←                    → HIGH EFFORT
          
HIGH   │ ╔════════════════════════════════════════════╗
IMPACT │ ║  QUICK WINS         │      MAJOR PROJECTS   ║
       │ ║  (Do First)         │      (Plan & Resource)║
       │ ║  ┌──────────────┐   │   ┌──────────────┐   ║
       │ ║  │ #2 Filter    │   │   │ #1 Tatkal    │   ║
       │ ║  │ Persistence  │   │   │ Virtual Q.   │   ║
       │ ║  ├──────────────┤   │   ├──────────────┤   ║
       │ ║  │ #3 Seat      │   │   │ #4 Waitlist  │   ║
       │ ║  │ Selection    │   │   │ Notifications│   ║
       │ ║  │ State        │   │   ├──────────────┤   ║
       │ ║  └──────────────┘   │   │ #5 Mobile    │   ║
       │ ║                     │   │ Form *** HIGHEST  ║
       │ ║────────────────────────│ PRIORITY ┌──────  ║
       │ ║  FILL-INS           │   │ Redesign│      ║
       │ ║  (When Capacity)    │   │         │      ║
       │ ║  ┌──────────────┐   │   │         │      ║
       │ ║  │ #6 Refund    │   │   │         │      ║
       │ ║  │ Dashboard    │   │   │         │      ║
       │ ║  └──────────────┘   │   └─────────┘      ║
       │ ║                     │                    ║
       │ ╚════════════════════════════════════════════╝
       │
LOW    │ ╔════════════════════════════════════════════╗
IMPACT │ ║  TIME SINKS         │    RECONSIDER        ║
       │ ║  (Avoid)            │    (Ask Questions)   ║
       │ ║  (None in this set) │    (None in this set)║
       │ ╚════════════════════════════════════════════╝
```

---

## Quadrant Placement Summary

| Quadrant | Solutions | Strategy |
|----------|-----------|----------|
| 🚀 **QUICK WINS** | #2 Filter Persistence #3 Seat State | **Do First** - Max ROI for effort |
| 🏗️ **MAJOR PROJECTS** | #1 Tatkal Queue #4 Waitlist Notifications #5 Mobile Form | **Plan carefully** - High impact, needs resourcing |
| 🧩 **FILL-INS** | #6 Refund Dashboard | **When capacity available** - Nice-to-have |
| ❌ **TIME SINKS** | (None) | **Avoid** - Not applicable |

---

## Prioritized Roadmap (16-Week Sprint Plan)

### Sprint 1-2 (Weeks 1-2): QUICK WINS
**Goals**: Early wins to build momentum; low risk

1. **#2 Filter Persistence** (Effort: 10 points)
   - Deploy Redux state management + client-side caching
   - Measure: filter success rate 40% → 95%
   - **Week 1 Target**: Deployed to 10% of users

2. **#3 Seat State Persistence** (Effort: 11 points)
   - Deploy localStorage + Redux integration
   - Measure: seat selection retention 30% → 95%
   - **Week 2 Target**: Deployed to 50% of users

**Outcome**: Two high-impact features shipped; team confidence high; 0 critical bugs

---

### Sprint 3-4 (Weeks 3-4): MAJOR PROJECT #5 (Highest Impact)
**Goal**: Mobile form redesign - affects 72% of users

1. **#5 Mobile Form Redesign** (Effort: 13 points)
   - Week 3: Design finalization + component library build
   - Week 4: A/B testing on 10% of mobile users
   - Measure: 38% → 60% mobile completion rate

**Outcome**: 22-point completion rate improvement = 💰 major revenue impact

---

### Sprint 5-6 (Weeks 5-6): MAJOR PROJECT #1 (Critical)
**Goal**: Tatkal Virtual Queue - prevents system collapse

1. **#1 Tatkal Queue System** (Effort: 19 points)
   - Week 5: Backend infrastructure (Redis, WebSocket server, APIs)
   - Week 6: Frontend UI + load testing at 2M concurrent
   - Measure: 60-70% → 5% error rate at 10 AM

**Outcome**: System stability during peak loads; handles 10x traffic

---

### Sprint 7-8 (Weeks 7-8): MAJOR PROJECT #4 (UX + Support Load)
**Goal**: Waitlist notifications - reduces anxiety + support tickets

1. **#4 Waitlist Notifications** (Effort: 15 points)
   - Week 7: In-app notifications + email (phases 1-2)
   - Week 8: SMS + push notifications (phases 3-4)
   - Measure: 12 checks/day → 1 check/day per user

**Outcome**: 60% fewer manual status checks; 80% fewer support tickets

---

### Sprint 9+ (Weeks 9+): FILL-INS + AI
**If capacity available:**

1. **#6 Refund Dashboard** (Effort: 11 points)
   - Build unified refund tracking page
   - Measure: support tickets -40%, user confidence +50 NPS points

2. **AI: Waitlist Confirmation Predictor** (Effort: 14 points)
   - Train model on historical data
   - Deploy as feature alongside #4 notifications
   - Measure: prediction accuracy >75%, user trust >80%

---

## Critical Path Analysis

### Dependencies

```
Week 1-2:   [#2 Filter] ──┐
            [#3 Seat]      ├── PARALLEL (independent)
                           │
Week 3-4:                  ├──→ [#5 Mobile Form] (independent)
                           │
Week 5-6:                  ├──→ [#1 Tatkal Queue] (independent)
                           │
Week 7-8:   [#4 Waitlist] ←────┴───────┐
                                       │
Week 9+:    [#6 Refund] ────────────────┤
            [AI Model] ─────────────────→ [#4 Integration]

Legend: ──→ blocks, ← feeds-into, (parallel)
```

**Key Insight**: Solutions are mostly independent; can execute in parallel
- Start #1 Tatkal in Week 5 while shipping #5 Mobile Form in Week 4
- Deploy AI model alongside #4 Notifications; they're complementary

---

## Resource Allocation (Estimated Team Size)

| Phase | Feature | FE | BE | DevOps | QA | Total |
|-------|---------|----|----|--------|----|----- |
| Sprint 1-2 | #2, #3 | 2 | 1 | 0 | 1 | 4 |
| Sprint 3-4 | #5 Mobile | 3 | 0 | 0 | 2 | 5 |
| Sprint 5-6 | #1 Tatkal | 2 | 3 | 2 | 2 | 9 |
| Sprint 7-8 | #4 Notifications | 2 | 3 | 1 | 2 | 8 |
| Sprint 9+ | #6, AI | 2 | 2 | 0 | 1 | 5 |

**Total Sustained Team**: ~8 people across 16 weeks

---

## Success Criteria for Each Quadrant

### QUICK WINS (Weeks 1-2)
✅ **Success**: Both deployed with zero P1 bugs
- #2: Filter success rate 40% → 95%
- #3: Seat persistence 30% → 95%

### MAJOR PROJECTS (Weeks 3-8)
✅ **Success**: All three deployed on schedule
- #5: Mobile completion 38% → 65%
- #1: Tatkal error rate <5%, handles 2M concurrent
- #4: Manual checks -60%, support tickets -40%

### OVERALL PROGRAM (Weeks 1-16)
✅ **Success Metrics**:
- **Booking Completion Rate**: 62% baseline → 75% (+13 points)
- **Mobile Completion Rate**: 38% → 65% (+27 points, primary)
- **Support Tickets**: -30% overall
- **Revenue Impact**: +18% from improved conversion (estimated)
- **System Availability**: 99.9% uptime during peak hours
- **User Satisfaction**: NPS +25 points

---

## Justifications for Each Placement

### #1 Tatkal (High Impact / High Effort) → MAJOR PROJECT ✅

**Why High Impact**:
- 500K users affected daily
- 60-70% experience complete failure during peak
- System-wide collapse; cascades to other users
- Direct revenue loss (failed bookings)

**Why High Effort**:
- Requires new infrastructure (Redis, WebSocket, Kafka)
- Load testing at 2M concurrent users
- Complex distributed system coordination
- Rollback risk if problems occur

**Justification**: Worth the effort because failure is existential crisis. Tatkal is 50% of IRCTC's traffic. System collapse is unacceptable.

---

### #2 Filter (High Impact / Low Effort) → QUICK WIN ✅

**Why High Impact**:
- 30% of users affected
- Wasted 2-5 minutes per search
- Core search flow broken
- Accumulated impact is huge (millions of minutes wasted daily)

**Why Low Effort**:
- Only client-side state management + caching
- No backend changes needed (backward compatible)
- Low risk to existing functionality
- Can be deployed independently

**Justification**: Best ROI. Ship first thing. Zero reasons not to. High confidence, low cost.

---

### #3 Seat State (High Impact / Low-Medium Effort) → QUICK WIN ✅

**Why High Impact**:
- 70% of multi-passenger bookings affected (20% of all bookings)
- Mobile users especially hurt (50% of them)
- Session recovery prevents checkout failure
- Eliminates 2-3 minute wasted re-selection time

**Why Low-Medium Effort**:
- Simple Redux + localStorage implementation
- New session table (straightforward schema)
- Low risk of breaking existing flows
- Can deploy as feature flag (easy rollback)

**Justification**: Second QUICK WIN after #2. Similar reasoning: high impact, low cost, high confidence.

---

### #4 Waitlist Notifications (High Impact / High Effort) → MAJOR PROJECT ✅

**Why High Impact**:
- 300K waitlist bookings daily; 1B annually
- Users check 12 times daily out of anxiety (massive server burden)
- Support ticket generator; reduces team effectiveness
- Improved experience = stronger user loyalty

**Why High Effort**:
- Multi-channel system (email, SMS, push, in-app)
- Integration with 4+ external services (SendGrid, Twilio, Firebase)
- Notification rules engine (complex state management)
- SMS requires telecom compliance (India-specific complexity)

**Justification**: High impact on UX and ops, but requires careful architecture. Phase rollout (in-app → email → SMS) reduces risk. Worth the investment for scale and user satisfaction.

---

### #5 Mobile Form (HIGHEST Impact / Medium-High Effort) → MAJOR PROJECT #1 PRIORITY ✅✅

**Why Highest Impact**:
- 68-72% of all IRCTC users access via mobile
- 27-point completion gap (38% mobile vs 65% desktop) = revenue hemorrhage
- 60% abandon at passenger form = 36% of mobile bookings lost
- Single largest conversion lever available

**Why Medium-High Effort**:
- Extensive component rewrite (bottom sheet, keyboard handling, touch targets)
- No backend changes (only frontend)
- Massive testing burden (50+ device combos)
- Medium risk: if broken, affects 70% of users

**Justification**: **MUST SHIP EARLY**. Highest ROI of all solutions. 22-point completion improvement = 💰💰💰 in revenue. Worth the testing burden. Risk is high but impact is highest. Start in Sprint 3, deploy in Sprint 4.

---

### #6 Refund Dashboard (Medium Impact / Low-Medium Effort) → FILL-IN ✅

**Why Medium Impact**:
- Only 5-8% of users need refunds (not core path)
- Post-booking feature (doesn't affect primary conversion)
- Support burden reducer (not revenue generator)
- Nice-to-have, not must-have

**Why Low-Medium Effort**:
- Straightforward UI (dashboard + timeline)
- Mostly read-only (low backend complexity)
- No third-party services required (optional bank API)
- Can build during spare capacity in Sprints 5+

**Justification**: Good feature, but deprioritized. Ship after the major projects. If team has spare capacity in Weeks 9+, build it. If not, defer to next quarter.

---

## Risk Mitigation

### High-Risk Items (Need Special Attention)

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| #1 Queue fails at 10 AM on launch | Medium | Critical | Extensive load testing; canary rollout (5% → 25% → 100%); instant rollback plan |
| #4 SMS spam complaints (India) | High | Medium | Strict opt-in; STOP reply mechanism; frequency caps |
| #5 Mobile form breaks on old Android | Medium | High | Test on Android 4.4+; progressive enhancement; fallback |
| AI Model predicts incorrectly | Low | Medium | Fallback to position-only display; continuous monitoring; user feedback loop |

### Mitigation Strategies

1. **#1 Tatkal Queue**: 
   - Week 5: Run load test at 2M concurrent (use AWS locust)
   - Week 6 Day 1: Canary to 5% of Tatkal users
   - Week 6 Day 3: Expand to 25%
   - Week 6 Day 5: Full rollout
   - Instant rollback: disable queue feature flag → direct booking

2. **#4 Waitlist Notifications**:
   - Phase SMS only to opted-in users
   - Monitor spam complaints; cap SMS to 1 per day
   - Test on different telecom networks (Airtel, Jio, Vodafone)

3. **#5 Mobile Form**:
   - Test every screen size: 320px, 375px, 414px, 480px
   - Test on Android 4.4, 5.0, 9.0, 14.0
   - Parallel testing: keep old form available; users can switch if new form breaks

4. **AI Model**:
   - Monitor prediction accuracy in production (should stay >75%)
   - If accuracy drops below 70%, disable feature (graceful fallback)
   - Retrain monthly with new data

---

## Conclusion

This matrix and roadmap provide a 16-week sprint plan to address all 6 problems:

- **Weeks 1-2**: Quick wins (#2, #3) - build momentum
- **Weeks 3-4**: Mobile form (#5) - biggest conversion lever
- **Weeks 5-6**: Tatkal queue (#1) - system stability
- **Weeks 7-8**: Notifications (#4) - UX & ops
- **Weeks 9+**: Refund dashboard (#6) + AI model - nice-to-have

**Expected Outcomes**:
- Booking completion rate: 62% → 75% (+13 points)
- Mobile completion rate: 38% → 65% (+27 points)
- Tatkal error rate: 35% → <5%
- System availability: 99.9% uptime
- Support tickets: -30% overall
- Estimated revenue impact: +18%
