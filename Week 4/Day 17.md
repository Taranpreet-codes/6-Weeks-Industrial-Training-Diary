# Day 17 - Multimodal Prescription Processing and Clinical Provider Integration

## Date

14 July 2026

## Topics Covered

- Multimodal Prescription Processing
- Gemini Vision Integration
- Structured Data Extraction
- Pydantic Validation
- Prescription Review Workflow
- RxNorm Medication Normalization
- Drug-Drug Interaction Checking
- openFDA Integration
- Clinical Provider Architecture
- Challenges Faced

## Learning Summary

On the seventeenth day of the training, we worked on one of the important parts of the MedGuardian system: **converting prescription images into structured medication information and using clinical data providers for medication safety analysis**.

The main objective was to allow the system to process a prescription image, identify the medicines mentioned in it, extract important details such as medication name, strength, dosage, and frequency, and then pass the extracted information to the clinical safety layer.

### Multimodal Prescription Processing

The system uses a multimodal AI approach to understand prescription images. A prescription can contain both visual information and medical text, so a vision-capable model was used to process the uploaded image.

The prescription upload workflow accepts supported image formats such as PNG, JPG, and WEBP.

The extracted information includes details such as:

- Medication name
- Strength
- Dosage
- Frequency
- Other available prescription instructions

The extracted information is then converted into a structured format instead of keeping it as unorganized text.

### Structured Extraction with Pydantic

To make the extracted information reliable and easier to process, Pydantic models were used to define the expected structure of the result.

A simplified representation of the structure is:

```python
class MedicationItem(BaseModel):
    name: str
    strength: str | None = None
    dosage: str | None = None
    frequency: str | None = None


class ExtractionResult(BaseModel):
    medications: list[MedicationItem]
```

This helped ensure that the output from the AI model followed a predictable structure before it was used by the rest of the application.

---

### Prescription Review Workflow

One important design decision was that extracted prescription information should **not immediately become part of the patient's medication cabinet**.

Instead, the system first shows the extracted medicines to the user for review.

```text
Prescription Image
        │
        ▼
   AI Vision Model
        │
        ▼
Structured Extraction
        │
        ▼
 Prescription Review
        │
   ┌────┴────┐
   │         │
 Reject    Confirm
             │
             ▼
     Medication Cabinet
```

This review step provides human verification before the information becomes part of the patient's medical record.

It also reduces the risk of an incorrectly interpreted prescription being stored automatically.

### Medication Normalization

After obtaining the medication name, the system needs to identify the medicine in a standardized way.

For this purpose, an **RxNorm-based normalization provider** was implemented. RxNorm is provided by the U.S. National Library of Medicine and helps connect medication names with standardized drug concepts.

The provider follows the general workflow:

```text
Extracted Medication Name
          │
          ▼
   RxNorm Normalization
          │
          ▼
 Standard Medication Concept
          │
          ▼
 Clinical Safety Analysis
```

A caching mechanism was also used so that repeated requests for the same medication did not always require another external request.

### Clinical Provider Architecture

The system was designed using separate clinical providers instead of placing all medication safety logic inside one large function.

The major providers include:

- `RxNavNormalizerProvider`
- `CompositeInteractionProvider`
- `CuratedDDIProvider`
- `OpenFDALabelProvider`
- `FallbackClinicalProvider`

This approach makes the clinical intelligence layer more modular and easier to maintain.

### Drug-Drug Interaction Checking

After medication information is normalized, the system can check for possible interactions between medicines.

The interaction layer combines information from multiple sources instead of depending on only one external source.

```text
Medication A ──┐
               ├──► Interaction Provider ──► Safety Result
Medication B ──┘
                         │
                  ┌──────┴──────┐
                  │             │
             Curated DDI     FDA Labels
```

The `CompositeInteractionProvider` allows multiple sources to contribute to the final medication safety assessment.

### openFDA Integration

The system also uses **openFDA** data to obtain official medication labeling information.

This can provide useful information related to:

- Drug warnings
- Contraindications
- Adverse reactions
- Medication safety information

The data is cached where appropriate to reduce unnecessary repeated requests and improve application responsiveness.

### Fallback Clinical Provider

External clinical services can sometimes become unavailable, respond slowly, or reach request limits.

To make the system more reliable, a fallback provider was included.

```text
Clinical Request
       │
       ▼
Primary Provider
       │
   ┌───┴────┐
Success    Failure
   │          │
   ▼          ▼
 Result    Fallback
              │
              ▼
         Safety Result
```

This makes the application less dependent on a single external service.

---

## Challenges Faced

- Extracting structured medication information from prescription images.
- Handling incomplete or unclear information in prescriptions.
- Ensuring that AI-generated extraction followed a predictable structure.
- Normalizing different medication names into standardized concepts.
- Managing dependencies on external clinical APIs.
- Combining information from different medication safety sources.
- Designing a reliable fallback mechanism when an external provider is unavailable.
- Making sure that extracted prescription data is reviewed before being stored.

## Key Takeaways

- Multimodal AI can be used to process prescription images and extract useful medical information.
- Structured validation is important when using AI-generated data.
- Human review is important before automatically storing extracted medical information.
- Medication normalization helps connect different drug names with standardized concepts.
- Clinical applications should not depend completely on a single external provider.
- A modular provider architecture makes the system easier to maintain and extend.
- Multiple clinical information sources can be combined to improve medication safety analysis.

## Outcome

By the end of the seventeenth day, we had developed a multimodal prescription processing workflow that could convert prescription images into structured medication information. The extracted medicines could then be reviewed, normalized, and passed to different clinical providers for further safety analysis.

This session helped us understand how **AI-based extraction, structured validation, medication normalization, external clinical data, and fallback mechanisms can work together in a healthcare application**.
