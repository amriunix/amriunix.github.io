--- 
draft: false
date: 2024-03-01T15:20:18+02:00
title: "OffSec EXP-401 (AWE) Course Review"
description: "OffSec EXP-401 Advanced Windows Exploitation (AWE) course covers a range of topics essential for mastering Windows exploitation."
slug: "offsec-exp-401-awe-course-review"
authors: ["Oussama Amri"]
tags: ["EXP-401", "Exploit Development", "CVE"]
categories: []
externalLink: ""
series: []
---

In February 2024, I had the privilege of attending the OffSec EXP-401 Advanced Windows Exploitation (AWE) course at QA.com in London. This review aims to provide a detailed day-by-day breakdown of the course content, offering insights into one of the most challenging and comprehensive advanced exploitation courses available.

{{< figure src="/img/offsec-exp-401-awe-course-review/AWE-WELCOME.png" width="70%" >}}

## **Course Overview**

The AWE course is designed to push even experienced penetration testers to their limits. Spanning five days, the course covers a range of topics essential for mastering Windows exploitation. The curriculum is split into five core modules, each focusing on different aspects of advanced Windows exploitation. From custom shellcode creation to kernel-mode exploits, the course demands a high level of prior knowledge and a strong determination to succeed.

{{< figure src="/img/offsec-exp-401-awe-course-review/AWE-WTF-LEVEL.png" width="60%" >}}

### **Motivation for Taking the Course**

For me, taking the AWE course was more about fulfilling a long-standing goal. Having completed nearly all of the OffSec certifications, this was one of the few remaining challenges. Even though my current work is not directly related to this type of area, I firmly believe that the more knowledge you acquire, the better you become. Among the topics covered in the course, I found the Edge Internals section particularly useful for my professional work. This topic was directly relevant to some of the challenges I face in my current role, making it a valuable addition to my skill set.

### **Course Prerequisites**

Before diving into the content of the AWE course, it’s crucial to highlight the prerequisites that are expected of participants. These prerequisites are essential for keeping pace with the material and successfully completing the hands-on exercises:

- **WinDbg Proficiency:** While not explicitly stated as a requirement, mastering WinDbg is crucial for kernel debugging, which plays a significant role in the course. Unfortunately, this was my only weak point, and I recommend becoming comfortable with this tool beforehand.
  
- **Basic Knowledge of Windows x86_64 Architecture and x64 Shellcode Creation:** A solid understanding of Windows architecture and experience in creating x64 shellcode is foundational. The course assumes familiarity with these topics as you will be building upon this knowledge from the first day.

- **Hands-on Exploitation Experience:** You should have prior experience writing exploits. The course will not cover basic exploitation techniques, so you should already be comfortable with them.

- **Static Analysis with IDA Free:** Being familiar with static analysis, particularly using IDA Free, is important. The course often requires you to analyze binaries and reverse-engineer components, so a solid grasp of IDA is beneficial.

- **Familiarity with C/C++ Programming and Visual Studio:** Since many of the exercises involve writing and modifying code in C/C++, you should be comfortable with these languages and the Visual Studio environment.

- **Hardware Requirements:** The course suggests bringing a laptop with the following specifications:
  - **Operating System:** Windows 10 or higher.
  - **Storage:** At least 200 GB of free usable storage.
  - **Processor:** A modern system with a 64-bit CPU, with at least 4 cores that support NX (No eXecute), SMEP (Supervisor Mode Execution Prevention), and VT-x/EPT functionalities.
  - **Memory:** A minimum of 16 GB of RAM.

Meeting these prerequisites is crucial for fully benefiting from the course and keeping up with the intensive material.

## **The Journey**


### **Day 1 – Custom Shellcode Creation and VMware Workstation Internals**

We began by creating custom x64 shellcode, which served as a foundation for more advanced topics later in the week. The course covered 64-bit architecture, including memory enhancements and calling conventions, before moving on to writing exploit code.

The real challenge of Day 1 was the deep dive into VMware Workstation internals. We explored the VMware Backdoor RPC mechanism, focusing on a User-After-Free (UaF) vulnerability. Despite the complexity, the course materials, including discussions on Front-End and Back-End allocators and the Windows Heap Memory Manager, made it possible to follow along, even for those with limited hypervisor experience.

### **Day 2 – VMware Workstation Guest-to-Host Escape and Edge Internals**

Building on the previous day’s work, Day 2 focused on the continuation of the VMware guest-to-host escape. We delved deeper into the exploitation process, covering topics such as Low Fragmentation Heap (LFH) architecture and the detailed steps involved in triggering and controlling a UaF vulnerability within VMware Workstation.

