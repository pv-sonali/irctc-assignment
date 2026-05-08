# Feature Spec 6: Unified Refund & TDR Tracking Dashboard

## Problem Statement

When users cancel bookings or initiate refunds, they cannot easily determine refund status. The system provides no unified refund dashboard; users navigate multiple sections to find refund info. Status terminology is inconsistent ("Pending", "Processing", "Initiated", "Approved"). Refund policy text is dense legal prose. Users don't know if refund takes 2 days or 30 days. There's no timeline visibility or status updates. TDR (Train Delay Refund) process is a separate, undiscoverable flow. Users check refund status 5-8 times before receiving refund, creating frustration and support burden.

**Impact from Part A**: ~30-50 million annual refund/cancellation requests; users check ~6 times daily; no timeline visibility; policy confusion leads to support queries.

---

## Proposed Solution

Create a unified refund dashboard that consolidates all refund information in one place:

1. **Single Refund History Page**: All refunds/cancellations in one view
2. **Clear Status Timeline**: Visual timeline showing refund stages (Initiated → Processing → Approved → Paid)
3. **Consistent Terminology**: Standardized status labels ("Pending", "Approved", "Paid")
4. **Plain-Language Policy**: Replace legalese with visual flow diagrams
5. **Automatic Status Updates**: Notifications when refund stage changes (no manual checking)
6. **TDR Integrated**: TDR claims shown alongside regular refunds
7. **Refund Reason Tracking**: Show why refund was issued + amount breakdown
8. **Quick Actions**: "Request Status Update", "Contact Support" in-place

### User Experience (Proposed)

```
Current Refund Flow (Broken):
────────────────────────────
User cancels booking
  ↓
System: "Refund initiated. You'll receive within 7-14 days."
  ↓ (Vague timeframe, no process explanation)
  ↓
User must navigate:
  "My Account" → "Transactions" → "Refunds" (3 levels deep)
  ↓
Status shown: "Refund Pending"
  (No indication of current stage or progress)
  ↓
User checks again Day 5: Still "Refund Pending"
  (Is it still being processed? Did it fail? Unknown.)
  ↓
User calls support in frustration
  (Support: "It's in the approval stage, takes 7-14 days")
  ↓
User checks again Day 10: Finally "Refund Paid"
  (Relief, but wasted 10 days of uncertainty)


Proposed Refund Flow (Fixed):
────────────────────────────
User cancels booking
  ↓
System shows: "Refund initiated"
Immediate navigation to "Refund Dashboard"
  ↓
Unified Dashboard shows:
┌──────────────────────────────────────────────────┐
│ Refund Details - Delhi to Mumbai (PNR: 123456)   │
├──────────────────────────────────────────────────┤
│ Refund Amount: ₹5,500                            │
│ Status Timeline:                                 │
│                                                  │
│ [✓] Initiated    [✓] Approved   [⏳] Paid        │
│  May 8, 2:30 PM   May 9, 10 AM    Est. May 15   │
│                                                  │
│ Current Stage: ⏳ Approved                       │
│ Process: "We've approved your refund. It will   │
│ appear in your bank within 3-5 business days.   │
│ (May 12-16)"                                     │
│                                                  │
│ Amount Breakdown:                                │
│ Ticket Price:        ₹6,000                      │
│ Cancellation Fee:   -₹500                        │
│ Refund Amount:      ₹5,500                       │
│                                                  │
│ Refund Account:                                  │
│ Bank: HDFC Bank                                  │
│ Account: ••••••7890 (Updated 2 days ago)        │
│                                                  │
│ [Request Status Update] [Chat Support]          │
└──────────────────────────────────────────────────┘
  ↓
(Automatic email: "Your refund is in approved stage")
(Automatic SMS: "Approved. Will arrive in 3-5 days")
  ↓
Day 10: Automatic notification
"Your refund of ₹5,500 has been sent to your bank."
  ↓
User checks dashboard: Status updated to "Paid" ✓
  (No uncertainty, clear timeline followed)


TDR Claim Integration:

┌──────────────────────────────────────────────────┐
│ Refunds & Claims                                 │
├──────────────────────────────────────────────────┤
│ FILTER: [All] [Regular Refunds] [TDR Claims]    │
├──────────────────────────────────────────────────┤
│ 1. Regular Refund - Delhi to Mumbai              │
│    Status: Paid ✓ | Amount: ₹5,500 | May 8      │
│    ┌────────────────────────────────────────┐   │
│    │ [✓] Initiated [✓] Approved [✓] Paid   │   │
│    │ May 8     May 9     May 15 (Received)  │   │
│    └────────────────────────────────────────┘   │
│                                                  │
│ 2. TDR Claim - Mumbai to Bangalore (2.5 hr delay)│
│    Status: Under Review | Amount: ₹800 | May 5  │
│    ┌────────────────────────────────────────┐   │
│    │ [✓] Filed [⏳] Reviewed [ ] Approved  │   │
│    │ May 5    May 10       Est. May 20      │   │
│    └────────────────────────────────────────┘   │
│    Message: "Claim under review by Railway. TDR │
│    approvals take 20-30 days."                   │
│    [View TDR Policy] [Upload Proof]             │
│                                                  │
│ 3. Regular Refund - Bangalore to Chennai         │
│    Status: Processing | Amount: ₹3,200 | May 2  │
│    ┌────────────────────────────────────────┐   │
│    │ [✓] Initiated [⏳] Approved [ ] Paid  │   │
│    │ May 2     May 8 (1 day)     May 14    │   │
│    └────────────────────────────────────────┘   │
│    Message: "Being processed. Est. arrival:    │
│    May 14-18"                                   │
│                                                  │
│ [Load More]                                     │
└──────────────────────────────────────────────────┘
```

