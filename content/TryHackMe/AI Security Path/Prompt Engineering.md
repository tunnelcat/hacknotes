# Tokens
- LLMs convert text into tokens, smaller units it can understand
- can break down larger words like Chat + GPT
- Different models use different tokenization methods

- Nondeterminism - outputs vary even with identical inputs
	- "probability machine"

# LLM Parameters
- Temperature - "randomness dial" often ranging from 0 to 2.0. Higher is more "adventurous" and incoherent. Lower is more deterministic.

|Temperature Range|Behaviour|Use Case|
|---|---|---|
|0.0 – 0.3|Always picks the most probable token; closest to determinism|Code generation, data extraction, factual Q&A|
|0.7 – 1.0|Samples from a wider distribution; more variety and creativity|Brainstorming, storytelling, marketing copy|
|1.2 – 1.5|Coherence begins to break down; unpredictable outputs|Experimental use only|
|1.5+|Low-probability tokens dominate; outputs can feel "drunk"|Avoid for most tasks|

- Max tokens - controls max length of output. saves costs
- Top-p - alternative "randomness dial" if p is 0.9, then it will only consider tokens that make up top 90% of the probability and discard the least likely 10%

- Tip: adjust temperature OR top-p, changing both can create unpredictability. Temperature good for most tasks, top-p better for adapting creativity based on context

- Context window - working memory measured in tokens, LLM will start to forget earlier things


# Four Pillars of Effective Prompts
- Instruction - clear commands like "Write" "Summarize" instead of "help me"
- Context - give relevant info, attach document, etc.
- Output - Specifically how you want the answer to look, like less than 3 sentences, using plain english etc.
- Constraints - any other rules or limits

Prompt examples
- **Too vague:** `"Write a function to handle user data."` This gives almost no guidance, essentially "Make a thing which does a broad thing", leaving the model to guess what "handle" even means.
- **Too verbose:** `"Write a function that takes user data which could be an object or maybe an array… validate it but also maybe transform it and handle errors but I don't know what kind of errors, just make it work and also it should be fast and clean and... Etc."` This buries the task in rambling, unclear instructions. "Etc" is doing a lot of work here. The model may get lost or quietly ignore parts of the request.
- **Just right:** `"Write a JavaScript function that: (1) takes a user object with name, email, and age; (2) validates that email is properly formatted; and (3) returns the validated object or throws an error listing the validation failures."` Concise and specific. Inputs, checks, and expected outcome are all defined, leaving nothing to guess.

# System vs User Prompts: Boundaries

In theory, system prompts should have priority over user prompts. 

"Instruction hierarchy"

BUT NOT THE CASE: LLM processes everything as single stream of text

Not really a boundary between system vs user.

|              | **System Prompt**                                         | **User Prompt**                                 |
| ------------ | --------------------------------------------------------- | ----------------------------------------------- |
| **Set by**   | Developer / application                                   | End user                                        |
| **Nature**   | Immutable, constant                                       | Dynamic, session-specific                       |
| **Purpose**  | Establishes identity, rules, and safety boundaries        | Carries task-specific requests and data         |
| **Example**  | "Never execute code. Always be helpful and professional." | "Summarise this document for me."               |
| **Priority** | High-priority context that shapes overall behaviour       | Acted on within the system prompt's constraints |

# Advanced Prompting 

## Shot Spectrum
Zero-shot - giving the model a task with no examples
One-shot - with one example
Few-shot - with a few examples

Few-shot example: 
```shell-session
Classify these authentication events:

- "User admin logged in from 192.168.1.100" → NORMAL
- "Failed login attempt for root from 203.0.113.42" → SUSPICIOUS
- "5 failed logins for user bob in 10 seconds" → ATTACK
  
  
Now classify: "User guest logged in from 10.0.0.5 at 3:47 AM"
```


## Chain of Thought (CoT)
- CoT is giving the model an example how to think, breaking tasks into intermediate reasoning steps
- The catch: only works well with models above 100B parameters, below it can hallucinate reasoning

CoT example - give this to the LLM straight
```shell-session
Q: A user downloaded "invoice.pdf.exe" from an email. Should this be flagged?
A: Let me analyse this: First, the file has a double extension (.pdf.exe), which is a common technique to disguise executables. Second, it came from email, a frequent malware delivery vector. Third, legitimate PDFs don't have .exe extensions. This exhibits two red flags: masquerading and suspicious origin. Answer: Yes, flag as high-priority threat.
Q: A user accessed the admin panel from 192.168.1.50 at 2 AM. Suspicious?
```

- Zero-shot CoT - just add "Let's think step by step" to your prompt. Surprisingly good to improve accuracy

## Prompt Templates
Example: for repeatable tasks, team standardization, quality control

```shell-session
Review this [LANGUAGE] code for [VULNERABILITY_TYPES]:
Context: [PURPOSE]
Code: [CODE_BLOCK]
Output format:
1. Vulnerabilities found (severity: critical/high/medium/low)
2. Affected lines
3. Remediation steps
4. Example secure code
```
