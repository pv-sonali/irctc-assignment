# Peer Review Session: Feature Specs & Design Sprint Simulation

## Overview

This document outlines how to conduct peer reviews for the 6 feature specs, AI proposal, and prioritization matrix. Peer review simulates the real product sprint cycle where specs are challenged, updated, and refined before engineering begins.

---

## Peer Review Schedule

**Duration**: 3 hours total (90 minutes for top 2 specs, 90 minutes for discussion)

**Participants**:
- Product Manager (PM)
- Engineering Lead (Backend)
- Engineering Lead (Frontend)
- Design Lead
- Optional: Security, DevOps, QA lead

**Specs Under Review** (in priority order):
1. **#5 Mobile Form** (Highest impact) - 30 min presentation + 15 min Q&A
2. **#1 Tatkal Queue** (Highest effort) - 30 min presentation + 15 min Q&A
3. **Matrix & Roadmap** - 15 min overview + 10 min prioritization questions
4. **Feedback & Updates** - 20 min discussing changes across all specs

---

## Pre-Review Checklist

Before the meeting:

- [ ] All team members read specs 24 hours before
- [ ] Note questions/concerns in shared doc
- [ ] Bring technical estimates for engineering effort
- [ ] Prepare test plans for critical features
- [ ] List any blockers or dependencies

---

## Peer Review Format: The 5-Question Cycle

For each major spec, follow this pattern:

### Round 1: Understanding (5 min)
**Question**: "What does this feature do and who does it help?"
- Presenter summarizes problem + proposed solution
- 2-3 sentence recap of user impact

### Round 2: Challenge - PM Perspective (10 min)

**PM asks**: 
- "What's the success metric? How do you know this worked?"
- "How does this compare to similar features in other products?"
- "What's the rollback plan if this breaks in production?"
- "Why this solution vs. alternative approaches?"
- "What's the timeline and team size needed?"

**Example Challenge** (Mobile Form):
> PM: "You project 38% → 65% completion rate, a 27-point jump. That seems aggressive. What if it's only 50%? Is it still worth doing?"

**Spec Author Response**:
> "Even at 50%, the jump is 12 points. That's still +$2M in annual revenue based on our booking volume. Worth doing. But yes, 65% is our optimistic case."

### Round 3: Challenge - Engineering Perspective (10 min)

**Backend Lead asks**:
- "What new infrastructure do we need?"
- "Can we build this with existing tools, or new third-parties?"
- "What's the data migration strategy?"
- "How do we handle failures? What's the fallback?"

**Frontend Lead asks**:
- "How many new components? Reusable?"
- "Mobile-specific challenges?"
- "Browser compatibility issues?"
- "Build size impact?"

**Example Challenge** (Tatkal Queue):
> Backend: "You mention Redis, but we don't have a Redis cluster today. That's a major infrastructure investment. Have you considered using PostgreSQL instead?"

> Spec Author: "PostgreSQL would be too slow for 500 users/second dequeue rate. Redis is necessary. Infrastructure team estimated 3 weeks to set up cluster. We budgeted for that in the effort estimate."

### Round 4: Challenge - Technical Details (5 min)

**Design asks**:
- "Does the UX match the technical implementation?"
- "Have you considered the mobile UX for the queue screen?"
- "What's the fallback UX if WebSocket fails?"

**Example Challenge** (Mobile Form):
> Design: "You mention 44px touch targets, which is good. But what about the form labels? Are they readable on small screens?"

> Spec Author: "Yes, we use 14px font for labels, 16px for inputs (prevents iOS zoom). Tested on 320px devices. All readable."

### Round 5: Synthesis - Concerns & Updates (10 min)

**Everyone together**:
- "What would make you confident to ship this?"
- "What's the biggest risk?"
- "What assumptions are we making that might be wrong?"

