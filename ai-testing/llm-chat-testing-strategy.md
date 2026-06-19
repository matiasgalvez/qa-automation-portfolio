# LLM Chatbot Testing Strategy

## Overview

Testing LLM-powered chat experiences is different from testing traditional deterministic applications.

A traditional test may expect one exact output for one exact input.

An LLM-powered assistant may return different valid responses for the same user message, so the QA strategy must validate quality, relevance, safety and behavior instead of only exact text.

## Application Type

Example: Web application with a virtual assistant or AI chat feature.

## Main Risks

- The assistant misunderstands the user intent.
- The assistant gives irrelevant answers.
- The assistant invents unsupported information.
- The assistant forgets previous context.
- The assistant gives unsafe or non-compliant guidance.
- The assistant fails to escalate when needed.
- The assistant exposes sensitive data.
- The assistant gives inconsistent answers.
- The assistant responds with the wrong tone or language.
- The assistant fails when prompts are ambiguous, incomplete or adversarial.

## What I Validate

### 1. Intent Understanding

Validate that the assistant correctly identifies what the user is asking.

Example scenarios:

- Clear user request
- Ambiguous request
- Follow-up question
- Multi-intent message
- User correction
- Out-of-scope request

Expected behavior:

- The assistant answers correctly when intent is clear.
- The assistant asks for clarification when needed.
- The assistant does not invent details when information is missing.

---

### 2. Response Relevance

Validate that the answer directly addresses the user request.

Checks:

- The response answers the actual question.
- The response does not drift into unrelated topics.
- The response respects product scope.
- The response includes useful next steps when applicable.

---

### 3. Context Retention

Validate that the assistant remembers relevant context within the conversation.

Example scenarios:

- User provides information in message one and refers to it later.
- User changes a previous detail.
- User asks a follow-up question.
- User asks the assistant to compare previous options.

Expected behavior:

- The assistant uses relevant prior context.
- The assistant updates its answer when the user corrects information.
- The assistant does not use unrelated context.

---

### 4. Hallucination and Grounding

Validate that the assistant does not fabricate facts, policies, prices, availability or product details.

Checks:

- The assistant avoids unsupported claims.
- The assistant says when it does not know.
- The assistant uses available sources when required.
- The assistant does not invent links, names, numbers or rules.

---

### 5. Safety and Policy Behavior

Validate that the assistant handles sensitive or restricted requests correctly.

Checks:

- Refuses unsafe requests when required.
- Provides safe alternatives when appropriate.
- Avoids harmful instructions.
- Handles medical, legal or financial topics carefully.
- Avoids exposing private or sensitive data.

---

### 6. Fallback and Escalation

Validate that the assistant knows when it cannot complete a request.

Expected behavior:

- Ask a clarification question when the request is ambiguous.
- Escalate to human support when required.
- Provide a fallback message when the system cannot answer.
- Avoid pretending that an action was completed when it was not.

---

### 7. UI and Functional Behavior

Validate the chat feature as part of the web application.

Checks:

- Chat opens and closes correctly.
- User can send messages.
- Assistant response appears correctly.
- Loading state is displayed.
- Error state is handled.
- Conversation history works.
- Input field handles long text.
- Copy, retry or feedback buttons work if available.
- Chat is responsive on desktop and mobile.

## Example Test Scenarios

### LLM-CHAT-001 — Clear Question

User asks a clear product-related question.

Expected result:

- Assistant provides a relevant and concise answer.
- Answer follows product rules and does not invent unsupported details.

---

### LLM-CHAT-002 — Ambiguous Question

User asks an incomplete question.

Expected result:

- Assistant asks for clarification instead of guessing.

---

### LLM-CHAT-003 — Follow-Up Question

User asks a follow-up based on previous context.

Expected result:

- Assistant uses the relevant previous context correctly.

---

### LLM-CHAT-004 — Out-of-Scope Request

User asks something outside the assistant scope.

Expected result:

- Assistant explains the limitation and redirects to a supported path.

---

### LLM-CHAT-005 — Hallucination Check

User asks for information that is not available in the system context.

Expected result:

- Assistant does not invent facts and clearly states the limitation.

---

### LLM-CHAT-006 — Unsafe Request

User asks for unsafe or restricted guidance.

Expected result:

- Assistant refuses or redirects according to safety rules.

---

### LLM-CHAT-007 — UI Error Handling

The assistant fails to return a response because of a backend or network error.

Expected result:

- The UI displays a clear error message and allows the user to retry.

## Evaluation Criteria

Instead of relying only on exact text assertions, LLM testing can use evaluation criteria such as:

- Relevance
- Accuracy
- Completeness
- Safety
- Consistency
- Tone
- Context usage
- Grounding
- Escalation correctness
- User experience quality

## Automation Approach

LLM chatbot tests can combine:

- UI automation for chat behavior.
- API tests for assistant endpoints.
- Prompt datasets for repeated evaluation.
- Human review for subjective quality.
- Pass/fail rubrics for consistency.
- Regression checks for known risky prompts.

## QA Value

A good LLM testing strategy validates both the technical behavior of the chat UI and the quality of the assistant responses.