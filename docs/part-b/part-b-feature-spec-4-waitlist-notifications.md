# Feature Spec 4: Multi-Channel Waitlist Notification System

## Problem Statement

When users book waitlist tickets, they receive one-time confirmation email at booking time. After that, there's no proactive communication. Users must manually refresh the website repeatedly to check if their waitlist status changed. The system lacks push notifications, follow-up emails, SMS alerts, and in-app messaging. Users check status 10-15 times daily out of anxiety, creating a poor experience and wasted server load from repeated status checks.

**Impact from Part A**: ~300,000 waitlist bookings daily; users check ~12 times daily average; ~1 billion annual waitlist tickets; only 1 notification channel (email) vs. industry standard of 4+ channels.

---

## Proposed Solution

Implement a multi-channel notification system with 4 parallel channels:

1. **Email**: Initial booking + status change updates (existing, improved)
2. **SMS**: Status change alerts for users who opt-in (new)
3. **Push Notifications**: In-app/mobile app alerts (new)
4. **In-App Notifications**: Badge on "My Bookings" + notification drawer (new)

Additionally, implement **smart notification triggers**:
- Send only when waitlist moves significantly (e.g., position improves by 10+)
- Send when status becomes "Confirmed"
- Send reminder 24 hours before travel if still waitlisted
- Send cancellation notification if user doesn't confirm in time

### User Experience (Proposed)

```
Waitlist Booking Notification Flow (Proposed):

Day 0 (Booking Day) - 3:45 PM:
  ├─ User books waitlist ticket (Delhi-Mumbai, Position #47)
  ├─ ✉️ Email: Booking confirmation + "Your position: #47"
  ├─ 📱 SMS (if opted-in): "IRCTC: Waitlisted for Delhi-Mumbai. Position: #47. Track: www.irctc.co.in/mybooks"
  ├─ 🔔 In-App: Badge appears on "My Bookings" tab (red "WL" badge)
  └─ 📲 Push Notification: "Your waitlist ticket is booked. Current position: #47"

Day 0 - 5:00 PM (1.25 hrs later):
  ├─ Position improves: #47 → #35 (significant change: 12-position jump)
  ├─ 🔔 In-App: "Great news! You moved to position #35" (banner in app)
  ├─ ✉️ Email: "Your waitlist position improved to #35" (sent immediately)
  └─ 📱 SMS: "Your position: #35. Travel in 19 hrs. Follow for updates."

Day 0 - 11:00 PM (7.25 hrs later):
  ├─ Position confirmed: #35 → "CONFIRMED"
  ├─ 🔔 In-App: LARGE BANNER: "🎉 Ticket Confirmed! Coach C, Berth #35" (green)
  ├─ ✉️ Email: "Your ticket is CONFIRMED. Coach C, Lower Berth 35"
  ├─ 📱 SMS: "CONFIRMED! Delhi-Mumbai. Coach C, L35. Travel Dec 10, 11 PM. Have a safe journey!"
  └─ 📲 Push Notification: "Ticket Confirmed! Your journey is confirmed for tomorrow at 11 PM"

Day 1 (Day of Travel) - 8:00 AM:
  ├─ Reminder notification: "Your train departs in 15 hours. Board at Platform 5"
  ├─ 🔔 In-App: "Reminder: Your train departs at 11 PM from Platform 5"
  ├─ ✉️ Email: "Your train departs soon. Boarding information inside"
  └─ 📱 SMS: "Reminder: Delhi-Mumbai train departs 11 PM from Platform 5"

---

If Waitlist Doesn't Confirm:

Day 2 - 2:00 PM (39 hrs before travel):
  ├─ Check triggered: If position hasn't improved significantly, send reminder
  ├─ 🔔 In-App: "Your position is still #8. Auto-cancel in 2 hours if not confirmed."
  ├─ ✉️ Email: "Waitlist still pending. Auto-cancel policy applied."
  └─ 📱 SMS: "Position: #8. We'll auto-cancel if not confirmed by 4 PM."

Day 2 - 4:00 PM (Cancellation Time):
  ├─ Status changed to "Auto-Cancelled" (user did not confirm in time)
  ├─ 🔔 In-App: "Your booking was auto-cancelled per policy"
  ├─ ✉️ Email: "Your booking was auto-cancelled. Refund issued."
  └─ 📱 SMS: "Booking cancelled. Refund of ₹5,000 processed in 2-5 days."
```

---

## Technical Implementation Plan

### Backend Notification Service Architecture

**Notification Service Components:**

1. **Notification Trigger Service**
   - Monitors waitlist position changes in real-time
   - Checks every 30 seconds (or on data change event)
   - Evaluates rules: "Has position improved by 10+?" / "Is status now Confirmed?"
   - Enqueues notifications to message queue

