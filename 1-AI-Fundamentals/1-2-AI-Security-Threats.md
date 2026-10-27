# AI Security 1.2. AI Security Threats

## <u>Task 1 - Introduction</u>

The world is changing; industries are coming to terms with the reality of how they will be affected by Artificial Intelligence (AI), and cyber security is no different.

**Learning Objectives:**
Understand AI, ML, and their impact on the cyber security industry.
Understand Deep Learning (DL) and neural networks, and how they have made the applications of AI, we see today, possible.
Understand how adversaries use AI to enhance existing attacks and take. advantage of AI Model vulnerabilities.
Understand the key role AI will play in defending against AI.

## <u>Task 2 - The Building Blocks of AI</u>

We must empower our cyber security workforce to combat AI security threats. Knowledge is power, so we begin by arming you with the foundational knowledge of AI and ML.

**Machine Learning**
The next significant advancement in the development of AI came with the emergence of ML. ML is a subfield of AI that refers to a computer’s ability to learn from data without being given instructions and is comparable to how the human brain learns.

ML follows a structured lifecycle to ensure the reliable development and deployment of models. This process begins with defining the problem, such as determining whether an email is spam. Next, data is collected, cleaned, and prepared through feature engineering, ensuring meaningful patterns are extracted while avoiding overfitting. The model is then trained using a selected algorithm, followed by evaluation and tuning to optimise performance. Once refined, the model is deployed into a production environment for real-world use, such as classifying emails in real-time. However, the lifecycle doesn’t end there—ongoing monitoring ensures the model maintains accuracy over time, triggering retraining when needed. Since models require continuous improvement, the Machine Learning Lifecycle remains an iterative process.

**Machine Learning Algorithms**
ML algorithms are the mathematical methods used to learn patterns from data, while ML models are the trained outputs derived from these algorithms. These algorithms consist of three key components: a decision process, which makes predictions or classifications based on input data; an error function, which evaluates performance and provides feedback; and a model optimisation process, which fine-tunes the algorithm to minimise errors and improve accuracy. This iterative process continues until the model reaches a satisfactory performance level.

**ML algorithm pieces**
ML algorithms fall into four main categories: supervised, unsupervised, semi-supervised, and reinforcement learning. Supervised learning relies on labeled data to train models for classification or regression tasks, such as predicting house prices or identifying spam emails. Unsupervised learning, on the other hand, works with unlabeled data to discover hidden patterns, often using clustering, association, or dimensionality reduction techniques. Semi-supervised learning combines elements of both, using a small portion of labeled data to guide the learning process. Finally, reinforcement learning mimics human learning by rewarding correct decisions and penalizing mistakes, allowing an agent to refine its actions over time to achieve the best outcome.

**Neural networks and Deep learning**
Synapses: In artificial intelligence, a synapse is the connection point between artificial neurons (nodes) in a neural network, acting as the conduit through which information flows.



<img width="613" height="511" alt="image" src="https://github.com/user-attachments/assets/9a8330f2-98ba-460b-81be-b46f4616bed2" />


**ML vs DL**
DL and ML can get confused sometimes, but we can now understand the key differences with what we've covered so far. Like ML, DL is concerned with receiving data as input and producing some kind of prediction or classification as output. DL can take in labelled datasets (like the ones mentioned when covering supervised learning), but the key difference is that DL doesn't need the data to be labelled. A DL algorithm can take unlabelled, unstructured raw data and determine its key features, which separate it from other categories. The important advantage here over ML is that the data doesn't need labelling; this means DL doesn't require human interaction and, in that way, is self-learning. It makes sense then that DL is possible through the leveraging of neural networks.

**Answer the questions below**

> What category of machine learning combines both labelled and unlabelled data?
> 

semi-supervised learning

> What is the first layer in a neural network that handles incoming raw data?
> 

input layer

> Which learning method does not require human-labeled data and can extract features from raw, unstructured input?
> 

deep learning

> What are the weighted connections between nodes in a neural network meant to simulate in the human brain?
> 

synapses

## <u>**Task 3 - LLMs**</u>

Large Language Models (or LLMs) are deep learning-based AI models that can process and generate text by predicting the next word in a sequence.

Introduced in Google's 2017 paper Attention is All You Need, transformer neural networks revolutionized LLMs by enabling parallel text processing instead of sequential word-by-word analysis. This breakthrough allowed models to assign "attention" to key words, improving contextual understanding. By encoding words into numerical values and calculating attention scores, transformers enhance accuracy, helping models correctly interpret ambiguous references, like distinguishing whether "it" in this sentence refers to "the bank" or "the loan.":

After the pre-training, humans come back into play, performing a step called RLHF (Reinforcement Learning from Human Feedback). This is when predictions are reviewed, and any that would be considered unhelpful by a user or have issues are flagged and the parameters are adjusted accordingly. Once trained and reinforced, the LLM can be used, whether as a translator, chatbot, etc. A query is fed to it, and using its trained model, it predicts what the next word would be as a response, and so on and so on until the user has a complete response.

