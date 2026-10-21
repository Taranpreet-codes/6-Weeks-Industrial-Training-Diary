
# Day 22 – Asynchronous Event-Driven Digital Twin

**Date:** 21 July 2026  

### Daily Objective

The objective of Day 22 was to transform the system into a proactive, event-driven **Medication Digital Twin**. The system was designed to automatically detect changes in patient information or medications and trigger safety re-evaluation in the background using **Django Signals, Celery, and Redis**.

### Technical Architecture & Implementation

#### 1. Event-Driven Signal Dispatcher

The event detection mechanism was implemented in:

`patients/signals.py`

Django `post_save` and `post_delete` signals were connected to:

- `PatientProfile`
- `MedicationCabinet`

Whenever important patient or medication information changed, the system automatically triggered an asynchronous safety audit.

For example, a significant biomarker change such as an **eGFR falling below 30 mL/min**, or adding/deactivating a medication, could automatically initiate a new safety evaluation.

This removed the need for users to manually request a safety check after every change.

#### 2. Celery Worker and Redis Task Queue

Celery was configured through:

`medguardian/celery.py`

and the task implementation was maintained in:

`patients/tasks.py`

The system used:

- **Celery 5.3** for asynchronous task processing
- **Redis 5.0** as the message broker and result backend

A task named:

`re_evaluate_patient_safety_task(patient_id)`

was created to perform comprehensive patient safety evaluations outside the main HTTP request.

This kept the main application responsive while potentially expensive safety checks were processed in the background.

#### 3. Atomic Redis Debouncing

A **10-second Redis debounce mechanism** was implemented to prevent repeated safety evaluations when multiple changes occurred within a short period.

A Redis key lock was created using the pattern:

`medguardian:debounce:{patient_id}`

For example, if a user updated several medications within a few seconds, each update could normally generate a separate safety evaluation. Instead, the debounce mechanism collapsed these rapid triggers into a single deferred execution.

This helped:

- Reduce unnecessary Celery tasks
- Prevent worker queue congestion
- Reduce repeated external API requests
- Avoid downstream API rate-limit issues

For local development environments where Redis was unavailable, synchronous execution was enabled using:

`CELERY_TASK_ALWAYS_EAGER=True`

### Key Implementation

```python
# patients/tasks.py - Debounced Asynchronous Safety Audit

from celery import shared_task

@shared_task
def re_evaluate_patient_safety_task(patient_id):
    # Perform comprehensive patient safety evaluation
    ...
```

The task was triggered automatically through Django model signals whenever relevant patient or medication data changed.

### System Flow

```text
Patient / Medication Change
          |
          v
   Django Model Signal
   post_save / post_delete
          |
          v
   Redis Debounce Lock
          |
          v
      Celery Queue
          |
          v
Background Safety Evaluation
          |
          v
Updated Patient Safety State
```

### Engineering Considerations

The main engineering challenge was preventing **multiple safety evaluations from being triggered by rapid consecutive updates**.

The Redis-based debounce lock provided a distributed mechanism for grouping these changes within a short time window. This improved system efficiency while still ensuring that the patient's latest state was evaluated.

Another consideration was local development. Since Redis may not always be available during development, the system included a synchronous fallback using Celery's eager execution mode.

### Outcome

By the end of Day 22, the system had been converted from a purely request-driven architecture into a **proactive event-driven Medication Digital Twin**.

The implementation provided:

- Automatic detection of patient and medication changes
- Django signal-based event dispatching
- Asynchronous safety evaluation using Celery
- Redis-based task management
- 10-second atomic debouncing
- Reduced duplicate safety evaluations
- Synchronous fallback for local development
