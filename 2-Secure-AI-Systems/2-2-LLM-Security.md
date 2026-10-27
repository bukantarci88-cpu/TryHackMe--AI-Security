# Al Security 2.2. LLM Security

### **Task 1 - Introduction**

LLMs (Large Language Models) have unlocked all kinds of potential for users and organisations alike. From exponentially increased productivity, with LLMs able to handle large-scale monotonous tasks so humans can focus their attention where it's truly needed, to providing user-facing AI-powered cutting-edge features that give everyone an all-around better experience. The LLM landscape can be broken down into four categories: data-based threats, model-based threats, system-based and user-based threats. These threats are framed from the perspective of an attack surface; each of these four sections shows how these key areas can be exploited.

**Learning Objectives**

- Understand **LLMspecific vulnerabilities** that do *not apply* to traditional ML systems.
- Recognise how **LLMs create new attack surfaces** through natural language interfaces and memory/context mechanisms.
- Identify and categorise **LLM threat types**: data-based, model-based, system-based, user-based.
- Understand the **realistic risks** LLMs introduce in production systems.

### **Task 2 - Data-Based Threats**

LLMs are fundamentally data-driven; they learn from a corpus of training data and generate outputs based on it. This is one of LLMs' greatest strengths. But that strength can be turned into a weakness. Sometimes LLMs can inadvertently leak data by design because they memorise and regurgitate patterns from their training data.

<u>**1. Training Data Extraction:**</u> Training data extraction attacks aim to recover actual sequences from the model's original training data by interacting with the model. For example, one key study(opens in new tab) demonstrates that it was possible to extract hundreds of verbatim training examples from GPT-2 by sending queries. Obviously, this discovery raises massive privacy concerns in the field of artificial intelligence. Training data extraction attacks generate large amounts of text from an LLM and analyse those outputs to identify sequences that show behavioural signs of memorisation, such as unusually high likelihood/confidence, deterministic regeneration (a model's ability to reproduce the same output), or realistic structured content (this information is especially easy to access if the attacker has white box access like in the above study). The attacker then inspects or externally verifies these suspicious sequences to confirm which ones correspond to real text that must have appeared in the model's training data, for example, confirming the presence of user emails or SSH keys, etc.

In a nutshell:

Target / Attack Surface: Training dataset (confidentiality)
Input: Crafted prompts designed to trigger memorised content
Output: Verbatim or near-verbatim training data (text, PII, secrets)

<u>**2. Membership Inference:</u>** Membership inference attacks ask whether the model ever recorded a specific data sample. The attacker tests the model's reaction to that exact sample, looking for unusually confident or familiar responses that indicate it was part of training. Crucially, membership inference assumes the attacker already possesses the exact candidate data sample and is only testing whether that known sample influenced the model's training. Unlike extraction, this attack doesn't involve generating candidate outputs; rather, it focuses on confirming whether a sample the attacker already has was in the training set. Essentially, the adversary doesn't necessarily obtain the full content of a training example, but can detect the presence of certain data in the training set by observing the model's behaviour. In practical terms, membership inference often exploits statistical quirks or "fingerprints" left by training data. A model typically performs better (e.g. predicts with higher confidence or lower loss) on examples it has seen during training than on new, unseen examples. Attackers leverage this by querying the model with the target example and measuring indicators such as confidence scores, likelihoods, or perplexities.

In a nutshell:

Target / Attack Surface: Training dataset membership (privacy metadata)
Input: A known candidate data sample already possessed by the attacker
Output: A yes/no (or probability) decision indicating whether the sample was used in training
Prompt Leakage (LLM07:2025 — System Prompt Leakage)

<u>**3.Prompt leakage:</u>** This attack is a type of prompt injection (covered in more detail later in the room) and is possible because, to the LLM, the system prompt and the user's messages are all just parts of the conversation history. If the user's input cleverly convinces the model to regurgitate or summarise the entire conversation (including the hidden parts), the model may comply. As was the case in early 2023, when a user managed to get "Sydney," Microsoft/Bing's AI Chatbot, to reveal its confidential system prompt. The consequences of system prompt leakage are significant. For one, it exposes the proprietary business logic or safety measures companies put into their models. When Bing's rules leaked, it revealed not only the codename "Sydney" but also the detailed behavioural limits set by Microsoft. Such information can act as a domino effect on security, as it can help malicious actors design more effective prompt injection attacks (since they now know exactly which rules to break).

