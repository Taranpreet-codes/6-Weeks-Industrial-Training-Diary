
# Day 21 – Grounded Clinical Chat Assistant & Strict Anti-Hallucination Framework

**Date:** 20 July 2026  

### Daily Objective

The objective of Day 21 was to develop an **evidence-grounded conversational clinical assistant** using **Gemini 2.0 Flash**. The assistant was designed to help clinicians and patients by generating responses strictly from indexed clinical guidelines and the patient's available profile data, while preventing unsupported medical claims and hallucinations.

### Technical Architecture & Implementation

#### 1. Retrieval-Augmented Clinical Assistant

The clinical chat workflow was implemented in:

`services/clinical_service.py`

When a user submits a clinical query, the system first performs a **semantic search against the ChromaDB knowledge base** developed earlier. Relevant excerpts from indexed clinical guidelines are retrieved and used as evidence for generating the response.

The patient's active digital twin information, including:

- Age
- Pregnancy status
- eGFR
- Current medications

is combined with the retrieved guideline evidence before sending the request to the language model.

This ensures that the generated response considers both **patient-specific context** and **verified clinical evidence**.

#### 2. Strict Anti-Hallucination Prompt Engineering

A strict system prompt was created for Gemini 2.0 Flash with explicit safety constraints.

The model was instructed to:

- Use only the provided clinical guidelines and patient profile.
- Refuse to speculate when sufficient evidence is unavailable.
- Provide a clear message when the indexed guidelines cannot safely answer a query.
- Include a source citation for every medical claim.
- Never assume normal organ function when abnormal laboratory values are available.

The required citation format was:

`[Source: <DocName>, Page <P>]`

This made the generated responses more transparent and easier to verify against the original clinical sources.

#### 3. Conversational API Endpoint

A dedicated endpoint was exposed through:

`POST /api/assistant/chat/`

The endpoint accepts:

- User queries
- Conversation history

and returns a structured response containing:

- Generated clinical answer
- Retrieved and cited evidence
- Evidence confidence score

This provided a clean backend interface for connecting the clinical assistant with the frontend application.

### Key Code Implementation

```python
# services/clinical_service.py - Grounded Clinical Synthesis

SYSTEM_PROMPT = """You are MedGuardian AI, a Clinical Decision Support Assistant.

CRITICAL SAFETY DIRECTIVE:
1. Ground your answer EXCLUSIVELY in the provided Clinical Guidelines and Patient Profile.
2. If the guidelines do not contain explicit evidence to answer the question, state:
   "I cannot find sufficient evidence in the indexed clinical guidelines to safely answer this query. Please consult your physician."
3. Every medical claim must cite its source in brackets:
   [Source: <Doc>, Page <P>].
4. Never assume normal organ function when lab values are explicitly flagged as abnormal.
"""

def generate_grounded_response(
    user_query: str,
    patient_profile: dict,
    retrieved_evidence: list
) -> dict:

    context_block = "\n---\n".join(
        [
            f"Source: {e['metadata']['source']}\n{e['text']}"
            for e in retrieved_evidence
        ]
    )

    prompt = f"""
    Patient Profile: {patient_profile}

    Evidence Context:
    {context_block}

    User Question: {user_query}
    """

    response = client.models.generate_content(
        model='gemini-2.0-flash',
        contents=prompt,
        config={
            "system_instruction": SYSTEM_PROMPT,
            "temperature": 0.2
        }
    )

    return {
        "reply": response.text,
        "evidence_used": retrieved_evidence
    }
```

### Challenge & Engineering Solution

**Challenge:**  
Generic LLM safety filters can sometimes reject valid clinical questions because of overly broad safety restrictions, making the assistant less useful.

**Solution:**  
Instead of relying only on generic safety filtering, the assistant was designed around a **grounded clinical decision-support framework**. Explicit guideline evidence, patient context, mandatory citations, and clear uncertainty handling were incorporated into the system prompt.

This approach allowed the model to provide useful answers when supporting evidence was available while refusing to generate unsupported clinical information.

### Outcome

By the end of Day 21, a **context-grounded clinical chat assistant** was implemented with:

- ChromaDB-based evidence retrieval
- Patient-specific context integration
- Gemini 2.0 Flash generation
- Strict anti-hallucination instructions
- Mandatory clinical source citations
- Evidence confidence information
- Safe refusal when sufficient evidence is unavailable

### Git Commit

```bash
git commit -m "feat(chat): implement evidence-grounded clinical assistant with
