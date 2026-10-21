# Day 16 - System Architecture, Clinical Domain Modeling & Secure Authentication

## Date

13 July 2026

## Topics Covered

- MedGuardian System Architecture
- Django Backend Scaffolding
- Django REST Framework
- Clinical Domain Modeling
- Patient Profile
- Medication Cabinet
- Safety Assessment History
- Proactive Alerts
- Renal Biomarkers
- JWT Authentication
- User Data Isolation
- Cockcroft-Gault Renal Clearance
- Database Migration
- Challenges Faced

## Learning Summary

On the sixteenth day of the training, we started the core backend engineering work for the **MedGuardian AI Proactive Medication Digital Twin**.

The main objective of the day was to establish the foundational backend architecture and create the relational data models required to represent patients, medications, clinical safety assessments, and proactive alerts.

The backend was developed using **Django 4.2 and Django REST Framework**, with the architecture divided into separate applications according to their responsibilities.

### Repository and Backend Scaffolding

The Django backend was organized into modular domain applications:

```text
MedGuardian_AI/
│
└── backend/
    ├── accounts/
    ├── patients/
    ├── services/
    ├── prescriptions/
    ├── reports/
    ├── medguardian/
    └── manage.py
```

The major responsibilities were separated into different applications instead of placing the complete system inside a single Django app.

Environment configuration was also separated using `python-dotenv` so that sensitive information such as API keys, database URLs, and Celery broker credentials could be managed through environment variables.

### Clinical Domain Modeling

The most important implementation work was performed in `patients/models.py`.

Four major entities were designed:

- `PatientProfile`
- `MedicationCabinet`
- `SafetyAssessmentHistory`
- `ProactiveAlert`

These models form the core clinical data layer of the application.

### Patient Profile

The `PatientProfile` model was designed to store both basic patient information and clinically relevant parameters.

The model includes:

- Name
- Age
- Gender
- Weight
- Serum creatinine
- eGFR
- Allergies
- Chronic conditions
- Pregnancy status
- Other clinical information

A simplified implementation was:

```python
class PatientProfile(models.Model):
    user = models.OneToOneField(
        User,
        on_delete=models.CASCADE,
        related_name='profile'
    )

    name = models.CharField(max_length=255)

    age = models.PositiveIntegerField(
        validators=[
            MinValueValidator(0),
            MaxValueValidator(125)
        ]
    )

    gender = models.CharField(
        max_length=20,
        choices=[
            ('male', 'Male'),
            ('female', 'Female'),
            ('other', 'Other')
        ]
    )

    creatinine = models.FloatField(
        null=True,
        blank=True
    )

    egfr = models.FloatField(
        null=True,
        blank=True
    )

    allergies = models.JSONField(
        default=list,
        blank=True
    )

    chronic_conditions = models.JSONField(
        default=list,
        blank=True
    )

    is_pregnant = models.BooleanField(
        default=False
    )

    updated_at = models.DateTimeField(
        auto_now=True
    )
```

The model was intentionally designed to allow some clinical measurements to be optional because a patient may not always have recent laboratory results.

### Renal Function and Cockcroft-Gault Calculation

Since renal function is important for medication safety, the patient model also included a renal clearance calculation.

When eGFR was unavailable but sufficient information such as age, weight, creatinine, and gender was available, the system could estimate creatinine clearance using the **Cockcroft-Gault equation**.

```python
@property
def estimated_crcl(self):
    """Cockcroft-Gault Equation estimation if eGFR is missing."""

    if self.creatinine and self.age and self.weight_kg:
        constant = (
            0.85
            if self.gender.lower() == 'female'
            else 1.0
        )

        return round(
            ((140 - self.age) * self.weight_kg /
             (72 * self.creatinine)) * constant,
            2
        )

    return self.egfr
```

This provided a fallback mechanism when laboratory information was incomplete.

### Medication Cabinet