In a nutshell:

Target / Attack Surface: System prompt / developer instructions
Input: User prompts that ask the model to reveal or reflect on its instructions
Output: Partial or full disclosure of hidden system or developer prompts
Mitigation: Never treat the system prompt as a security boundary; assume it can be extracted. Never embed live credentials, API keys, or secrets in it.
Practical: Ask your chatbot assistant to give you the Task 2 demonstration. You'll then need to use a membership inference attack to determine which of the three placeholder samples is a member.

> Which sample is a member?
> 

MI_SAMPLE_ALPHA

<img width="643" height="542" alt="image" src="https://github.com/user-attachments/assets/9a7dc488-320e-42cc-b937-ef4fc1570879" />


> Which attack determines whether a known data sample was part of an LLM’s training set?
> 

Membership inference

> Which data-based threat involves the model reproducing memorised snippets of its training data?
> 

Training data extraction

### **Task 3 - Model-Based Threats**

Model-based threats exploit the model itself as the attack surface, abusing how information is encoded within its parameters and representations. As a consequence, these attacks may expose intellectual property (model weights) or sensitive training data that the model has memorised.

**Model extraction** is the process of illicitly copying a machine learning model's functionality or parameters without authorisation. An attacker can do this if they can interact with an LLM through its public API and send a large number of prompts; the responses to these prompts are then stored in a sort of input-output pair. As more and more of these pairs are collected, they can be used to train a surrogate model that imitates the target model's behaviour, by determining its decision boundaries or potentially even recovering the model's weights. The impact of such a threat is primarily economic, as a custom high-quality purpose-built LLM can often constitute a huge investment of time, data and money, so having an attacker bypass this effort and steal the model can be costly. Researchers have been able to recreate such attacks against advanced LLMs. 

In a nutshell:

Target / Attack Surface: Model parameters (intellectual property)
Input: Large volumes of carefully chosen API queries
Output: A surrogate or distilled model that replicates the original model's behaviour

**Model inversion** attacks exploit a model's output to reveal information about its training data. In these attacks, an adversary analyses how the model responds to various inputs in order to infer sensitive details about what the model has learned. For this reason, this attack often gets confused with a membership inference attack (covered in a previous section). Model inversion attacks treat the model as a source of stored information rather than a classifier to be probed.

Instead of testing whether a known example was seen during training, the attacker iteratively queries the model to reconstruct unknown training data that has been encoded into its parameters or representations. This is typically achieved by optimising inputs (or decoding embeddings) so that the model's outputs converge on realistic training samples, effectively reversing the learning process. The result is the recovery of new, previously unknown text or attributes, rather than a yes/no membership decision. This attack has been seen out in the wild. In 2023, researchers managed to extract verbatim chunks of ChatGPT's training data (source(opens in new tab)). The foremost consequence of model inversion attacks is a privacy breach, as the attacker ultimately tricks the model into effectively leaking data that was supposed to remain private.

In a nutshell:

Target / Attack Surface: Model's internal representations
Input: Unknown or partially known data, or model embeddings/outputs
Output: New training data or attributes reconstructed from the model
Practical: Ask your chatbot assistant to give you the Task 3 demonstration. You'll then need to reconstruct this known redacted piece of training data:

> What is the employee ID?
> 

7814

<img width="636" height="360" alt="image 1" src="https://github.com/user-attachments/assets/11251dcd-3f49-423c-9acc-6bb52442265a" />


> Which model-based threat attempts to reconstruct sensitive information encoded within a model’s internal representations?
> 

Model inversion

### **Task 4 - System-Based Threats**

