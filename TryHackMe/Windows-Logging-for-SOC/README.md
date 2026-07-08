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
> &nbsp;&nbsp;&nbsp;
> By looking at the last screenshot, we can see that the log source is **Security** , and **Event ID 4624** represents a successful logon.

Answer: ``` Security / 4624 ```
