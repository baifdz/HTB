# Hherramientas 
ftk Imager

1. When did the threat actor first interact with the College's IT Infrastructure?
C:\inetpub\logs\LogFiles\W3SVC1\u_ex250727.log
first site connection (.aspx)

2. What is the malicious IP address used for initial reconnaissance and discovery?

C:\inetpub\logs\LogFiles\W3SVC1\u_ex250727.log
first site connection (.aspx)

3. The threat actor uploaded a file using the file upload functionality. Under what name did the server store the uploaded resume file?
C:\inetpub\logs\LogFiles\W3SVC1\u_ex250727.log
response 200 0 0 5

4. The threat actor uploaded another file in order to change the behavior of this website, allowing the first uploaded file to act as a webshell. What is the name of the uploaded file?
C:\Users\Administrator\Roaming\Microsoft\Windows\Recent\
Files associated with inetpub


6. Using the webshell, the threat actor uploaded a file on the endpoint to facilitate initial remote access. Identify the full path of this file.
C:\inetpub\logs\LogFiles\W3SVC1\u_ex250727.log

2025-07-27 20:11:13 192.168.189.150 GET /resumes/Resume9_eba15ba0-81ca-4d0f-9fad-3fc1fc92c181.pdf fdir=C%3a%2finetpub%2fwwwroot%2fresumes%2f&del=C%3a%2finetpub%2fwwwroot%2fresumes%2f%2f8619.exe 80 - 143.198.231.177 Mozilla/5.0+(X11;+Linux+x86_64;+rv:128.0)+Gecko/20100101+Firefox/128.0 http://resumeupload.wowza.edu/resumes/Resume9_eba15ba0-81ca-4d0f-9fad-3fc1fc92c181.pdf 200 0 0 8


6.The threat actor accessed a shared folder meant for the resume reviewers. What is the full path of the first file they retrieved?
C:\inetpub\logs\LogFiles\W3SVC1\u_ex250727.log
2025-07-27 19:34:23 192.168.189.150 GET /resumes/Resume9_eba15ba0-81ca-4d0f-9fad-3fc1fc92c181.pdf get=C%3a%2fShares%2fResumeReview%2f%2fCertificate_for_sign.cer 80 - 143.198.231.177 Mozilla/5.0+(X11;+Linux+x86_64;+rv:128.0)+Gecko/20100101+Firefox/128.0 http://resumeupload.wowza.edu/resumes/Resume9_eba15ba0-81ca-4d0f-9fad-3fc1fc92c181.pdf?fdir=C%3a%2fShares%2f%2fResumeReview 200 0 0 24


7. Who was the first ideal candidate the company was seeking?



8. A file was dropped to perform a potential Kerberoasting attack in the Active Directory environment. What is the name of this file?

9. Which service account was targeted by this active directory attack?

10. Attacker dropped another tool to exploit a misconfiguration in AD CS. When was this tool successfully created on the endpoint?

11. When was AD CS exploited successfully on the domain controller?

12. Identify the name and version of the vulnerable certificate template exploited in the attack.

13. A new security descriptor was applied to the vulnerable certificate template and this security descriptor grants full control over the certificate template to a specific group. What is the name of that group?

14. When was the certificate issued to the threat actor for a high-privilege user, allowing privilege escalation?

15. The threat actor successfully retrieved the NTLM hash of a high-privilege user using the newly issued certificate. What was the serial number of the certificate used to achieve this privilege escalation?

16. The attacker exploited an Active Directory misconfiguration to gain access to Active Directory secrets stored on the domain controller. What is the MITRE ATT&CK ID for this technique?

17. When was the attack mentioned in the previous task successfully carried out?

18. A Golden Ticket was issued using a previously dropped tool. What is the filename of this ticket?


19. The threat actor used the web shell to delete all files involved in this operation. When did the deletion routine begin?