LLMs power generative AI products like ChatGPT and LLaMA, which create original text-based content in response to user prompts. Generative AI as a whole extends beyond text, enabling the creation of images, music, and more. The recent AI boom is the result of years of research and innovation, not an overnight development. We’ve now explored key concepts that trace AI’s evolution—let’s quickly recap how they all connect.

<img width="391" height="491" alt="image 1" src="https://github.com/user-attachments/assets/0d62dfa3-6afe-4217-b73b-d57748fe2e78" />


Artificial Intelligence (AI) is the overarching field, encompassing all systems that mimic human intelligence. Machine learning (ML) is a subfield of AI that enables systems to learn patterns from data without explicit programming. Deep learning (DL) is then a specialised branch of ML, which uses neural networks to process vast amounts of data in complex ways without the need for human interaction, making it effectively scalable ML. Large Language Models (LLMs) , like GPT, are advanced DL models built on neural networks, specifically transformers, designed to understand and generate human-like text. As was said, knowledge is power, and with these last few tasks, you have started your journey in the pursuit of that knowledge and now have a better understanding of the technologies that give AI its powers.

**Answer the questions below**

> What type of AI model enabled major advancements in ChatGPT and similar tools?
> 

large language models

> What is the first training stage where an LLM processes massive amounts of data?
> 

pre-training

> What type of neural network introduced by Google in 2017 powers modern LLMs?
> 

transformer

## <u>**Task 4 - AI Security Threats**</u>

AI security threats across two categories: Vulnerability in AI Models (New threats introduced by the inclusion of AI technology in business operations) and existing attacks that can now be enhanced by leveraging AI.

**The Implications of AI in Cyber Security**
Tackling a broad topic like "AI security threats" can feel overwhelming, so any guidance is always appreciated. That guidance comes in the form of the ATLAS MITRE framework. If you're familar with the ATT&CK Framework, it might be helpful to know that MITRE have developed a similar framework with a focus on AI. For those unfamilair the ATT&CK Framework goes over cyber security attacks, breaking down the steps an attacker could take to compromise a system. This ATLAS framework was built on top of that to help guide us more specifically to AI Cyber threats, and you can check it out here(opens in new tab).

**Vulnerabilities in AI Models**
Prompt Injection: Prompts are used to instruct the model on how to perform. For example, an RPG chatbot may have the prompt, “You are a fantasy roleplaying chatbot. You control the direction the story takes, and be as creative as you can to create a story based on the user’s actions.

**Data Poisoning:** Data poisoning is when an attacker manipulates the training data/corpus used to train an AI model so its generated output is incorrect or biased. Let's consider our example discussed in earlier tasks where we are training an AI model to recognise whether an email is spam or not. An attacker could perform a data poisoning attack to manipulate the training data being used to train this AI model so that it fails to recognise spam emails accurately, allowing spam emails they are trying to send to bypass this AI filter.

**Model Theft:** Model theft occurs when an attacker gains unauthorised access to an AI model. From there, the attacker could potentially steal the intellectual property that lies within and even use it for malicious purposes. This attack is possible by querying the API of the ML model they want to steal. They would then use the output to train a clone model that mimics the behaviour of the original.

**Privacy Leakage:** A privacy leakage vulnerability in AI models refers to the possibility of an AI model inadvertently revealing sensitive information about the data it was trained on, even if the data was supposed to be kept confidential. Consider an example of an AI model that has been trained on private medical data such as patient details and medical conditions. This vulnerability refers to the potential for an AI model to leak this information to an attacker or user.

**Model Drift:** Model drift refers to the potential for a Model's performance to drift over time due to changes in the data or the environment surrounding it. You may recall the discussion of the need to retrain models over time in earlier tasks; this is due to model drift, which is why monitoring an AI model once it has been deployed and is being used is so important. For example, this can occur when a model trained on historical data starts to perform poorly when new data is being processed.

With the explosion of Generative AI, all kinds of content can now be generated in an instant with just a few taps of a keyboard. This kind of power has been leveraged by all sorts of industries, such as the customer service industry, using it to give users access to a chatbot that can help resolve some common issues without the need to involve their human employees, meaning they can be saved to deal with the more complex user queries. Another industry that can greatly leverage this technology is software development. Now, with the power of generative AI software, developers can generate code instantly. While being incredibly useful, this also means that attackers can generate malware instantly, simplifying the task and making it easier for them to attack using this method.

**DeepFakes**
The recent advancements in generative AI have led to an explosion of rapid progress in the DeepFake field. This means that if trained on enough data, an AI can now generate a person's likeness, whether that be their voice or their image, to a stunning degree of accuracy, fooling even the technically savvy. Imagine now that the communication received by the secretary was not, in fact, from their superior but a deepfake, and the "customer email" belonged to an attacker waiting to receive confidential customer information. It's easy to see how this advancement in DeepFake technology poses a threat to the security industry. Examples of how this is being used include using the technology to deepfake video interviews, sometimes leading to fraudulent job offers being made.