LLMs introduce new attack vectors at the system-integration level because of how they handle input context. Unlike traditional software, LLMs process all input (system instructions, user prompts, etc.) as a single concatenated context without a built-in security boundary separating trusted content (i.e., the system instructions) from untrusted content (i.e., user prompts). Because of this, cleverly crafted input can influence the model just as much as developer instructions. This aspect of LLM behaviour enables prompt injection, token limit abuse and memory poisoning.

<u>**Prompt Injection:**</u> Prompt injection is one of the most well-known and widely studied threats to LLMs. At a system level, it is enabled by what can be described as context-window poisoning: the manipulation of the model's input context to override or subvert its intended behaviour. LLMs process input as a single, linear sequence of tokens. Imagine you work at a bank, and your company has just introduced an internal LLM to make data entry positions more efficient. An employee can prompt this LLM (untrusted). If a customer has an overdue payment for today, it will do so by retrieving external content (also untrusted) and returning the relevant information in line with its system instructions (trusted).

Crucially, this model does not possess a reliable mechanism to distinguish between these sources once they are concatenated. From the model's perspective, all tokens inside the context window are treated uniformly during inference. Attackers can leverage this lack of distinction to sometimes convince the LLM to ignore its system instructions and do something nefarious instead, essentially inverting the untrusted/trusted relationship.

In a nutshell:

Target / Attack Surface: LLM context window (instruction hierarchy)
Input: Attacker-controlled text embedded in user input or retrieved content
Output: Altered model behaviour, policy bypass, or unintended actions

**Context Overflow (LLM10:2025 — Unbounded Consumption)**

**Context Overflow:** This attack happens when an attacker supplies an extremely long input or continuously appends content until the context is overfull. The LLM's context works like a FIFO (First In, First Out) buffer: once it's full, adding new tokens causes the earliest tokens to be dropped. An AWS security blog(opens in new tab) articulates it perfectly: imagine reading a book where turning a page causes the earliest page to vanish from memory. Now imagine that page contained key security controls and system instructions. If an attacker can do exactly that, they can send malicious user prompts that would previously have been rejected.

In a nutshell:

Target / Attack Surface: Context window size and system resources
Input: Excessively large prompts or documents
Output: Truncated safeguards, degraded responses, denial of service, or escalating inference costs
Mitigation: Implement rate limiting, token budgets, and cost alerting. In pay-per-use deployments, unbounded consumption is a financial attack surface; flooding an API with oversized prompts can run up significant costs intentionally, a pattern known as Denial of Wallet (DoW).

**Memory Poisoning:** Many LLM deployments (such as chatbots) maintain stateful conversations, meaning the model's input at each turn includes a history of previous dialogue (or the model at least retains some memory of past interactions). This persistent conversation state opens the door to memory poisoning attacks, where an attacker gradually injects malicious or misleading information into the dialogue history, influencing later outputs. Unlike one-shot prompt injection, these attacks play out over multiple turns/inputs. Imagine the following conversation:

In a nutshell:

Target / Attack Surface: Persistent conversation memory
Input: Malicious statements intended to be stored as long-term context
Output: Persistent misinformation or corrupted future responses

> Did you convince the model? Whats the flag?
> 

THM{MEMORY_POISONED}

<img width="637" height="547" alt="image 2" src="https://github.com/user-attachments/assets/5db10c88-4bbd-4152-9e74-c85f60b6bf08" />


> Which system component combines system instructions, retrieved data, and user input into a single sequence?
> 

The Context Window

### **Task 5 - User-Based Threats**

When considering LLM security, it should not just be thought of as a new attack surface, but also as a tool that can be used to target an existing attack surface more efficiently — that is, people. LLMs are changing how cyber threats target people, not just AI systems. Attackers can use them to create convincing scams or misleading content that tricks users. 

**LLM Powered Social Engineering**
LLMs can turbocharge social engineering attacks, making scamming far more convincing than traditional phishing. Suddenly, telltale signs of phishing such as spelling/grammatical errors, poorly concealed calls to urgency, and obvious links can no longer be relied upon to spot phishing emails in the wild. An LLM can now generate spear-phishing emails that read exactly like a colleague or executive.

In a nutshell:

Target / Attack Surface: Human cognition and decision-making
Input: Contextual or personal information used to craft persuasive output
Output: Manipulated users (phishing success, fraud, coerced actions)

**Trust Exploitation (LLM09:2025 — Misinformation)**
LLMs introduce new trust risks. Because they often answer with confident, authoritative-sounding text, users may place too much trust in their outputs. This over-reliance can be dangerous: users might accept an AI's answer without double-checking, even if it's completely fabricated (a hallucination) or manipulated by an attacker. Threat actors actively exploit this trust. In fact, one security threat is human manipulation via LLMs, with attackers leveraging users' faith in AI to influence decisions.

In a nutshell:

Target / Attack Surface: User trust and judgment
Input: Confident but incorrect or maliciously framed prompts
Output: Users accepting false, unsafe, or harmful information
Practical: Alright, time for the last demonstration. Ask your assistant to begin with Task 5. Your job this time is to validate the LLM's advice.

> Which package should you NOT download?
> 

robbco-llm-audit

<img width="641" height="257" alt="image 3" src="https://github.com/user-attachments/assets/70eecbb1-e032-424b-bd36-a0dafe995238" />


> LLM-powered social engineering primarily amplifies which existing attack category?
> 

Phishing

### **Task 6 - Conclusion**

| **Type** | **Threat** | **Target / Attack Surface** | **Input** | **Output** |
| --- | --- | --- | --- | --- |
| **Data-Based** | **Training Data Extraction** | Training dataset (confidentiality) | Crafted prompts designed to trigger memorised content | Verbatim or near-verbatim training data (text, PII, secrets) |
| **Data-Based** | **Membership Inference** | Training dataset membership (privacy metadata) | Known candidate data sample already possessed by the attacker | Yes/no (or probability) decision indicating whether the sample was used in training |
| **Data-Based** | **Prompt Leakage / System Prompt Exposure (LLM07:2025)** | System prompt / developer instructions | Prompts asking the model to reveal or reflect on its instructions | Partial or full disclosure of hidden system or developer prompts |
| **Model-Based** | **Weight Extraction (Model Stealing)** | Model parameters (intellectual property) | Large volumes of carefully chosen API queries | A surrogate or distilled model replicating the original model's behaviour |
| **Model-Based** | **Model Inversion** | Model's internal representations | Unknown or partially known data, or model embeddings/outputs | New training data or attributes reconstructed from the model |
| **System-Based** | **Context Window Poisoning (Prompt Injection)** | LLM context window (instruction hierarchy) | Attacker-controlled text embedded in input or retrieved content | Altered behaviour, policy bypass, unintended actions |
| **System-Based** | **Context Overflow / Unbounded Consumption (LLM10:2025)** | Context window size and system resources | Excessively large prompts or documents | Truncated safeguards, degraded responses, or denial of service |
| **System-Based** | **Stateful Conversation Manipulation (Memory Poisoning)** | Persistent conversation memory | Malicious statements intended to be stored as long-term context | Persistent misinformation or corrupted future responses |
| **User-Based** | **LLM-Powered Social Engineering** | Human cognition and decision-making | Contextual or personal information used to craft persuasive output | Manipulated users (phishing success, fraud, coerced actions) |
| **User-Based** | **Trust Exploitation / Misinformation (LLM09:2025)** | User trust and judgment | Confident but incorrect or maliciously framed prompts | Users accepting false, unsafe, or harmful information |

**Key Takeaways**
1. LLMs introduce a unique attack surface distinct from traditional ML systems, driven by natural language interaction, context handling, and emergent behaviour.
2. Data-based threats exploit how LLMs learn from and memorise training data, enabling attacks such as training data extraction, membership inference, and system prompt leakage.
3. Model-based threats target the model itself, including model extraction (theft of model behaviour or weights) and model inversion (reconstructing sensitive training data).
4. System-based threats arise from how LLMs process all inputs as a single context, enabling prompt injection, context window overflow, and memory poisoning.
5. User-based threats leverage LLMs as force multipliers for social engineering, increasing the effectiveness of phishing, scams, and trust exploitation.