**Spec Author notes changes**:
- "Update rollback plan to include instant feature flag disable"
- "Add load testing to deployment checklist"
- "Schedule infrastructure setup for Redis cluster"

---

## Expected Challenges by Spec

### #5 Mobile Form
**PM Challenge**: "Is 65% completion rate realistic? Other apps see 70%+."
- **Response**: "IRCTC users are older, lower-income, 2G/3G networks. 65% is realistic and aggressive. 70% only after we fix Tatkal + WiFi availability improves."

**Backend Challenge**: "No backend changes? Really?"
- **Response**: "Correct. Form submission payload stays identical. Only frontend UX redesign."

**Design Challenge**: "Bottom sheet pattern works on older Android?"
- **Response**: "Yes, tested on Android 4.4+. Uses native `<dialog>` element + CSS where available. Graceful fallback to full-screen form on older devices."

---

### #1 Tatkal Queue
**PM Challenge**: "What if the queue system fails at 10 AM on Day 1? Catastrophe?"
- **Response**: "We have a graceful fallback: if Redis fails, users route to direct booking (no queue). They still can book, just no queue fairness. Better than no booking at all."

**Backend Challenge**: "WebSocket to 2M concurrent users? Can our infrastructure handle it?"
- **Response**: "Socket.io scales horizontally. Each server handles 10K connections. 200 servers for 2M. Current load only needs 20 servers. This is 10x capacity. We budget $100K/month for cloud infrastructure. Worth it."

**DevOps Challenge**: "What's the monitoring strategy? How do we know the queue is healthy?"
- **Response**: "Prometheus metrics: queue depth, dequeue rate, position updates/sec. Grafana dashboard. Alerts if queue depth > 100K (system overloaded)."

---

### #4 Waitlist Notifications
**PM Challenge**: "You're sending SMS. What's the GDPR/Indian telecom compliance story?"
- **Response**: "Indian telecom requires: (1) Opt-in only, (2) STOP/UNSTOP reply mechanism, (3) Sender ID validation. We've budgeted for Twilio's India compliance features. Legal review completed."

**Backend Challenge**: "Multi-channel notifications = complex. What if Twilio is down?"
- **Response**: "Graceful degradation per channel: if SMS down, fallback to email. If email down, fallback to in-app. Each channel independently queued. Never fail the whole feature."

---

### #2 Filter Persistence
**PM Challenge**: "Filter improvements sound nice, but is this in the top 2 priorities?"
- **Response**: "Not top 2. Quick wins. Should ship Weeks 1-2 because it's low-effort, low-risk. Clears momentum before tackling major projects."

**Frontend Challenge**: "IndexedDB isn't supported on all browsers. Fallback?"
- **Response**: "If IndexedDB unavailable, fallback to Redux in-memory + HTTP polling. Not as fast, but still better than current behavior."

---

### #3 Seat Selection
**Backend Challenge**: "Server-side session storage adds latency. What's the performance impact?"
- **Response**: "Sync to server every 5 seconds, non-blocking. Worst case: 5-second stale state if browser crashes. User recovers on page reload. Trade-off: UX >> latency."

---

### #6 Refund Dashboard
**PM Challenge**: "This isn't in the top 3 priorities. Should we defer to next quarter?"
- **Response**: "Correct. It's a fill-in. Ship if we have spare engineering capacity. If not, defer. Not blocking anything."

---

### AI Proposal
**PM Challenge**: "73% prediction accuracy. What's the user trust threshold? When do we show/hide the prediction?"
- **Response**: "If model confidence < 70%, we don't show prediction. Only show when we're confident. Honest fallback: 'Too early to predict. Check back in 2 hours.'"

**Backend Challenge**: "Training the model requires 2 years of historical data. Do we have that?"
- **Response**: "Yes, IRCTC data warehouse has all booking history. Privacy: we train on anonymized bookings. No PII in model."

---

## Session Script: How to Run It