---

## Technical Implementation Plan

### Frontend Architecture

**New Pages & Routes:**
- `/refunds` - Main refund dashboard
- `/refunds/:refundId` - Detailed view for single refund
- `/refunds/history` - Historical refund list
- `/refunds/tdr` - TDR claims specific view

**Components:**
```
RefundDashboard.jsx (main page)
├── FilterBar.jsx (All / Regular / TDR)
├── RefundSummaryCard.jsx (quick stats: total refunded, pending)
├── RefundTimelineList.jsx (list of all refunds)
│   ├── RefundTimelineItem.jsx (single refund with status)
│   │   ├── StageTimeline.jsx (visual timeline stages)
│   │   ├── AmountBreakdown.jsx (ticket price - fee = refund)
│   │   ├── RefundAccountInfo.jsx (bank details)
│   │   └── ActionButtons.jsx (Request Update, Support)
│   └── TDRClaimItem.jsx (TDR-specific UI)
├── RefundFAQ.jsx (plain-language policy explanation)
└── RefundNotificationPreferences.jsx (when to notify user)
```

**State Management (Redux):**
```javascript
{
  refunds: {
    allRefunds: [
      {
        refundId: "ref_123",
        bookingId: "bk_456",
        route: "Delhi → Mumbai",
        originalAmount: 6000,
        cancellationFee: 500,
        refundAmount: 5500,
        refundType: "regular" | "tdr",
        status: "initiated" | "approved" | "paid" | "rejected",
        initiatedAt: "2026-05-08T14:30:00Z",
        approvedAt: "2026-05-09T10:00:00Z",
        paidAt: null,
        estimatedPayDate: "2026-05-15",
        refundAccount: {
          bankName: "HDFC Bank",
          accountNumber: "••••••7890", // Masked
          accountHolder: "Raj Kumar",
        },
        currentStage: "approved",
        stageDescription: "We've approved your refund. It will appear in your bank within 3-5 business days.",
      }
    ],
    selectedRefund: null,
  }
}
```

### Backend Changes

**New Database Tables:**

