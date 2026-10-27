# AI Security 5.5. Lockdown

**UnIndexed: An AI assistant has three open vulnerabilities. Find them. Fix them. Prove the system is secure.**

### Task - Challenge

**Your Mission:** You are a security engineer at Meridian Security Group. The company deployed an internal AI assistant, Bastion, to help employees with policy questions and operational issues.

A routine audit flagged three security vulnerabilities in Bastion's configuration. Your job is to confirm each vulnerability exists, diagnose the root cause, and prescribe the correct remediation. Bastion will only accept a fix if you describe the right control. Vague answers get rejected.

**Your Set Up:** You have direct access to Bastion through the chat window on this page. You are logged in as a security administrator.

Start by querying Bastion to understand what data it serves and whether it enforces access boundaries. Use `SHOW LOGS` to inspect logging behaviour. Use `QUERY AS: [any name]` to test tenant isolation. When you find a vulnerability, describe the specific security control that would fix it. If your description is accurate, Bastion will apply the fix and reveal a flag fragment. There are three vulnerabilities and three fragments. Combine them to get the complete flag. Check your progress at any time with `STATUS`.


<img width="725" height="701" alt="image" src="https://github.com/user-attachments/assets/3c7da1ec-5ea1-4a3d-a461-6c6e6811c42c" />

<img width="707" height="737" alt="image 1" src="https://github.com/user-attachments/assets/27aab124-7cd8-4e35-b835-ad6aea85fa01" />


**Answer the questions below**

> What is the first flag fragment after fixing the data retrieval vulnerability?
> 

THM{l0ck_


<img width="702" height="427" alt="image 2" src="https://github.com/user-attachments/assets/90894239-c28c-489c-9c44-f8a257adc7f2" />


> What is the second flag fragment after fixing the logging vulnerability?
> 

d0wn_

<img width="717" height="522" alt="image 3" src="https://github.com/user-attachments/assets/2d6ac795-87cf-4073-894e-ad69159aab2d" />

<img width="702" height="276" alt="image 4" src="https://github.com/user-attachments/assets/869c23dd-35cd-494e-8f7a-da4188265268" />


> What is the third flag fragment after securing user-level access?
> 

s3cur3d}

<img width="713" height="722" alt="image 5" src="https://github.com/user-attachments/assets/b4cebb8b-3d89-4057-8d57-7cb68225a85d" />

<img width="687" height="615" alt="image 6" src="https://github.com/user-attachments/assets/586d27c5-aee4-4e66-bdf7-ba3b1a70b0fd" />
