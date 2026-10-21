# Day 16 - Core Architecture, Data Modeling & Authentication

## Date

13 July 2026

## Topics Covered

- Project Architecture
- Django Backend Setup
- Patient Data Modeling
- Medication Cabinet
- Safety Assessment History
- Proactive Alerts
- JWT Authentication
- User Data Isolation
- Backend Structure
- Challenges Faced
- Core Backend Foundation

## Learning Summary

On the sixteenth day of the training, the focus was on establishing the **core backend architecture of the MedGuardian AI project**. Before implementing advanced AI and clinical intelligence features, it was important to create a structured backend capable of securely managing patient information, medications, safety assessments, and alerts.

The backend was developed using **Django and Django REST Framework (DRF)**. The project was organized into separate modules so that different responsibilities could be maintained independently.

The main objective of this stage was to create a reliable foundation for the later **Medication Digital Twin** workflow.

### Backend Architecture

```text
                    MedGuardian AI
                          │
                          ▼
                 Django REST Backend
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
      Accounts         Patients       Medications
          │               │               │
          └───────────────┼───────────────┘
                          ▼
                 Safety Assessment
                          │
              ┌───────────┴───────────┐
              ▼                       ▼
          History                  Alerts
```

---

The first step was to create the basic Django project structure and divide the backend into logical applications.

### Project Structure

```text
MedGuardian/
│
├── accounts/
├── patients/
├── prescriptions/
├── reports/
├── services/
│
├── manage.py
└── requirements.txt
```

Separating the application into modules makes the project easier to understand, maintain, and extend as new healthcare and AI features are added.

---

## Patient Profile Modeling

A central component of the system was the **PatientProfile** model.

The model was designed to store important patient and clinical information that can influence medication safety.

Some of the important parameters include:

```text
Age
Gender
Weight
eGFR
Serum Creatinine
ALT / AST
Allergies
Chronic Conditions
Pregnancy Status
Lactation Status
```

These parameters are important because medication safety depends not only on the drug itself but also on the patient's current clinical condition.

For example, changes in renal function may affect how certain medications should be evaluated.

### Patient Data Flow

```text
Patient Information
        │
        ▼
PatientProfile
        │
        ├── Demographics
        ├── Clinical Parameters
        ├── Allergies
        ├── Conditions
        └── Pregnancy/Lactation
```

---

## Medication Cabinet

A **MedicationCabinet** model was also introduced to maintain the patient's active medications.

The medication cabinet stores information such as:

- Medication name
- Dosage
- Frequency
- Administration route
- Active/discontinued status
- Prescribing information

The purpose of this component is to maintain a continuously updated representation of the medications currently associated with a patient.

This later becomes an important part of the **Medication Digital Twin** concept.

### Medication Cabinet Flow

```text
Prescription / Manual Entry
            │
            ▼
     Medication Cabinet
            │
      ┌─────┴─────┐
      ▼           ▼
   Active     Discontinued
 Medication    Medication
```

---

## Safety Assessment History

Another important part of the backend was maintaining a history of previous medication safety evaluations.

Instead of storing only the latest result, the system records historical assessments.

The risk levels were organized as:

```text
Safe
  ↓
Low
  ↓
Moderate
  ↓
Severe
```

Maintaining historical assessments is useful because the system can later compare a patient's current risk with previous evaluations and identify whether their overall safety status has changed.

---

## Proactive Alerts

The `ProactiveAlert` model was introduced to store important safety notifications.

These alerts can later be generated when the system identifies a meaningful deterioration in the patient's medication safety profile.

The basic concept is:

```text
Patient / Medication Change
            │
            ▼
      Safety Evaluation
            │
            ▼
      Risk Assessment
            │
            ▼
      Proactive Alert
```

This architecture prepares the project for the event-driven alerting system that will be implemented in the upcoming stages.

---

## JWT Authentication

Since MedGuardian handles sensitive patient-related information, authentication was an important part of the backend foundation.

**SimpleJWT** was integrated with Django REST Framework to provide token-based authentication.

The basic workflow is:

```text
User Login
    │
    ▼
Authentication
    │
    ▼
JWT Access Token
    │
    ▼
Authenticated API Request
    │
    ▼
Protected Backend Resource
```

This ensures that protected API endpoints are accessible only to authenticated users.

---

## User Data Isolation

Another important requirement was ensuring that users cannot access another user's patient information.

Query filtering and custom permission logic were introduced to maintain user-level data isolation.

Conceptually:

```text
User A
  │
  └── Patient Data A ✓

User B
  │
  └── Patient Data B ✓

User A ─────X─────> Patient Data B
User B ─────X─────> Patient Data A
```

This is particularly important for a healthcare-oriented application where patient information must be handled carefully.

---

## Handling Missing Clinical Information

During the backend design, one challenge was handling incomplete patient information.

For example, a patient may have a serum creatinine value available while another clinical parameter such as eGFR may not be available.

The system therefore needed to handle missing values without causing the complete safety evaluation process to fail.

Validation and fallback logic were introduced so that available information could still be processed appropriately.

This helped establish a more robust foundation for the clinical evaluation layer.

---

## Challenges Faced

- Designing a backend structure that could support future AI and clinical modules.
- Deciding which patient and medication parameters should be stored.
- Handling missing or incomplete clinical information.
- Implementing authentication for protected healthcare-related APIs.
- Maintaining proper separation between users and their patient data.
- Designing safety history so that previous assessments could be compared with future evaluations.

## Key Takeaways

- Django and Django REST Framework can be used to build a modular backend for healthcare applications.
- Proper data modeling is important before implementing advanced AI functionality.
- Patient information and medication information should be represented as separate but connected entities.
- Maintaining historical safety assessments enables future risk-change detection.
- JWT authentication provides a structured way to protect API endpoints.
- User-level data isolation is essential when handling patient-related information.
- Designing the backend carefully at an early stage makes future features easier to integrate.

## Outcome

By the end of the sixteenth day, I successfully understood and implemented the **core backend foundation of MedGuardian AI**.

The project now had structured models for patients, medications, safety history, and proactive alerts along with JWT-based authentication and user-level data isolation.

This foundation prepared the project for the next stage, where **AI-based prescription processing and clinical intelligence providers** would be integrated into the system.

