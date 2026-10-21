# Day 17 - Multimodal Prescription Vision Ingestion & Pydantic Medical NER

## Date

14 July 2026

## Topics Covered

- Prescription Image Upload
- Gemini 2.5 Flash Vision
- Multimodal Medical NER
- Prescription OCR
- Pydantic Schema Validation
- Structured Medication Extraction
- Human-in-the-Loop Review
- Prescription Staging
- API Integration
- Offline Mocking
- Challenges Faced

## Learning Summary

On the seventeenth day of the training, we implemented the **multimodal prescription processing pipeline** for MedGuardian.

The objective was to allow the system to accept scanned or photographed prescriptions and automatically extract structured medication information using **Google Gemini 2.5 Flash**.

Instead of requiring users to manually enter every medicine from a prescription, the system processes the prescription image, extracts medication entities, validates the extracted information, and then presents it for human verification before adding it to the patient's medication cabinet.

### Prescription Upload API

The first part of the implementation was the prescription upload endpoint.

A multipart upload API was created at:

```text
POST /api/prescriptions/upload/
```

The endpoint accepts common prescription image formats:

- PNG
- JPG
- WEBP

The upload size was limited to **10 MB**.

The backend also performs image buffering and generates a unique SHA-256 based filename before storing the uploaded prescription in:

```text
media/prescriptions/
```

This prevents filename conflicts and provides a consistent way of handling uploaded prescription files.

### Multimodal Gemini Vision Pipeline

The main AI component was implemented in:

```text
services/ingestion.py
```

An `IngestionPipeline` was created using the **Google GenAI SDK** and the `gemini-2.5-flash` model.

The model receives the prescription image together with an extraction instruction and identifies medication-related information from the image.

The workflow is:

```text
Prescription Image
       │
       ▼
Image Upload API
       │
       ▼
IngestionPipeline
       │
       ▼
Gemini 2.5 Flash
       │
       ▼
Medical Entity Extraction
       │
       ▼
Pydantic Validation
       │
       ▼
Structured Result
```

The purpose was not simply to generate a textual description of the prescription. The model was instructed to return structured medication information that could be consumed by the backend.

### Pydantic Medical NER Schema

To make the AI output predictable, strict Pydantic schemas were created.

The `MedicationItem` model defines the fields expected for each extracted medicine.

```python
class MedicationItem(BaseModel):
    name: str = Field(
        description="Brand or generic medication name"
    )

    dosage: str = Field(
        description="Dosage strength, e.g., 500mg or 10ml"
    )

    frequency: str = Field(
        description="Dosing schedule, e.g., Once daily, BID, Q8H"
    )

    route: Optional[str] = Field(
        default="Oral",
        description="Administration route"
    )

    instructions: Optional[str] = Field(
        default="",
        description="Prescriber instructions"
    )
```

A higher-level `ExtractionResult` model was also created:

```python
class ExtractionResult(BaseModel):
    raw_ocr_text: str = Field(
        description="Transcribed plain text from prescription"
    )

    medications: List[MedicationItem] = Field(
        description="Structured extracted medication items"
    )

    confidence_notes: Optional[str] = Field(
        description="Legibility or ambiguity notes"
    )
```

This structure allows the application to receive both the raw OCR text and structured medication entities while also recording possible ambiguity in the prescription.

### Information Extracted from Prescriptions

The Gemini pipeline was designed to extract information such as:

- Medication name
- Dosage strength
- Dosage form
- Frequency
- Administration route
- Prescriber instructions
- Raw prescription text
- Confidence or ambiguity notes

For example, a medication entry could contain:

```text
Medication: Example Medicine
Strength: 500 mg
Form: Tablet
Frequency: Twice Daily
Route: Oral
Instructions: As directed by physician
```

The structured representation makes the extracted information easier to validate and use in later clinical workflows.

### Human-in-the-Loop Review

An important part of the implementation was that AI-extracted medications were **not automatically added to the patient's medication cabinet**.

Instead, the prescription was first stored in a staging state:

```text
Prescription.status = "staged"
```

A confirmation endpoint was created:

```text
POST /api/prescriptions/{id}/confirm/
```

The workflow was therefore:

```text
Prescription Image
       │
       ▼
Gemini Vision Extraction
       │
       ▼
Pydantic Validation
       │
       ▼
Staged Prescription
       │
       ▼
User / Clinician Review
       │
   ┌───┴────┐
   │        │
Edit     Confirm
   │        │
   └───┬────┘
       ▼
Medication Cabinet
```

The reviewer can verify, modify, or deselect extracted medications before they become active medication records.

This provides an important human verification layer for AI-generated medical information.

### Handling Unclear Prescriptions

Prescription images can contain handwriting, poor image quality, partially visible medication names, or unclear dosage information.

A major challenge was therefore preventing the model from simply guessing when information was difficult to read.

The vision prompt was configured with a low temperature of **0.1** and instructed the model to identify ambiguous information through `confidence_notes` rather than inventing missing details.

The frontend staging workflow could then highlight information requiring user verification.

This created a safer workflow:

```text
Clear Information
       │
       ▼
Structured Extraction
       
Unclear Information
       │
       ▼
Confidence Note
       │
       ▼
Human Verification
```

### Development and Offline Mocking

Another implementation detail was the development fallback.

The prescription pipeline includes an automatic mock parser when the `GEMINI_API_KEY` is unavailable or when the application is running unit tests.

```text
Prescription Request
        │
        ▼
Gemini API Available?
     /        \
   Yes         No
   │            │
   ▼            ▼
Gemini       Mock Parser
   │            │
   └──────┬─────┘
          ▼
   Structured Result
```

This allowed the prescription workflow to be tested without requiring an active external API connection for every test run.

### Complete Prescription Processing Workflow

The complete implementation can be summarized as:

```text
User Uploads Prescription
          │
          ▼
Multipart Upload API
          │
          ▼
Image Validation & Storage
          │
          ▼
Gemini 2.5 Flash Vision
          │
          ▼
OCR + Medical NER
          │
          ▼
Pydantic Schema Validation
          │
          ▼
Prescription Staging
          │
          ▼
Human Review
          │
          ▼
Confirmation API
          │
          ▼
Medication Cabinet
```

This workflow connected the AI vision layer with the existing clinical data models created on Day 16.

---

## Challenges Faced

- Processing prescription images with different layouts and image quality.
- Extracting medication names and dosages from handwritten or unclear prescriptions.
- Preventing the vision model from guessing when information was ambiguous.
- Ensuring Gemini output followed a predictable structure.
- Validating AI-generated medical entities using Pydantic.
- Designing a review workflow before adding extracted medicines to the patient record.
- Making the system testable without requiring a live Gemini API connection.

## Key Takeaways

- Multimodal AI can process prescription images and extract structured medical entities.
- Gemini 2.5 Flash can combine image understanding with structured information extraction.
- Pydantic provides a reliable schema for validating AI-generated data.
- Medical AI systems should include human verification for uncertain extracted information.
- Staging extracted data before permanent storage provides an additional safety layer.
- API fallback and mocking are useful for reliable development and testing.
- Separating image ingestion from medication storage makes the system easier to maintain.

## Outcome

By the end of the seventeenth day, the **end-to-end prescription vision ingestion pipeline** was implemented.

MedGuardian could accept prescription images through the upload API, process them using **Gemini 2.5 Flash**, extract structured medication entities through Pydantic schemas, stage the results for human review, and confirm the verified medicines into the medication cabinet.

This implementation established the foundation for the next stage of the system: **normalizing extracted medications and performing clinical drug-safety checks through dedicated clinical providers**.