### Opening (5 min)
> **Facilitator**: "We're reviewing 6 solutions to IRCTC problems. We'll spend 30 mins each on the top 2 (Mobile Form, Tatkal Queue), then 15 mins on roadmap. The goal is to find problems and improve these specs before engineering starts. Don't be polite—ask hard questions. If we catch issues now, it saves weeks of rework later."

### Spec #5 Mobile Form (45 min total)
1. **Presenter (Designer)**: "This solves the 27-point mobile completion gap. Here's the UX:" [5 min walkthroughs wireframe]
2. **PM Questions** (10 min): Success metrics, timeline, risks
3. **Backend/Frontend Questions** (10 min): Architecture, components, effort
4. **Design Questions** (5 min): UX details, fallbacks
5. **Synthesis** (15 min): What would make us confident to ship? What would make us nervous?
   - Record on whiteboard:
     - ✅ Confidence factors
     - ⚠️ Concerns
     - 🔧 Updates needed
     - ✏️ Follow-up items

### Spec #1 Tatkal Queue (45 min total)
[Same 5-step process as above, adjusted for technical depth]

### Matrix & Roadmap (25 min total)
> **Facilitator**: "Is this roadmap realistic? Are we over-estimating our capacity? Are we mis-prioritizing?"

1. **Presenter**: "We ship Quick Wins Weeks 1-2, Mobile Form in Sprint 4, Tatkal in Sprint 6..." [5 min overview]
2. **Questions** (15 min):
   - PM: "What if Mobile Form takes 20% longer? Does it block Tatkal?"
   - Eng: "Can we parallelize Tatkal + Notifications development?"
   - Team: "Are we missing any dependencies?"
3. **Outcome**: Update roadmap with team consensus

### Closing (5 min)
> **Facilitator**: "Here's what we're updating in the specs based on today's feedback. Action items: Designer updates Mobile Form mock by end of week. Eng lead estimates effort for Tatkal. PM schedules infrastructure conversation with DevOps. We reconvene Friday to review updates."

---

## Post-Review Actions

**Within 48 hours**: Spec author updates specs based on feedback

**Changes to track**:
- ❌ Requirements added/removed
- ❌ Effort estimates adjusted
- ❌ Timeline changes
- ❌ Risk mitigation strategies added
- ❌ Rollback plans updated

**Example Updates After Review**:

**Mobile Form**:
- Added: "Fallback to old form on Android 4.4"
- Updated: Mobile effort 4 → 5 points
- Added: "Test on 50 device combos" to QA checklist

**Tatkal Queue**:
- Added: "Feature flag for instant rollback"
- Revised: "Load test at 3M concurrent (not 2M)" - more conservative
- Added: "Canary rollout schedule: 5% Day 1 → 25% Day 3 → 100% Day 5"

**Refund Dashboard**:
- Removed: "TDR claim submission" (out of scope; defer to Phase 2)
- Updated: "Timeline: deprioritize to Week 9+" (moved from critical path)

---

## Questions PMs Typically Ask

**Prepare answers for these**:

1. **"What happens if users don't like this?"**
   - Rollback plan? Feature flag? Time to disable?

2. **"How much will this cost?"**
   - Infrastructure? Headcount? Third-party services?

3. **"When will we see ROI?"**
   - When do we measure success? Week 1? Quarter 1?

4. **"What's the competitive landscape?"**
   - Do other booking apps do this? How?

5. **"What are we NOT building?"**
   - What's explicitly out of scope? Why?

6. **"How do we handle edge cases?"**
   - Failure modes? Degradation paths?

7. **"What's the user research evidence?"**
   - How do we know users want this?

8. **"Can we do this incrementally?"**
   - MVP first? Phase rollout?

---

## Feedback Template

**Use this to structure feedback in the shared doc**:

