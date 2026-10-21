# Day 23 – Risk Escalation Engine & Real-Time Server-Sent Events

**Date:** 22 July 2026  

### Daily Objective

The objective of Day 23 was to develop a **risk escalation engine** that could identify changes in patient safety status and communicate important risk updates to the frontend in real time.

The implementation focused on connecting the backend safety evaluation system with a real-time notification mechanism so that newly generated clinical alerts could be delivered without requiring the user to manually refresh the application.

### Technical Architecture & Implementation

#### 1. Risk Escalation Engine

The risk evaluation workflow was extended to identify changes in the patient's safety condition after background safety assessments.

The system evaluates the latest patient and medication state and determines whether a new safety concern requires escalation.

Risk information can be associated with different clinical safety conditions, including:

- Drug-drug interactions
- Drug-allergy conflicts
- Organ impairment
- Other medication-related safety concerns

When a significant risk is identified, the system prepares an alert that can be communicated to the user.

#### 2. Real-Time Server-Sent Events

A **Server-Sent Events (SSE)** mechanism was implemented to provide real-time updates from the backend to the frontend.

Instead of repeatedly polling the backend for new alerts, the frontend can maintain an SSE connection and receive updates whenever the server publishes a new event.

The general communication flow was:

```text
Patient / Medication Update
          |
          v
Background Safety Evaluation
          |
          v
Risk Escalation Engine
          |
          v
Clinical Alert Generated
          |
          v
SSE Event Stream
          |
          v
Frontend Notification
```

This architecture allows safety notifications to reach the user with minimal delay.

### Real-Time Alert Handling

The real-time layer was designed around the existing `ProactiveAlert` system. Newly generated safety alerts could be communicated to the frontend so that important changes became visible without requiring a complete page reload.

This was particularly useful for a proactive clinical application because the user should not have to continuously check whether the patient's safety status has changed.

### Engineering Considerations

The main focus was maintaining a clear separation between:

1. **Safety evaluation** – determining the patient's current risk.
2. **Risk escalation** – identifying whether the result requires an alert.
3. **Real-time delivery** – communicating the alert to the frontend.
S
This separation keeps the clinical logic independent from the notification mechanism and makes the overall architecture easier to maintain and extend.

### Outcome

By the end of Day 23, the system had a real-time risk notification architecture connecting background safety evaluation with frontend alert delivery.

The implementation established:

- Risk escalation based on safety evaluation results
- Proactive clinical alert generation
- Server-Sent Events for real-time communication
- Backend-to-frontend alert delivery without continuous polling
- A clearer separation between clinical evaluation and notification handling

This provided the foundation for a more responsive clinical portal where important patient safety changes can be surfaced immediately.