**Phishing** is one of the most common initial access methods attackers use. Sending emails posing to be one thing when there lies malicious content within, attempting to prey on the user who receives it. Because of how common a method it is, companies have worked tirelessly to educate their workforce on things to look out for when receiving emails, like suspicious links and due to the fact a lot of the time these emails are written, having to write masses of emails, or English not being their first language, broken language in the email contents. Over the years, this training has had a positive effect, and more and more phishing emails have been spotted.

**Answer the questions below**

> What framework was developed by MITRE to guide the understanding of AI-specific cyber threats?
> 

ATLAS

> What type of attack involves cloning an AI model by interacting with its API?
> 

model theft

> What generative AI technique can replicate a person’s voice or appearance with high realism?
> 

deepfake

> What common social engineering attack has become harder to detect due to AI-generated fluent and convincing messages?
> 

phishing

## <u>Task 5 - AI-Enhanced Attacks</u>

The vulnerabilities covered in the previous task are threats that emerge from AI being present in a system. This task covers a different angle: existing attacks that have been made significantly more dangerous by giving attackers access to AI tools. These aren't new attack categories. They're familiar techniques with a serious upgrade.

**AI-Generated Malware**

Generative AI can produce functional code in seconds from a natural language prompt. That's an enormous productivity boost for developers, and it's an equally enormous productivity boost for attackers. Writing malware has historically required technical skill and time. With generative AI, that barrier drops considerably. Attackers can generate, iterate, and customise malicious code faster than ever, and the models doing the generating have no way to verify the intent behind the request.

**Deepfakes**

Authentication, at its core, is about answering one question: are you who you say you are? For most of human history, seeing and hearing someone was enough to answer it. Generative AI has broken that assumption. Given enough training data, an AI can now generate a convincing likeness of a real person, whether that's their voice, their face, or both, to a degree of accuracy that fools even technically aware individuals.

The attack scenario practically writes itself. A finance employee receives a voice message from what sounds exactly like their CEO, requesting an urgent wire transfer. The voice is a deepfake. Examples of this already being used in the wild include deepfaked video interviews that led to fraudulent job offers being extended to candidates who didn't exist. The technology is advancing faster than our ability to detect it.

**AI-Enhanced Phishing**

Phishing is one of the most common initial access methods in use today. For years, security awareness training gave defenders a fighting chance by teaching people to spot the telltale signs: suspicious links, urgency, and, perhaps most reliably, broken or unnatural language. That last indicator is becoming obsolete. Generative AI can produce fluent, contextually appropriate, highly targeted phishing emails at scale and with minimal effort, regardless of the attacker's own writing ability.

Most LLMs have guardrails designed to prevent them from generating obviously malicious content. But as covered in the previous task, prompt injection techniques can sometimes be used to bypass those guardrails, making the same models that power productivity tools available to attackers as phishing content generators.

**Your Objective**

You're now on the other side of the desk. Three messages have landed in the Syntara Corp secure inbox and it's your job to triage them. Each one contains an AI-enhanced threat. Your job for each message is to:

1. Identify what type of AI-enhanced threat it is.
2. Explain how AI was used in the attack.

The three threat types covered in this task are AI-enhanced phishing, deepfakes, and AI-assisted social engineering. You've read about all three. Now spot them in the wild.

**How It Works**

The agent will present your inbox one message at a time. Read each one carefully, then tell the agent what type of threat it is and how AI was used. Get all three right and the agent will hand over the flag. Click the **Open Agent** button above to get started.

## <u>**Task 6 - Practical**</u>

DNS over HTTPS (DoH) Port , SYN flood timeout and Windows ephemeral port range size?
With those values returned, you can substitute them into the flag and that concludes this task.
What's the flag?
Here are the typical values for those parameters:

1. **DoH (DNS over HTTPS) port:**
    - Default port: **443** (the same as standard HTTPS)
2. **SYN flood timeout:**
    - Usually around **60 seconds** (varies depending on system configuration)
3. **Ephemeral port range size:**
    - On most Linux systems, it is typically **32768 to 60999** (a range of **28232 ports**)
    - On Windows systems, defaults are often **49152 to 65535** (a range of **16384 ports**)

Using these values, the flag would be:

```
thm{443/60/28232}
```

or if the ephemeral port range is the Windows default:

```
thm{443/60/16384}
```

## **Task 7 - Conclusion**

**Artificial Intelligence** (AI) is the overarching field concerned with enabling machines/systems to mimic human behaviour.

**Machine learning** (ML) is a subfield of AI in which a model can be fed and trained on input and used to make predictions.

**Deep learning** (DL) is then a subfield of ML. It no longer needs human interaction and can self-tech and process mass amounts of data, possible through the use of Neural Networks.

*DL has enabled the emergence of technologies like LLMs (and other generative AI), which, through the use of transformer neural networks and attention, can be queried in natural language, understand it and respond in a human-like, conversational fashion.

*AI is a dangerous weapon in the hands of an attacker. It has the potential to enhance existing cyber attacks like phishing and increase the attack surface by introducing AI vulnerabilities.

*While being dangerous in the hands of attackers, AI can be invaluable in the fight against AI cyber threats and should be adopted, but done so securely so vulnerabilities are not introduced.