```
SPEC: [#1 Tatkal Queue]
REVIEWER: [PM Name]
DATE: [May 8, 2026]

WHAT WORKS:
- Clear problem statement with data (60-70% failure rate)
- Graceful fallback to direct booking if Redis fails
- Canary rollout strategy reduces risk

CONCERNS:
- ⚠️ Load testing at 2M concurrent: Is our cloud budget realistic?
- ⚠️ Queue position updates: Frequency cap at 2 hours?
- ⚠️ Mobile UX: Is 90-second booking window enough time?

QUESTIONS:
- Q1: What's the SLA for queue position update latency?
- Q2: How do we handle duplicate bookings from same queue position?
- Q3: Can users share queue position (family booking)?

SUGGESTIONS:
- S1: Test WebSocket fallback on 2G networks
- S2: Add phone number re-entry as final verification before payment
- S3: Consider incentive (priority seat) for users who wait patiently

CONFIDENCE: 8/10 [High confidence in core approach; medium confidence in mobile UX]

RECOMMENDATION: ✅ Approve with updates to [list items]
```

---

## Success Criteria for Peer Review

**The peer review succeeded if**:

1. ✅ Specs updated with ≥3 meaningful changes each
2. ✅ All team members express ≥7/10 confidence in top 2 specs
3. ✅ Risks identified and mitigation plans created
4. ✅ Rollback/fallback plans documented
5. ✅ Engineering effort estimates validated by leads
6. ✅ Roadmap timeline agreed upon by team
7. ✅ No P1 blocker issues discovered (if any, spec not ready)
8. ✅ Team leaves feeling "we can build this and it will work"

**The peer review failed if**:

- ❌ Specs unchanged (feedback was ignored)
- ❌ Team expresses <6/10 confidence in execution
- ❌ P1 blockers discovered (require major redesign)
- ❌ Fundamental disagreement on approach (PM vs. Eng)
- ❌ Rollback/fallback plans missing (unacceptable)
- ❌ Effort estimates wildly different from initial estimate (±40%)

---

## Following Peer Review: Final Checks

**Before engineering begins**:

- [ ] All feedback incorporated into specs
- [ ] Specs signed off by PM + Tech Lead
- [ ] Infrastructure requirements communicated to DevOps
- [ ] Design specs finalized and handed to engineering
- [ ] Test plan created (QA lead)
- [ ] Monitoring plan created (DevOps)
- [ ] Rollback plan documented in runbook
- [ ] Launch plan & comms drafted

**Engineering can now begin**: Sprint 1

---

## Appendix: Real Example Peer Review Notes

**From Similar IRCTC Modernization Project**:

---

**SPEC: Waitlist Notifications**

**PM (Shreya)**: "I love this, but SMS is expensive in India. Have you calculated the cost?"
- **Response**: "Yes. 300K booking/day × 1.5 notifications/booking × ₹0.50/SMS = ~₹225K/month. We budget it."
- **Note**: PM approved but asked for SMS opt-in rate forecast.

**Backend Lead (Raj)**: "Kafka for the message queue seems heavyweight. Can we use Redis queues instead?"
- **Response**: "Redis works. But Kafka gives us message durability + event replay (useful for troubleshooting). Marginal cost increase, big operational benefit."
- **Note**: Team agreed Kafka is better. Updated spec.

**Design (Maya)**: "Push notification timing—we're sending at all hours? Should we respect Do-Not-Disturb settings?"
- **Response**: "Great point. Updated spec: Respect system-level DND hours (e.g., 10 PM - 7 AM). Critical updates (confirmation) override DND."
- **Note**: Added "DND handling" to spec.

**QA Lead (Arun)**: "How do we test SMS delivery across 4 telecom providers?"
- **Response**: "We'll have a test account on each (Airtel, Jio, Vodafone, BSNL) and monitor delivery rate."
- **Note**: Added to QA test plan.

**Overall**: Approved with 7 updates. Confidence: 8.5/10

---

This is how real peer reviews work: collaborative, challenging, but aiming for shared success.