2. **Message Queue** (Apache Kafka or AWS SQS)
   - Decouples notification triggers from channel delivery
   - Ensures no notifications are lost
   - Allows retry logic for failed deliveries

3. **Channel-Specific Delivery Services**
   - **Email Service**: Sends to SendGrid / AWS SES
   - **SMS Service**: Sends to Twilio / AWS SNS
   - **Push Service**: Sends to Firebase Cloud Messaging (FCM)
   - **In-App Service**: Writes to database, delivered via WebSocket

4. **Notification Preference Engine**
   - Stores user's channel preferences (opt-in/opt-out per channel)
   - Checks frequency caps ("Max 1 email per hour")
   - Respects "Do Not Disturb" hours (e.g., 10 PM - 7 AM)

### Data Model

**New Table: `notification_preferences`**
```sql
CREATE TABLE notification_preferences (
  pref_id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT NOT NULL,
  channel ENUM('email', 'sms', 'push', 'inapp'),
  enabled BOOLEAN DEFAULT true,
  opt_in_date TIMESTAMP,
  do_not_disturb_start TIME, -- e.g., '22:00:00'
  do_not_disturb_end TIME,   -- e.g., '07:00:00'
  max_notifications_per_day INT DEFAULT 10,
  FOREIGN KEY (user_id) REFERENCES users(user_id),
  UNIQUE KEY (user_id, channel)
);
```

**New Table: `waitlist_notifications_sent`**
```sql
CREATE TABLE waitlist_notifications_sent (
  notification_id INT AUTO_INCREMENT PRIMARY KEY,
  booking_id INT NOT NULL,
  user_id INT NOT NULL,
  channel ENUM('email', 'sms', 'push', 'inapp'),
  event_type ENUM('booking_confirmed', 'position_improved', 'confirmed', 'reminder', 'cancelled'),
  trigger_position INT,  -- Waitlist position that triggered this
  sent_at TIMESTAMP DEFAULT NOW(),
  status ENUM('sent', 'failed', 'bounced'),
  FOREIGN KEY (booking_id) REFERENCES bookings(booking_id),
  FOREIGN KEY (user_id) REFERENCES users(user_id),
  INDEX idx_booking_id (booking_id)
);
```

**New Table: `in_app_notifications`**
```sql
CREATE TABLE in_app_notifications (
  notif_id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT NOT NULL,
  booking_id INT,
  title VARCHAR(255),
  message TEXT,
  icon_type ENUM('success', 'warning', 'info'),
  created_at TIMESTAMP DEFAULT NOW(),
  read_at TIMESTAMP NULL,
  expires_at TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(user_id),
  INDEX idx_user_read (user_id, read_at)
);
```

### API Changes

| Method | Endpoint | Purpose | Request | Response |
|--------|----------|---------|---------|----------|
| GET | `/user/notification-preferences` | Get user's prefs | - | `{email_enabled, sms_enabled, ...}` |
| PUT | `/user/notification-preferences` | Update prefs | `{channel, enabled}` | `{success}` |
| GET | `/notifications/in-app` | Get unread in-app notifs | `?limit=10` | `{notifications: [...]}` |
| POST | `/notifications/in-app/:id/read` | Mark as read | - | `{success}` |
| POST | `/waitlist/:bookingId/manual-check` | Force status check | - | `{position, status}` |

### Frontend Components

**New Components:**
- `NotificationCenter.jsx` - Drawer showing all notifications (in-app + recent)
- `NotificationBell.jsx` - Badge showing unread count
- `NotificationPreferences.jsx` - Settings page for enabling/disabling channels
- `WaitlistStatusBanner.jsx` - Prominent banner on booking detail page showing current position
- `SmartNotificationTrigger.jsx` - Listens to WebSocket for real-time position updates

