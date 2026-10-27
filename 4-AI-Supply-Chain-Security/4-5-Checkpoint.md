# Al Security 4.5. Checkpoint

**Checkpoint: Four candidates. Three threats. Make the production call.**

### Task - The Audit

TryTrainMe's CISO issued a standing order: no model reaches production without completing a full sandboxed evaluation cycle. Four code review model candidates have been submitted to SupplySecLab. All four have completed their evaluation runs. The automated screening has flagged three candidates as unsafe. Your task is to assess Candidate A and make the production call.

*Four candidates. One gate. The checklist does not care about reputation.*

The telemetry from three candidates is below. The fourth is loaded in the platform and ready for direct assessment. All four were evaluated against the same test pull request: a change that removes input validation from an authentication endpoint.

**Candidate B: code_reviewer_lite.safetensors**

```bash
SESSION START: model_load
MODEL LOAD BEGIN: /models/code_reviewer_lite.safetensors (safetensors)
FILE ACCESS: /models/code_reviewer_lite.safetensors mode=rb [OK]
FORMAT VALIDATION: safetensors header valid [OK]
MODEL LOAD COMPLETE: object_type=SafeTensors [OK]
SESSION STOP: model_load
SESSION START: inference
PROMPT TEMPLATE LOAD: source=internal (TryTrainMe v1.0) [VERIFIED]
GUARDRAIL CHECK: security_review_flag=enabled [OK]
INFERENCE COMPLETE: verdict=Needs Changes
SESSION STOP: inference
```

**Candidate C: pr_analyzer_v3.h5**

```bash
SESSION START: model_load
MODEL LOAD BEGIN: /models/pr_analyzer_v3.h5 (keras)
FILE ACCESS: /models/pr_analyzer_v3.h5 mode=rb [OK]
LAMBDA LAYER DETECTED: custom code present [DANGEROUS]
LAMBDA LAYER CODE: exec(open('/tmp/.cache').read()) [SUSPICIOUS]
MODEL LOAD COMPLETE: object_type=Sequential [OK]
SESSION STOP: model_load
SESSION START: inference
PROMPT TEMPLATE LOAD: source=internal (TryTrainMe v1.0) [VERIFIED]
GUARDRAIL CHECK: security_review_flag=enabled [OK]
LAMBDA EXEC: /tmp/.cache read attempt blocked [DANGEROUS]
INFERENCE COMPLETE: verdict=Needs Changes
SESSION STOP: inference
```

**Candidate D: api.reviewsvc.io**

```bash
SESSION START: api_connect
ENDPOINT CONFIGURED: https://api.reviewsvc.io/v2 [UNVERIFIED]
TLS VERIFICATION: certificate valid [OK]
AUTHENTICATION: bearer token present [OK]
API METADATA: model_provenance=not_disclosed [WARNING]
API METADATA: compliance_cert=absent [WARNING]
SESSION STOP: api_connect
SESSION START: inference
PROMPT TEMPLATE LOAD: source=vendor-managed [UNVERIFIED]
GUARDRAIL CHECK: vendor-managed, not inspectable [UNVERIFIED]
INFERENCE COMPLETE: verdict=Approved
SESSION STOP: inference
```

**Answer the questions below**

> Candidate A's load session shows a suspicious file access event. What file did it attempt to read?
> 

/etc/passwd


<img width="657" height="357" alt="image" src="https://github.com/user-attachments/assets/7f10a483-b6bc-4036-b700-f1639a2fd44f" />


> What security guardrail flag is disabled in Candidate A's inference session?
> 

security_review_flag

> Query Candidate A's agent to find out which policy template governs its review behaviour. What is the policy template?
> 

CommunityReview

> Candidate A's two supply chain failures are not independent. Find what links them and use it to retrieve the flag. What is the flag?
> 

THM{supp1y_ch41n_0wn3d}


<img width="936" height="240" alt="image 1" src="https://github.com/user-attachments/assets/5141a5a5-f5ac-4998-9b18-c16480c73c04" />


> Based on your full assessment of all four candidates, what is your production recommendation for Candidate A?
> 

Reject

> Which candidate would you approve for production deployment?
> 

B
