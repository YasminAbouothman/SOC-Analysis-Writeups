# **Windows Logging for SOC - THM Walkthrough**

<img width="1311" height="238" alt="image" src="https://github.com/user-attachments/assets/919b227a-7ea4-4977-a922-9c3a3b5e2a45" />

In this [TryHackMe](https://tryhackme.com/room/windowsloggingforsoc) room, you will learn how to navigate **Windows Event Viewer**, understand the purpose of the different Windows log categories, and identify important Event IDs that are commonly used during security investigations.

In this walkthrough, I will guide you through the room step by step, explaining each task, the key concepts behind it, and how the information can be applied in real-world SOC investigations.

## **Task 1: Introduction**

No answer needed

## **Task 2: What is Logged**

In this task, you’ll learn where Windows stores its event logs and how to view them. The easiest way to access these logs is by using **Event Viewer**, a built in Windows tool that lets you browse and analyze system events.

 ***📂 Windows event logs are stored by default in the following directory:***
```text
C:\Windows\System32\winevt\Logs
```

which can be opened with Event Viewer or analyzed using other log analysis and forensic tools.

<img width="1905" height="585" alt="image" src="https://github.com/user-attachments/assets/ce50b7c3-a2f0-40c1-8d66-555a6763d322" />
&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;


>  By looking at the last screenshot, we can see that the log source is **Security** , and **Event ID 4624** represents a successful logon.

Answer: ``` Security / 4624 ```

## **Task 3: Security Log Authentication**

Let’s assume we are investigating a security incident where an attacker is attempting to perform a brute-force or password spraying attack against an RDP service.

The first step is to examine **Event ID 4625 (Failed Logon)** to identify failed authentication attempts. We should also check **Event ID 4624 (Successful Logon)** to determine whether any of those attempts eventually resulted in a successful login.

When examining these events, the most important fields to check are:

    1. Logon Type: If the attack targets an RDP service, the Logon Type will typically be 3 or 10, depending on whether Network Level Authentication (NLA) is enabled.

    - Logon Type 10 (Remote Interactive): where NLA is disabled.
    - Logon Type 3 (Network): Common on modern Windows systems where NLA is enabled.
&nbsp;&nbsp;&nbsp;

    2. Account Name: This field identifies the user account that is being targeted. 
    It helps you determine which account the attacker is attempting to authenticate with.
    It is also important to note the Logon ID, which is a unique identifier assigned to each logon session. we can use the Logon ID to correlate related events in the Security log.
    For example, you can use it to trace what happened after a successful logon, such as when the user logged off, started a process
    This allows you to build a timeline of the user’s activity and determine whether the account was used maliciously after authentication.

&nbsp;&nbsp;&nbsp;
    
    3. Source Network Address: During an investigation, this is one of the most important fields because it helps identify the attacker’s source IP
&nbsp;&nbsp;&nbsp;

    4. Failure Reason: This field explains why the logon attempt failed.
    
Now that we’ve covered the important fields to examine, let’s answer the questions for this task.

&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;

> Open the **Practice-Security.evtx** file on the VM’s Desktop.
Which IP performed a brute force of the THM-PC?

&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;


*open the Practice-Security.evtx file in Event Viewer, then filter the Security log for Event IDs 4624 and 4625. This will display all successful and failed logon events, making it easier to analyse the authentication activity.*

&nbsp;&nbsp;&nbsp;

<img width="1275" height="538" alt="image" src="https://github.com/user-attachments/assets/ef83ca5b-4bc4-4297-929d-19f73f3e10b4" />

&nbsp;&nbsp;&nbsp;

Sort the events by the **Date and Time** column in **ascending order (↑)** to display the oldest events first. This makes it easier to follow the sequence of authentication attempts from the beginning of the attack.

&nbsp;&nbsp;&nbsp;

<img width="1272" height="529" alt="image" src="https://github.com/user-attachments/assets/1681924a-5234-4884-bb9a-0b4f7013d1c3" />

&nbsp;&nbsp;&nbsp;

After sorting the events by **Date and Time** in ascending order, we can reconstruct the attack timeline.

The first **Failed Logon (Event ID 4625)** occurred at ***10:53:26*** for the account ```thmadmin``` The attacker then attempted to authenticate with several other accounts in quick succession:
&nbsp;&nbsp;&nbsp;

- ***10:53:27 PM*** — ```superuser``` (Event ID 4625)
- ***10:53:27 PM*** — ```helpdesk``` (Event ID 4625)
- ***10:53:28 PM*** — ```support``` (Event ID 4625)
- ***10:53:30 PM*** — ```Administrator``` (Event ID 4625)
  
&nbsp;&nbsp;&nbsp;

Immediately after the failed logon attempt against the ```Administrator``` account, we observe a **Successful Logon (Event ID 4624)** for the same account at ***10:53:30 PM***

This sequence of events suggests that the attacker tried multiple accounts in a short period of time and eventually succeeded in authenticating as ```Administrator ```.

&nbsp;&nbsp;&nbsp;


<img width="1252" height="346" alt="image" src="https://github.com/user-attachments/assets/f1634f01-075a-40b0-9afe-2c63768d6d54" />
&nbsp;&nbsp;&nbsp;


Now that we’ve identified the accounts targeted by the attacker, the next step is to determine **which IP address** performed the attack.


To do this, examine the **Source Network Address** field in the **Event ID 4625** entries

&nbsp;&nbsp;&nbsp;

<img width="723" height="256" alt="image" src="https://github.com/user-attachments/assets/f2d28a5d-9ca4-4cb1-8964-76d8bce085f1" />

&nbsp;&nbsp;&nbsp;

Answer: ```10.10.53.248```

&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;


> Which user has been breached as a result of the attack?

To answer this question, look for a **Successful Logon (Event ID 4624)** that occurs after the series of **Failed Logon (Event ID 4625)** events.

In the previous step, we observed multiple failed logon attempts against different accounts. Immediately afterward, there is a **Successful Logon (Event ID 4624)** for the ```Administrator``` account.

Answer: ```Administrator```

&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;

> What was the Logon ID of the malicious RDP login?
Note: The login you are looking for has a Logon Type 10.

To answer this question, locate the **Successful Logon (Event ID 4624)** for the ```Administrator``` account that occurred after the password spraying attack.

Next, open the event details and verify that the **Logon Type** is **10** which indicates a successful RDP logon.


<img width="721" height="322" alt="image" src="https://github.com/user-attachments/assets/7e0bc7ec-d9e9-4cf3-b86d-4ae2755d9736" />


## **Task 4: Security Log User Management**

In Windows Security logs, **User Management events** are events that record changes made to user accounts and groups. These events are very important for SOC analysts because attackers often create new accounts, reset passwords, or add users to privileged groups after compromising a system.

Some of the most important User Management Event IDs are:

- **4720** => A user account was created.
- **4732** => A member was added to a security group.
- **4733** => A member was removed from a security group.
- **4723** => generated when a user attempts to change their own password. While this is often a legitimate action, it can also be a sign of malicious activity if a compromised account’s password is changed to maintain unauthorised access.

during an investigation, we come across an unfamiliar account named ```svc_sysrestore```. Earlier, we determined that the attacker successfully compromised the ```Administrator``` account through RDP. After gaining administrative access to ```THM-PC```, the attacker created the ```svc_sysrestore``` account to establish persistence.
  
Now, let’s verify these actions by examining the **User Management events** in the **Windows Security log**.

&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;

> Continue with the “Practice-Security.evtx” file on the VM’s Desktop.
Which user was created by the attacker soon after the RDP login?

open the Practice-Security.evtx file in Event Viewer, then filter the Security log for **Event ID 4624 (Successful Logon)** and Event ID 4720 (A user account was created).


<img width="928" height="538" alt="image" src="https://github.com/user-attachments/assets/93785a61-04e1-47a7-ac38-6056df38975d" />


First, identify the successful **RDP** logon **(Event ID 4624)** for the ```Administrator``` account. Then, look for an **Event ID 4720** that occurs shortly afterward.

<img width="961" height="556" alt="image" src="https://github.com/user-attachments/assets/f839f042-9dff-42fd-b300-51cc1af0b878" />

&nbsp;&nbsp;&nbsp;


The **Successful Logon (Event ID 4624)** we’re looking for occurred at ***10:53:41 PM***. We know this is the correct event because, in the previous task, we identified the ```Administrator``` account’s Logon ID as ```0x183C36D```.



#### Note: The Logon ID allows SOC analysts to correlate events that belong to the same logon session. Rather than relying solely on the username or timestamp, you can use the Logon ID to track all actions performed after a user successfully authenticates. This makes it much easier to reconstruct the attacker’s activity and build an accurate timeline during an investigation.

&nbsp;&nbsp;&nbsp;

<img width="520" height="192" alt="image" src="https://github.com/user-attachments/assets/8a04605b-b4ba-4c81-98bf-ff5512282999" />

&nbsp;&nbsp;&nbsp;


By matching this Logon ID, we can correlate the **successful RDP session** with subsequent events, such as **Event ID 4720**, to determine which user account the attacker created after gaining access.

&nbsp;&nbsp;&nbsp;

<img width="520" height="205" alt="image" src="https://github.com/user-attachments/assets/5ee4c6c8-2ab6-4e74-bbe2-bfbdd6d9ad1f" />

&nbsp;&nbsp;&nbsp;

Answer: ```svc_sysrestore```

&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;

> Which two privileged groups was the backdoor user added to?
(Answer in alphabetical order, e.g. “Administrators, Power Users”)

filter the Security log for Event IDs **4624** (Successful Logon), **4720** (A user account was created), and **4732** (A member was added to a security-enabled local group).

&nbsp;&nbsp;&nbsp;

<img width="910" height="546" alt="image" src="https://github.com/user-attachments/assets/eff719d6-8761-45ee-a743-82ba59e90440" />

&nbsp;&nbsp;&nbsp;

We previously located the **Successful Logon (Event ID 4624)** for the ```Administrator``` account at ***10:53:41 PM***. Next, we found Event ID **4720** at ***10:54:58 PM***, which shows that the attacker created the  ```svc_sysrestore``` account.

Our next step is to look for Event ID 4732 entries that occur shortly afterward. These events will show which local security groups the attacker added ```svc_sysrestore``` to

<img width="522" height="208" alt="image" src="https://github.com/user-attachments/assets/7120ba05-3572-4ee0-875c-3cf91e677a5a" />

&nbsp;&nbsp;&nbsp;

The first privileged group that the svc_sysrestore account was added to is Backup Operators.

&nbsp;&nbsp;&nbsp;

<img width="522" height="237" alt="image" src="https://github.com/user-attachments/assets/da15c3e7-f61d-4813-a833-4f4b7aab273b" />

&nbsp;&nbsp;&nbsp;

<img width="520" height="186" alt="image" src="https://github.com/user-attachments/assets/e86faf3f-1c8a-4b95-a676-58acbb387a4f" />

&nbsp;&nbsp;&nbsp;

The second **Event ID 4732** shows that the attacker added the ```svc_sysrestore``` account to the Remote Desktop Users group

<img width="516" height="228" alt="image" src="https://github.com/user-attachments/assets/ce71dacc-b8e2-46a9-aff7-2c7ab98de5df" />

&nbsp;&nbsp;&nbsp;

**Answer**: ```Backup Operators, Remote Desktop Users```

&nbsp;&nbsp;&nbsp;

> Does the Logon ID field match what you saw in the previous task (Yea/Nay)?

yea

<img width="247" height="51" alt="image" src="https://github.com/user-attachments/assets/bfdbfce6-ffa4-4288-9bc6-ed5f718cb4ba" />

&nbsp;&nbsp;&nbsp;


## **Task 5: Sysmon Process Monitoring**

Let’s assume that an employee in our organisation, **Sarah**, downloaded and executed a file from the Internet. Shortly afterward, the SOC team observed that Sarah’s workstation was attempting to brute-force the organisation’s production server.

As SOC analysts, our next step is to investigate the processes that were launched on Sarah’s machine to determine whether the downloaded file was responsible for the suspicious activity.

To do this, we can examine **Event ID 4688 (Process Creation)** or **Sysmon Event ID 1 (Process Create)** to identify the executable that was run

&nbsp;&nbsp;&nbsp;

> Open the “Practice-Sysmon.evtx” file on the VM’s Desktop.
Which web browser does Sarah use to browse the web?

&nbsp;&nbsp;&nbsp;

<img width="918" height="540" alt="image" src="https://github.com/user-attachments/assets/74dcfbd7-842b-4588-88a0-cb202b1a7171" />

&nbsp;&nbsp;&nbsp;

We can see that at ***4:02:29 PM*** , Sarah launched Google Chrome, which indicates that she uses Google Chrome to browse the web.

Answer: ```Google Chrome```

&nbsp;&nbsp;&nbsp;

> Which file did Sarah download from the browser?

To determine which file Sarah downloaded from the browser, we need to examine Sysmon **Event ID 15 (FileCreateStreamHash)**, which records files downloaded from the Internet.

&nbsp;&nbsp;&nbsp;

<img width="913" height="465" alt="image" src="https://github.com/user-attachments/assets/c9ddf17b-aeb1-466d-b938-2188f843bd1e" />

&nbsp;&nbsp;&nbsp;

**Answer**: ```C:\Users\sarah.miller\Downloads\ckjg.exe```

> Which URL was the file downloaded from? Note: Use other Sysmon events to find out!

By examining the **Contents field** of the **Event ID 15** entry associated with the downloaded file, we can identify the URL from which the file was downloaded.

&nbsp;&nbsp;&nbsp;

<img width="895" height="517" alt="image" src="https://github.com/user-attachments/assets/a19d3583-8ef2-4128-a32a-d3ec7f0f3aa3" />

&nbsp;&nbsp;&nbsp;

<img width="903" height="370" alt="image" src="https://github.com/user-attachments/assets/c15224d3-5fca-47db-9766-f375aa5d5653" />

&nbsp;&nbsp;&nbsp;

**Answer**: ```http://gettsveriff.com/bgj3/ckjg.exe ```


## **Task 6: Sysmon Files and Network**

So far, we have determined that Sarah was using Google Chrome and downloaded the file **ckjg.exe** from the following URL:

=>    http://gettsveriff.com/bgj3/ckjg.exe

After downloading and executing the file, our next step is to determine whether the malware established persistence on the system.

&nbsp;&nbsp;&nbsp;

> Continue with the “Practice-Sysmon.evtx” file on the VM’s Desktop.
Which file was created by the downloaded malware to persist on the host?

To determine which file the malware created for persistence, we need to examine **Sysmon Event ID 11 (File Create)** events associated with **ckjg.exe**. By filtering for the malware's **Process ID (PID) 4600**, we can identify files created by that process


&nbsp;&nbsp;&nbsp;

<img width="904" height="391" alt="image" src="https://github.com/user-attachments/assets/2fe001b1-016e-45b6-b0ef-2ba6d0e5373e" />

&nbsp;&nbsp;&nbsp;

we can see that the malware created the following file:

```C:\Users\sarah.miller\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\DeleteApp.url```

The file was created in the user’s Startup folder, which means it will automatically execute whenever Sarah logs in to the system.

This is a common **persistence technique** used by malware to maintain access

&nbsp;&nbsp;&nbsp;

> What is the Command & Control server malware connected to?
(Answer in format IP:Port, e.g. 1.1.1.1:80) 

To determine the malware’s **Command & Control (C2) server**, we need to look for **network activity** generated by **ckjg.exe** after it was executed.

By examining the **Sysmon Event ID 3 (Network Connection)** events associated with the malware process, we can identify the external server it communicated with.

&nbsp;&nbsp;&nbsp;

<img width="925" height="397" alt="image" src="https://github.com/user-attachments/assets/c5010e49-3fe2-4a3c-8b32-ed9dcac3e381" />

&nbsp;&nbsp;&nbsp;

The **Destination IP** or Destination Hostname field will reveal the **C2 server** used by the attacker

&nbsp;&nbsp;&nbsp;

> Finally, which domain does the malicious IP correspond to?

the final step is to determine which domain is associated with that **IP address**.

To answer this question, examine the Sysmon DNS **Query events (Event ID 22)** associated with **ckjg.exe**. The queried domain that matches the malicious IP address is the domain used by the Command & Control server

&nbsp;&nbsp;&nbsp;

<img width="915" height="394" alt="image" src="https://github.com/user-attachments/assets/109dee06-b816-4aac-8039-9bffa39d6e82" />

&nbsp;&nbsp;&nbsp;

Answer: ```hkfasfsafg.click```


## **Task 7: Powershell Logging Commands**

In this task, we will learn about the PowerShell History file, which records commands executed in interactive PowerShell sessions and can provide valuable evidence during an investigation.

In some cases, it can be more useful than Sysmon Event ID 1 because it records the exact commands entered by the user or attacker, even if those commands do not appear completely in the process creation logs.

By default, PowerShell stores the command history for interactive sessions in the following file:


```C:\Users\<USER>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt```

&nbsp;&nbsp;&nbsp;

> Review the Administrator’s PS history on the attached VM.
Which PowerShell command was executed first?

&nbsp;&nbsp;&nbsp;

<img width="910" height="507" alt="image" src="https://github.com/user-attachments/assets/0e7cb0b3-657d-4618-b9e3-e6f8d8959e11" />

&nbsp;&nbsp;&nbsp;

By examining the PowerShell History file, we can identify the commands that were executed during the interactive PowerShell session.

&nbsp;&nbsp;&nbsp;
<img width="895" height="493" alt="image" src="https://github.com/user-attachments/assets/ad96fee4-d529-42d0-872f-c0e210cfe3d2" />

&nbsp;&nbsp;&nbsp;

**Answer**: ```Get-ComputerInfo```

&nbsp;&nbsp;&nbsp;

> When did the Administrator run the first PS command? (Format: April 18, 2025)
Note: You might need to right-click the history file and open “Properties” to get the answer!
> 
&nbsp;&nbsp;&nbsp;

<img width="913" height="505" alt="image" src="https://github.com/user-attachments/assets/40869128-4570-40ac-b780-5d62c5c50432" />

&nbsp;&nbsp;&nbsp;

**Answer**: ```May 18, 2025```

&nbsp;&nbsp;&nbsp;

> Can you find the flag stored in the PowerShell history? (Format: THM{…})
Note: You might want to check the PS history of other local users!

Navigating to the PSReadLine directory of another local user (```thm.bob```) to examine the ```ConsoleHost_history.txt```
Press enter or click to view image in full size

&nbsp;&nbsp;&nbsp;

<img width="916" height="457" alt="image" src="https://github.com/user-attachments/assets/f255965b-ead0-4b0f-ab41-ae4512fdc113" />

&nbsp;&nbsp;&nbsp;

<img width="912" height="475" alt="image" src="https://github.com/user-attachments/assets/8b1971b0-3eb5-4edc-a44c-34de5681b9e8" />

&nbsp;&nbsp;&nbsp;

&bull; &bull; &bull; &bull; &bull; &bull;

## **Thank you for reading this walkthrough, and I hope it helped you gain a better understanding of Windows logging for SOC investigations.**