The `MedicationCabinet` model was designed to maintain the patient's active medication regimen.

It tracks information such as:

- Medication name
- Brand name
- Generic name
- Dosage strength
- Administration route
- Active or discontinued status
- Prescribing clinician

This creates a structured medication record that can later be used by the clinical safety engine.

```text
Patient
   │
   ├── Clinical Profile
   │      ├── Age
   │      ├── eGFR
   │      ├── Creatinine
   │      ├── Allergies
   │      └── Conditions
   │
   └── Medication Cabinet
          ├── Medication A
          ├── Medication B
          └── Medication C
```

### Safety Assessment History

The `SafetyAssessmentHistory` model was introduced to maintain a historical record of clinical safety evaluations.

It stores information such as:

- Risk score
- Risk level
- Flagged contraindications
- Clinical rationale
- Historical assessment information

The risk levels were structured as:

```text
Safe
  │
  ▼
Low
  │
  ▼
Moderate
  │
  ▼
Severe
```

This historical structure would later support the risk timeline and proactive alerting features of the system.

### Proactive Alerts

The `ProactiveAlert` model was designed to store safety alerts generated by the clinical engine.

The alert system supports categories such as:

- Drug-drug interaction
- Allergy
- Organ impairment

It also maintains alert state and acknowledgment information so that users can distinguish between new and already-reviewed alerts.

### JWT Authentication and User Isolation

Security was another major part of the day's implementation.

JWT-based authentication was implemented using `djangorestframework-simplejwt`.

The system uses:

- Short-lived access tokens
- Refresh tokens
- Custom DRF permissions
- User-specific query filtering

The access token lifetime was configured for 60 minutes, with rotating refresh tokens.

Custom permission handling such as `IsOwnerOrClinician` was introduced to ensure that users could access only the clinical records they were authorized to view.

The backend also overrides `get_queryset()` where required to enforce strict user-level data isolation.

The authentication flow can be represented as:

```text
User
 │
 ▼
Login / Authentication
 │
 ▼
JWT Access + Refresh Tokens
 │
 ▼
Authenticated API Request
 │
 ▼
Permission Check
 │
 ▼
User-Specific Clinical Data
```

### Handling Missing Clinical Information

One of the challenges identified during implementation was that clinical risk calculations could fail if important laboratory information was missing.

For example, a patient might not have a recent creatinine value.

Instead of allowing missing values to cause failures in downstream calculations, fallback logic was introduced through properties such as `estimated_crcl`.

This allowed the system to make a controlled calculation when sufficient information was available and otherwise fall back to the available eGFR value.

---

## Challenges Faced

- Designing relational models capable of representing complex clinical information.
- Handling optional patient laboratory values without breaking downstream calculations.
- Representing active and discontinued medications separately.
- Maintaining historical safety assessments.
- Designing a proactive alert structure for different types of clinical risks.
- Implementing authentication while maintaining strict patient data isolation.
- Providing a renal function fallback when eGFR or other laboratory information was unavailable.

## Key Takeaways

- A healthcare application requires carefully designed clinical data models.
- Patient information and medication information should be stored as structured data rather than unorganized text.
- Renal biomarkers such as creatinine and eGFR are important inputs for medication safety.
- Missing clinical information needs to be handled safely through controlled fallback logic.
- JWT authentication provides secure stateless authentication for the REST API.
- User-level query filtering is essential for protecting patient-specific medical data.
- Separating clinical models into independent entities makes the system easier to extend.

## Outcome

By the end of the sixteenth day, the foundational backend structure of MedGuardian was established using **Django 4.2 and Django REST Framework**.

The core patient, medication, safety history, and proactive alert models were designed, database migrations and seed data were prepared, and JWT authentication with user-level data isolation was implemented.

This foundation provided the required backend structure for the next stages of MedGuardian, including **prescription vision processing, clinical medication providers, and automated safety analysis**.
