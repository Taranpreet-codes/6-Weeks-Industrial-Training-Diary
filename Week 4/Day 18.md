# Day 18 - Pluggable Clinical Knowledge Providers (NLM RxNorm & Curated DDI Engine)

## Date
15 July 2026

## Topics Covered
- Pluggable clinical provider architecture
- NLM RxNorm and RxNav API integration
- Drug name normalization and RxCUI mapping
- Local LRU caching for frequent drug lookups
- Curated Drug-Drug Interaction (DDI) engine
- Offline-first clinical safety screening
- Deterministic interaction severity classification

## Learning Summary

### 1. Pluggable Clinical Provider Architecture

The third day focused on making the clinical knowledge layer modular and independent from the main business logic. A common abstract interface, `BaseClinicalProvider`, was created inside `services/providers/`.

The provider interface defined three core operations:

```python
normalize_medication()
check_interactions()
get_safety_summary()
```

This architecture allowed external clinical data sources and internal safety logic to be updated independently. It also made the providers easier to mock and test without depending on live services.

### 2. NLM RxNorm Drug Normalization

The `RxNavNormalizerProvider` was implemented to connect with the National Library of Medicine's RxNorm REST API.

The system used:

```text
/REST/approximateTerm.json
/REST/rxcui.json
```

The purpose was to convert different medication names into a standard representation.

For example:

```text
Glucophage  →  metformin
Prinivil    →  lisinopril
Lasix       →  furosemide
```

Along with the canonical drug name, the system retrieved the corresponding **RxCUI**, which provides a standardized identifier for the medication.

A local LRU cache was also added so that frequently requested medications did not require repeated API calls.

```python
@lru_cache(maxsize=1024)
def normalize_drug_name(self, query_name: str) -> dict:
    ...
```

This improved lookup performance and reduced unnecessary network requests.

### 3. Curated Drug-Drug Interaction Engine

Instead of depending completely on an external interaction API, a deterministic offline DDI provider was implemented.

The curated dataset:

```text
curated_ddi_database.json
```

was compiled from sources including DDInter 2.0, ONCHigh, and FDA guidance.

Interactions were classified into four fixed severity levels:

```text
Contraindicated
Severe
Moderate
Minor
```

The system also stored the clinical mechanism and management recommendation associated with an interaction.

For example:

```text
Drug Combination
      ↓
Interaction Match
      ↓
Severity Classification
      ↓
Clinical Mechanism
      ↓
Management Recommendation
```

This made the interaction results more consistent and explainable instead of depending on unpredictable external responses.

## Challenges Faced

- The public NLM RxNav Drug Interaction API had been officially decommissioned, which caused compatibility problems with older interaction-based implementations.
- Drug normalization and interaction checking needed to remain reliable even when an external API was unavailable.
- Repeated drug lookups could introduce unnecessary latency.

### Engineering Solutions

- NLM RxNav was used specifically for **drug name normalization and RxCUI mapping**.
- Drug interaction checking was separated from the NLM service and handled using the curated offline dataset.
- LRU caching was implemented for frequently requested drug names.
- A fallback response was added when the RxNorm service could not be reached.

## Key Takeaways

- Learned how to design modular provider-based clinical architectures.
- Understood the importance of standardizing medication names before performing safety checks.
- Learned how RxCUI identifiers can provide a consistent representation of medications.
- Implemented caching to improve repeated clinical data lookups.
- Understood why safety-critical interaction checking should have a reliable offline fallback.
- Learned to separate external data retrieval from deterministic clinical decision logic.

## Outcome

By the end of Day 18, the clinical knowledge layer had been successfully modularized. The system could normalize brand and trade medication names into canonical drug concepts and RxCUI identifiers while performing multi-level drug-drug interaction screening through an offline-first curated dataset.

The implementation provided a reliable foundation for the later clinical safety assessment and alert-generation modules.