```sql
CREATE TABLE refund_tracking (
  refund_id VARCHAR(50) PRIMARY KEY,
  booking_id INT NOT NULL,
  user_id INT NOT NULL,
  refund_type ENUM('regular', 'tdr'),
  original_amount DECIMAL(10, 2),
  cancellation_fee DECIMAL(10, 2),
  refund_amount DECIMAL(10, 2),
  refund_status ENUM('initiated', 'approved', 'paid', 'rejected', 'failed'),
  
  initiated_at TIMESTAMP DEFAULT NOW(),
  approved_at TIMESTAMP NULL,
  paid_at TIMESTAMP NULL,
  estimated_pay_date DATE,
  
  cancellation_reason VARCHAR(255),
  tdr_reason VARCHAR(255), -- For TDR refunds
  
  refund_account_id INT,
  is_wallet_refund BOOLEAN, -- Refund to IRCTC wallet vs. bank
  
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (booking_id) REFERENCES bookings(booking_id),
  FOREIGN KEY (user_id) REFERENCES users(user_id),
  FOREIGN KEY (refund_account_id) REFERENCES refund_accounts(account_id),
  INDEX idx_user_booking (user_id, booking_id),
  INDEX idx_status (refund_status)
);

CREATE TABLE refund_status_history (
  history_id INT AUTO_INCREMENT PRIMARY KEY,
  refund_id VARCHAR(50),
  old_status VARCHAR(50),
  new_status VARCHAR(50),
  changed_at TIMESTAMP DEFAULT NOW(),
  changed_by VARCHAR(50), -- 'system' or user ID
  reason VARCHAR(255),
  FOREIGN KEY (refund_id) REFERENCES refund_tracking(refund_id),
  INDEX idx_refund_status (refund_id, changed_at)
);

CREATE TABLE refund_accounts (
  account_id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT NOT NULL,
  account_number_masked VARCHAR(50), -- ••••••7890
  bank_name VARCHAR(100),
  account_holder_name VARCHAR(100),
  is_default BOOLEAN,
  verified_at TIMESTAMP NULL,
  FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

### API Changes

| Method | Endpoint | Purpose | Request | Response |
|--------|----------|---------|---------|----------|
| GET | `/refunds` | Get all refunds | `?type=all\|regular\|tdr` | `{refunds: [...], summary}` |
| GET | `/refunds/:refundId` | Get single refund details | - | `{refund, statusHistory}` |
| POST | `/refunds/:refundId/request-update` | Request status update | - | `{success, message}` |
| GET | `/refunds/policies/plain-language` | Get simplified policy | - | `{sections: [{title, text, visual}]}` |
| POST | `/refunds/:refundId/contact-support` | Open support chat | `{message}` | `{ticketId}` |
| GET | `/refunds/statistics` | Get refund summary stats | - | `{total, pending, paid, average_days}` |

### Frontend Data Flow

**On Page Load:**
```javascript
useEffect(() => {
  // Fetch all refunds for current user
  dispatch(fetchAllRefunds());
  
  // Set up WebSocket listener for real-time updates
  socket.on('refund:status_changed', (data) => {
    // {refundId, oldStatus, newStatus}
    dispatch(updateRefundStatus(data.refundId, data.newStatus));
    showNotification(`Refund ${data.refundId} status updated to ${data.newStatus}`);
  });
}, [dispatch]);
```

### Third-Party Services

- **Stripe / Razorpay API**: To verify refund account details (optional)
- **Bank API**: To check refund payment status (if IRCTC has integration)
- **None mandatory** - most info comes from IRCTC's internal tracking

---

## Success Metrics

| Metric | Current | Target | Measurement |
|--------|---------|--------|-------------|
| User manual status checks | 6 per refund | 1 per refund | API calls to `/my-bookings` |
| Refund policy clarity | Low (dense text) | High | Post-refund survey: "Do you understand timeline?" |
| Support tickets (refund-related) | ~20% of total | ~5% | Support ticket categorization |
| Time to first refund insight | Unknown | <2 sec (page load) | Page load performance |
| TDR claim discoverability | Low (hidden) | 80% aware | Product analytics on TDR clicks |
| Refund satisfaction NPS | Low | +40 points | NPS survey sent post-refund |
| Refund page visit rate | N/A (no page) | 85% of refund users | Analytics: refund page visits |
| User confidence in refund | Low (uncertain) | 90% confident | Survey: "I know when my refund arrives" |

---

## Edge Cases & Constraints

### Edge Cases

1. **Refund Status Doesn't Update in Real-Time**: Backend delay in updating refund_tracking table
   - **Handling**: Show "Last updated 2 hours ago" timestamp
   - **Manual refresh**: "Request Status Update" button triggers manual API call to railway backend
   - **Auto-refresh**: Background job checks status every 6 hours

2. **Refund Amount is Different Than Expected**: User expected ₹6,000 but got ₹5,500
   - **Handling**: Show clear breakdown: Ticket (6000) - Cancellation Fee (500) = Refund (5500)
   - **Policy link**: "Why this fee?" links to plain-language policy explanation

3. **Refund Rejected**: Railway rejected TDR claim due to insufficient proof
   - **Handling**: Show reason clearly: "Claim rejected: No ticket attachment provided"
   - **Recovery**: "Upload proof & resubmit" button to retry
   - **Support**: "Contact support for appeal" option

4. **Refund to Multiple Accounts**: User changed bank account between booking and refund processing
   - **Handling**: Show current refund account; ask user if they want to update
   - **Verification**: New bank account must be verified before updating

5. **Refund Takes >30 Days**: Unusual delay
   - **Handling**: Alert user: "Your refund is taking longer than expected"
   - **Action**: "Contact support" button to investigate

### IRCTC-Specific Constraints

- **Railway Backend**: IRCTC refunds are processed by Railway backend; can't force faster processing
- **TDR Policy**: Railway TDR claims take 30-90 days; must manage user expectations
- **Refund Methods**: Refunds can go to bank account, wallet, or credit card; handle all three
- **Tax Implications**: Refunds affect user's refund wallet balance (used for future bookings)
- **Cancellation Policies**: Vary by train class (AC has different cancellation fee than Non-AC)

---

## Rollout Plan

**Phase 1 (Week 1)**: Deploy refund tracking page (read-only)
- Shows refund history with status timeline
- No policy changes; no behavior changes
- Cost: $0

**Phase 2 (Week 2)**: Deploy notifications
- Send email/SMS when refund status changes
- Use existing notification infrastructure
- 50% rollout

**Phase 3 (Week 3)**: Deploy plain-language policy
- Add "Why this fee?" explanations
- Add TDR policy guide
- A/B test against current dense policy text

**Phase 4 (Week 4)**: Deploy "Request Status Update" + support chat
- Allow users to request manual status check
- Show support chat inline
- Full rollout

**Full Rollout**: Week 5 (all features enabled for all users)

---

## Related Problems Solved

- **Problem 6 (Refund Tracking)**: ✅ Directly solves
- **Problem 4 (Waitlist Notifications)**: Uses same notification infrastructure