**WebSocket Integration:**
```javascript
// Real-time position updates via WebSocket
socket.on('waitlist:position_update', (data) => {
  // {bookingId, oldPosition, newPosition, status}
  if (data.status === 'CONFIRMED') {
    showNotification('✅ Your ticket is confirmed!', 'success');
  } else if (data.newPosition < data.oldPosition) {
    showNotification(`📍 Position improved to #${data.newPosition}`);
  }
});
```

### Third-Party Services

| Service | Purpose | Cost |
|---------|---------|------|
| SendGrid / AWS SES | Email delivery | Existing (likely) |
| Twilio / AWS SNS | SMS delivery | ₹0.50-1 per SMS (optional feature) |
| Firebase Cloud Messaging (FCM) | Push notifications | Free |
| Apache Kafka | Message queue | Existing (likely) |

### Notification Rules Engine

```javascript
const notificationRules = {
  'waitlist_position_improved': {
    trigger: (oldPos, newPos) => (oldPos - newPos >= 10),
    channels: ['email', 'sms', 'push', 'inapp'],
    template: 'waitlist_position_improved',
    frequency_cap: '1 per hour', // Max 1 such notification per hour
  },
  'waitlist_confirmed': {
    trigger: (status) => status === 'CONFIRMED',
    channels: ['email', 'sms', 'push', 'inapp'],
    template: 'waitlist_confirmed',
    frequency_cap: 'once',
    priority: 'high',
  },
  'pre_travel_reminder': {
    trigger: (status, hoursUntilTravel) => (hoursUntilTravel <= 24),
    channels: ['email', 'sms', 'push', 'inapp'],
    template: 'pre_travel_reminder',
    frequency_cap: '1 per day',
  },
  'auto_cancelled': {
    trigger: (status) => status === 'AUTO_CANCELLED',
    channels: ['email', 'sms', 'inapp'],
    template: 'auto_cancelled',
    frequency_cap: 'once',
  }
};
```

---

## Success Metrics

| Metric | Current | Target | Measurement |
|--------|---------|--------|-------------|
| Manual status checks per user | 12/day | 1/day | Analytics: API calls to `/my-bookings` |
| Notification opt-in rate | 0% (no channels) | 70% SMS + 80% email | Preference page analytics |
| Waitlist confirmation awareness | 0% real-time | 95% notified | Notification delivery logs |
| User satisfaction (waitlist) | Low (unknown) | High | NPS survey post-confirmation |
| Booking confirmation rate | Current baseline | +15% | Conversion tracking |
| Customer support queries (waitlist) | High | -40% reduction | Support ticket analysis |
| SMS engagement rate | N/A (new) | 40% click-through | Link tracking in SMS |
| Push notification click rate | N/A (new) | 25% | Firebase analytics |

---

## Edge Cases & Constraints

### Edge Cases

1. **User's Ticket Confirmed While App is Closed**: User misses the notification
   - **Handling**: Show prominent banner on next app open
   - **Storage**: Store "unread important notifications" in DB

2. **SMS Delivery Fails** (telecom network issue): Message not delivered
   - **Handling**: Retry logic (3 retries with exponential backoff); fallback to email
   - **Tracking**: Log failed SMS in database

3. **Duplicate Notifications**: Multiple channels send same message
   - **Handling**: De-duplication: if SMS sent, don't send push for same event
   - **Time window**: 5 minutes to consider as duplicate

4. **User Wants to Opt-Out of SMS** (expensive in India): Mid-journey opt-out
   - **Handling**: Immediate opt-out; mark as "SMS disabled"
   - **Notification**: "SMS notifications disabled. Use email instead."

5. **Waitlist Position Fluctuates Rapidly** (position goes 47 → 35 → 45 → 30)
   - **Handling**: Send notification only on net improvement (start=47, final=30)
   - **Batching**: Collect changes over 1 minute, then send single notification

### IRCTC-Specific Constraints

- **Indian Telecom Regulations**: SMS must include opt-out instructions (e.g., "Reply STOP")
- **Phone Number Format**: Handle various +91, 91, 0 formats for Indian numbers
- **Capacity**: System handles 300K waitlist bookings daily; 300K potential notifications
- **Cost Optimization**: SMS is expensive; prioritize based on confirmation likelihood
- **Time Zones**: Users across India; send notifications in local time
- **Festival Seasons**: 5x surge in notifications during Diwali, summer holidays

---

## Rollout Plan

**Phase 1 (Week 1)**: Deploy in-app notifications only (lowest risk)
- No external dependencies (no SMS, email infrastructure)
- Measure: In-app notification engagement
- Cost: $0

**Phase 2 (Week 2)**: Deploy email notifications (using existing SendGrid/SES)
- Should be seamless integration
- 50% rollout

**Phase 3 (Week 3)**: Deploy push notifications (Firebase)
- Requires mobile app integration
- Only for users with mobile app installed
- 100% rollout for mobile users

**Phase 4 (Week 4)**: Deploy SMS (Twilio/SNS)
- Highest cost; requires opt-in
- Target power users first (those who check frequently)
- Measure: SMS opt-in rate
- Rollback if adoption <20%

**Full Rollout**: Week 5 (all channels, all users)

---

## Related Problems Solved

- **Problem 4 (Waitlist Notifications)**: ✅ Directly solves
- **Problem 6 (Refund Tracking)**: Partial; same notification infrastructure reusable
