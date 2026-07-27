# Day 12 - Hallucinations, Embeddings and Introduction to Retrieval-Augmented Generation (RAG)

## Date

7 July 2026

## Topics Covered

- Hallucinations in Large Language Models
- Embeddings
- Semantic Search
- Fine-Tuning vs Retrieval-Augmented Generation (RAG)
- Conceptual RAG Pipeline

## Learning Summary

On the twelfth day of the training, our mentor introduced us to some of the core concepts that improve the reliability and accuracy of Large Language Models (LLMs). The session began with an explanation of **hallucinations**, one of the most common challenges faced while working with AI models.

We learned that hallucinations occur when an LLM generates information that is incorrect, misleading, or unsupported by factual evidence. The mentor explained that LLMs are probabilistic language models that predict the next most likely token rather than retrieving verified facts from a database. As a result, they may sometimes produce confident but inaccurate responses, especially when sufficient context is unavailable or when answering questions beyond their knowledge.

The mentor discussed several factors that contribute to hallucinations, including ambiguous prompts, outdated knowledge, insufficient context, and the absence of grounding information. We also learned that providing relevant external information significantly improves response accuracy and reduces the likelihood of hallucinations.

The session then focused on **Embeddings**, which convert text into high-dimensional numerical vectors that capture semantic meaning. Instead of comparing exact words, embeddings allow AI systems to compare the meaning of different pieces of text. We learned that semantically similar sentences produce vectors that are located close to one another in vector space, making semantic search possible.

Another important concept introduced today was **Semantic Search**. Unlike traditional keyword-based search, semantic search retrieves information based on meaning and context. This enables AI systems to locate relevant documents even when the exact keywords used in the query are not present in the document.

The mentor also compared **Fine-Tuning** and **Retrieval-Augmented Generation (RAG)**. We learned that fine-tuning modifies the internal weights of a language model to improve its behavior for specific tasks or domains. In contrast, RAG does not retrain the model. Instead, it retrieves relevant information from external knowledge sources during inference and provides that information to the LLM before generating a response. This makes RAG particularly suitable for applications that require access to frequently updated or domain-specific information.

Finally, we studied the conceptual workflow of a **Retrieval-Augmented Generation (RAG) Pipeline**. The mentor explained that a user's query is first converted into an embedding, after which a vector database retrieves the most relevant document chunks. These retrieved documents are then included in the model's context, allowing the LLM to generate a more accurate and grounded response.

Overall, today's session provided the theoretical foundation necessary for understanding how modern AI systems combine semantic retrieval with language models to improve response quality and minimize hallucinations.

## Key Takeaways

- Hallucinations occur when LLMs generate unsupported or inaccurate information.
- Providing relevant context helps reduce hallucinations.
- Embeddings represent text as numerical vectors based on semantic meaning.
- Semantic search retrieves information using meaning instead of exact keyword matching.
- Fine-Tuning modifies a model's behavior, while RAG retrieves external knowledge during inference.
- A RAG pipeline combines vector search with an LLM to produce more accurate and grounded responses.

## Outcome

By the end of the twelfth day, I gained a strong conceptual understanding of hallucinations, embeddings, semantic search, Fine-Tuning, and Retrieval-Augmented Generation (RAG). I also understood how these concepts work together to improve the reliability of AI systems and prepare the foundation for building practical RAG applications in the upcoming sessions.
