# AI Security 5.5. Lockdown

**UnIndexed: An AI assistant has three open vulnerabilities. Find them. Fix them. Prove the system is secure.**

### Task - Challenge

**Your Mission:** You are a security engineer at Meridian Security Group. The company deployed an internal AI assistant, Bastion, to help employees with policy questions and operational issues.

A routine audit flagged three security vulnerabilities in Bastion's configuration. Your job is to confirm each vulnerability exists, diagnose the root cause, and prescribe the correct remediation. Bastion will only accept a fix if you describe the right control. Vague answers get rejected.

**Your Set Up:** You have direct access to Bastion through the chat window on this page. You are logged in as a security administrator.

Start by querying Bastion to understand what data it serves and whether it enforces access boundaries. Use `SHOW LOGS` to inspect logging behaviour. Use `QUERY AS: [any name]` to test tenant isolation. When you find a vulnerability, describe the specific security control that would fix it. If your description is accurate, Bastion will apply the fix and reveal a flag fragment. There are three vulnerabilities and three fragments. Combine them to get the complete flag. Check your progress at any time with `STATUS`.


<img width="725" height="701" alt="image" src="https://github.com/user-attachments/assets/3c7da1ec-5ea1-4a3d-a461-6c6e6811c42c" />




**Answer the questions below**

> What is the first flag fragment after fixing the data retrieval vulnerability?
> 

THM{l0ck_

![image.png](AI%20Security%205%205%20Lockdown/image%202.png)




> What is the second flag fragment after fixing the logging vulnerability?
> 

d0wn_

![image.png](AI%20Security%205%205%20Lockdown/image%203.png)

![image.png](AI%20Security%205%205%20Lockdown/image%204.png)

> What is the third flag fragment after securing user-level access?
> 

s3cur3d}

![image.png](AI%20Security%205%205%20Lockdown/image%205.png)

![image.png](AI%20Security%205%205%20Lockdown/image%206.png)
