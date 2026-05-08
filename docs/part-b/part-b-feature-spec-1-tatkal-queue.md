# Feature Spec 1: Tatkal Virtual Queue System

## Problem Statement

IRCTC receives 20–40 lakh concurrent requests at 10:00 AM when Tatkal quota opens, causing server overload, session crashes, and zero user feedback during failure. 60–70% of users who attempt Tatkal booking fail due to server-side errors, not quota unavailability. The system lacks any queue management mechanism, leading to cascading failures, invisible retry loops, and user frustration.

**Impact from Part A**: 100% failure rate during peak window, 2-5 minute system unavailability, 500K affected daily users.

---

## Proposed Solution

Replace the current direct-hit booking flow with a virtual waiting room. Users who visit the Tatkal section between 9:55–10:05 AM are assigned a queue position before the booking window opens. At 10:00 AM, positions are served in order at a controlled rate (500 users/minute). Each user sees their live queue number, estimated wait time, and a countdown. When their turn arrives, they get a 90-second booking window to complete checkout. If unused, the slot passes to the next user in queue.

### User Experience (Proposed)

```
Timeline: User's Tatkal Booking Journey (Proposed)

09:55 AM - User navigates to Tatkal section
   ↓
System: "Tatkal opens in 4:23. You are in queue. Position: #4,281"
   ↓
UI shows:
   - Live countdown timer (4:23 → 4:22 → ... → 0:00)
   - Queue position with progress bar
   - Message: "Your seat preferences are pre-loaded"
   ↓
10:00 AM - Queue begins processing
   ↓
Position updates in real-time:
   - #4,281 → #3,100 (est. 5 min wait)
   - #3,100 → #1,205 (est. 2 min wait)
   - #1,205 → #89 (est. 30 sec wait)
   - #89 → "YOUR TURN" + Green countdown (90 seconds)
   ↓
Booking window opens:
   - Seat map loads (pre-filled with saved preferences)
   - Passenger details auto-filled
   - Payment form ready
   ↓
User has 90 seconds to:
   - Review seats → Confirm → Process payment
   ↓
Outcome:
   - ✅ Booking confirmed OR
   - ⏱️ Time expired → Position released to next user + user offered next available slot
```

---

## Technical Implementation Plan

### System Architecture Changes

**Backend Components:**
1. **Redis-backed Queue** (O(log N) operations)
   - Sorted set: `tatkal:queue:2026-05-08` → {userId: priority, timestamp}
   - Queue capacity limit: 50K concurrent queue positions
   - Auto-expiry: Positions expire after 15 minutes of inactivity

2. **Queue Controller Service**
   - Processes N users per second based on backend capacity (start: 500 users/min)
   - Dequeues position → Issues session token → Sets 90-second TTL
   - Monitors backend success rate; throttles if errors exceed 5%
   - Fallback: If Redis fails, system falls back to direct booking (graceful degradation)

3. **WebSocket Server** (Socket.io)
   - Live position update channel: `tatkal:queue:[queueId]`
   - Emits position update every 2 seconds (or on rank change)
   - Fallback: HTTP polling every 5 seconds for 2G/3G users

4. **Session & Token Management**
   - Queue token issued on join: `{queueId, position, tokenId, expiresAt}`
   - Booking token issued at turn: `{bookingWindowToken, expiresAt: now + 90s}`
   - Prevents double-booking: Token consumed on first payment submission

### Frontend Implementation

**New Pages/Routes:**
- `/tatkal/queue` - Waiting room interface (primary)
- `/tatkal/booking?token=xxx` - Booking window (appears only when token active)

**Components:**
- `QueueWaitingRoom.jsx` - Countdown + position display + progress bar
- `BookingWindow.jsx` - Pre-filled form with 90-second timer
- `QueueExpiredModal.jsx` - Shows next available slots when time expires

**State Management (Redux):**
```javascript
// Redux store structure
{
  tatkal: {
    queue: {
      queueId: "q_abc123",
      position: 4281,
      estimatedWait: 300, // seconds
      status: "waiting" | "your_turn" | "expired",
      tokenId: "token_xyz",
    },
    booking: {
      windowToken: "bw_123",
      timeRemaining: 90,
      selectedSeats: {...},
      passengerDetails: {...},
    }
  }
}
```

**Real-time Updates:**
- WebSocket listener: `socket.on('tatkal:queue:update', (data) => dispatch(updateQueuePosition(data)))`
- Countdown timer: `setInterval(() => dispatch(decrementTimer()), 1000)`

### API Changes