A significant portion of the day was dedicated to bypassing Windows Defender Exploit Guard (WDEG), specifically focusing on defeating Export Address Filtering (EAF), bypassing modern exploit mitigations, including Data Execution Prevention (DEP) and Address Space Layout Randomization (ASLR). We used a combination of ROP and COP to achieve a successful guest-to-host exploit, resulting in a reverse shell on the VMware host. 

A lesser-documented countermeasure that added an extra layer of challenge. The day also introduced Chakra Core internals, laying the groundwork for more advanced browser exploitation techniques that would be covered later in the week.


### **Day 3 – Edge Exploitation and Sandbox Escape**

Day 3 shifted the focus to Microsoft Edge internals, specifically type confusion vulnerabilities within the Chakra JavaScript engine. The course provided an in-depth analysis of Just-In-Time (JIT) compilation and how it can be exploited to control memory. We learned to exploit type confusion by controlling the auxSlots pointer, which allowed us to create read and write primitives necessary for further exploitation.

The course also covered various mitigations, such as Control Flow Guard (CFG) and Arbitrary Code Guard (ACG), with practical exercises on bypassing these protections. The day concluded with a hands-on sandbox escape exercise, demonstrating how to break out of Edge’s sandbox and execute arbitrary code on the host system.


### **Day 4 – Windows Kernel Internals and Driver Callback Overwrite**

The fourth day delved into even more advanced exploitation techniques, and introducing kernel-mode debugging. The day began with an introduction to kernel-mode shellcode, covering crucial topics like token stealing and manipulating Access Control Lists (ACLs). We explored CFG bypass techniques, including return address overwriting and leveraging Intel CET (Control-flow Enforcement Technology) to execute out-of-context calls.

Kernel-mode debugging was a highlight of the day, with exercises involving remote debugging over TCP/IP and using VirtualKD for local kernel debugging within VMware. This segment was particularly challenging as it required a deep understanding of Windows internals and the ability to navigate kernel-mode execution.


### **Day 5 – Native Windows Kernel Exploitation**

The final day of the course was the most intense, focusing entirely on native Windows kernel exploitation. The course then moved on to more advanced techniques, such as stack pivoting, kernel read/write primitives, and bypassing Supervisor Mode Execution Prevention (SMEP).

A particularly challenging section involved exploiting a vulnerability by redirecting execution from kernel mode to user mode while overcoming security features like Kernel Virtual Address Shadowing (KVA Shadow) and flipping PML4 (Page Map Level 4) bits to control memory paging structures. This day was the culmination of all the skills and knowledge gained throughout the week, pushing participants to apply everything they had learned in a complex, real-world scenario.


## **Offensive Security Exploitation Expert (OSEE)**
Offensive Security Exploitation Expert (OSEE) is an OffSec certification you obtain after completing the course EXP-401: Advanced Windows Exploitation (AWE) and passing a 72-hour grueling and hard exam.

### **Preparation**

While I haven’t yet taken the OSEE exam, it’s known within the community that the exam can be straightforward if you approach it with thorough preparation. Based on the recommendations of those who have taken the exam, here are some tips to help you prepare:

1. **Read the Course Material Thoroughly:** The AWE course book is your best friend. Make sure you understand each concept before moving on. The exam will test your ability to apply the knowledge covered in the course, so a deep understanding is essential.

2. **Repeat All Proof-of-Concepts (PoCs):** The hands-on exercises provided in the course are designed to reinforce your understanding of the material. Don’t just go through them once—repeat them until you can execute them confidently without referring to the instructions.

3. **Practice All Exercises:** In addition to the PoCs, ensure you complete all exercises provided in the course. The more familiar you are with the types of challenges presented, the better prepared you will be for the exam.

4. **Master the Tools:** Ensure you’re proficient with the tools used in the course, especially WinDbg, IDA Pro, and Visual Studio. These tools are integral to the exploitation techniques you’ll need to demonstrate during the exam.

By following these preparation steps, you can increase your confidence and readiness for the OSEE exam. While challenging, the exam is a testament to your skills and knowledge in advanced Windows exploitation.


## **Conclusion**

The OffSec EXP-401 Advanced Windows Exploitation course is a true test of one’s skills and knowledge in Windows exploitation. It’s a course that requires significant preparation and a solid understanding of both user-mode and kernel-mode exploitation techniques. Each day builds on the previous one, culminating in a comprehensive mastery of Windows exploitation.

For those aiming to achieve the OSEE certification, this course is an essential step. However, the knowledge and experience gained are invaluable even beyond the certification. If you are serious about advancing your Windows exploitation skills, the AWE course is a must.

I would recommend this course to anyone with a strong background in exploitation and a desire to push their skills to the next level. It’s an intense, rewarding experience that will leave you with a deep understanding of advanced Windows exploitation techniques.

