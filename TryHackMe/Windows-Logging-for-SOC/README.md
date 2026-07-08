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


## **Task 4: Security Log User Management**

## **Task 5: Sysmon Process Monitoring**

## **Task 6: Sysmon Files and Network**

## **Task 7: Powershell Logging Commands**