| Method | Endpoint | Purpose | Request | Response |
|--------|----------|---------|---------|----------|
| POST | `/tatkal/queue/join` | Join waiting room | `{route, date}` | `{queueId, position, eta}` |
| GET | `/tatkal/queue/status/:queueId` | Check position | - | `{position, eta, status}` |
| POST | `/tatkal/booking/start` | Get 90s window | `{queueId, tokenId}` | `{bookingWindowToken, expiresAt}` |
| POST | `/tatkal/booking/confirm` | Submit booking | `{bookingWindowToken, seats, passengers, paymentId}` | `{confirmationId, pnr}` |
| GET | `/tatkal/queue/next-slots` | Show next bookable times | - | `[{trainId, departure, seats}...]` |

### Database Schema

**New Table: `tatkal_queue_positions`**
```sql
CREATE TABLE tatkal_queue_positions (
  queue_id VARCHAR(50) PRIMARY KEY,
  user_id INT,
  route_id INT,
  position INT,
  joined_at TIMESTAMP,
  turn_started_at TIMESTAMP NULL,
  expires_at TIMESTAMP,
  status ENUM('waiting', 'active', 'expired', 'completed'),
  booking_token VARCHAR(100) UNIQUE NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_status ON tatkal_queue_positions(status);
CREATE INDEX idx_user_id ON tatkal_queue_positions(user_id);
```

**Modified Table: `user_sessions`**
```sql
ALTER TABLE user_sessions ADD COLUMN 
  tatkal_queue_token VARCHAR(100) NULL,
  tatkal_window_expires_at TIMESTAMP NULL;
```

### Third-Party Services

- **Redis**: In-memory queue management (existing infrastructure likely has this)
- **Socket.io**: Real-time communication (open-source, no additional cost)
- **Kafka (optional)**: Event stream for queue analytics and monitoring

### Infrastructure Requirements

- Redis cluster: 4GB+ memory (handles 50K concurrent positions)
- WebSocket server: 4 instances (load balanced, can handle 10K concurrent connections each)
- No changes to payment gateway or railway backend API

---

## Success Metrics

| Metric | Current | Target | Measurement |
|--------|---------|--------|-------------|
| Tatkal booking completion rate | 40% | 70% | Conversions during peak window |
| Server error rate at 10:00 AM | 35% | <5% | Error logs, APM dashboards |
| User wait time visibility | 0 (invisible) | 100% visible | % users who see queue position |
| Queue-related complaints | Baseline (high) | -80% reduction | Support ticket analysis |
| System availability (10-10:05 AM) | 60% uptime | 99.9% uptime | Infrastructure monitoring |
| Avg booking completion time | 8-12 min (failures) | 2-3 min (success) | Session analytics |

---

## Edge Cases & Constraints

### Edge Cases

1. **Queue Position Expires**: User is inactive for 15 minutes
   - **Handling**: Auto-expiry of queue position; notify user of next available slot

2. **Booking Window Token Expires**: User doesn't complete booking in 90 seconds
   - **Handling**: Token revoked; position released to next user; user shown "next available" options

3. **Payment Fails During 90-Second Window**: User initiates payment but transaction fails midway
   - **Handling**: Keep booking token valid for 2 additional retries within 90s window; after 90s, release slot

4. **User Refreshes Page While in Queue**: Session lost, but user rejoins
   - **Handling**: Queue position persisted in Redis; rejoin with same queue ID if within 15 min expiry

5. **WebSocket Connection Drops**: Real-time updates fail
   - **Handling**: Fallback to HTTP polling every 5 seconds; notify user of degraded experience

6. **Redis Cluster Fails**: Queue system goes down
   - **Handling**: Graceful degradation to direct booking (no queue); alert ops team; restore from backup

### IRCTC-Specific Constraints

- **Railway Backend API**: Cannot be modified; queue system must call railway seat API only at booking confirmation time
- **Payment Gateway SLA**: Payment processing takes 5-8 seconds; extend 90-second window to 120 seconds if payment is slow
- **Mobile Network Constraints**: Must support 2G users; polling fallback every 5 seconds uses minimal bandwidth (<1KB per poll)
- **Government Compliance**: All queue positions must be logged for audit purposes (which queue position booked which seats)
- **Scalability**: System must handle 50x current load; test with 2 million concurrent users before launch

---

## Rollback Plan

If queue system fails at 10:00 AM on launch day:
1. **Immediate**: Disable queue feature flag; users routed to direct booking
2. **Notification**: "Queue system temporarily offline. Please try direct booking."
3. **Recovery**: Restart Redis cluster; restore from backup
4. **Canary Launch**: Roll out to 5% of Tatkal users first (Day 1); 25% (Day 2); 100% (Day 3)

---

## Related Problems Solved

- **Problem 1 (Tatkal Crashes)**: ✅ Directly solves
- **Problem 4 (Waitlist Notifications)**: Partial; same notification infrastructure can be reused
