# Day 11 - Understanding Tokens, Tokenization and Context Windows in Large Language Models

## Date

6 July 2026

## Topics Covered

- Tokens and Tokenization
- Tokenizer
- Input and Output Tokens
- Context Window
- Token-Based Processing
- Importance of Prompt Optimization

## Learning Summary

On the eleventh day of the training, our mentor introduced us to the fundamental concepts that govern how Large Language Models (LLMs) process and understand text. Before building AI-powered applications, it is essential to understand how these models interpret user input internally.

The session began with **Tokens and Tokenization**. We learned that LLMs do not process complete words or sentences directly. Instead, every piece of text is divided into smaller units called **tokens**. A tokenizer converts human-readable text into tokens before sending it to the model for processing. The mentor explained that a single word may consist of one or more tokens depending on the language and vocabulary used by the model.

We also discussed the importance of token count in AI applications. Since API usage is measured in tokens, both the input prompt and the generated response contribute to overall token consumption. The mentor emphasized that writing concise prompts helps reduce unnecessary token usage, lowers API costs, and improves response efficiency.

The second major topic covered during today's session was the **Context Window**. We learned that the context window represents the maximum amount of information an LLM can consider while generating a response. This includes system instructions, conversation history, user prompts, retrieved information, and the model's previous responses.

The mentor explained that although modern LLMs support increasingly larger context windows, they still have practical limitations. When the available context exceeds the model's capacity, older information may no longer be considered during response generation. This limitation highlights the importance of efficient prompt design and retrieval techniques when working with large documents or long conversations.

Overall, today's session focused on building a strong conceptual understanding of how Large Language Models process information internally. These concepts provide the foundation for understanding more advanced topics such as Retrieval-Augmented Generation (RAG), semantic search, and vector databases.

## Key Takeaways

- LLMs process text as tokens rather than complete words.
- Tokenizers convert human-readable text into machine-processable tokens.
- Both input prompts and generated responses consume tokens.
- Token count directly influences API usage and processing efficiency.
- The context window defines how much information an LLM can process at one time.
- Efficient prompt design helps maximize the effective use of the available context window.

## Outcome

By the end of the eleventh day, I gained a clear understanding of tokens, tokenization, and context windows in Large Language Models. These concepts helped me understand how AI systems process text internally and why efficient prompt design is important for developing scalable and cost-effective AI applications.
