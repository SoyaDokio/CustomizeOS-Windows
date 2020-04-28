## Make win10 taskbar buttons Never combine, hide labels

> Forward from: https://gist.github.com/blole/428d67218642379489fe

Normally, this option isn't available, and the valid options for how the taskbar button should behave are:

option | registry value	| hide bit | combine bit
-|:-:|:-:|:-:
Never combine | 2 | 0 | 0
Combine when taskbar is full | 1 | 0 | 1
Always combine, hide labels | 0 | 1 | 1

These options are set in `Taskbar and Start Menu Properties`, accessible by right clicking the taskbar and selecting Properties.

Changing the option will modify the registry keys 
```
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced\TaskbarGlomLevel
```
and
```
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced\MMTaskbarGlomLevel
```
where the first controls the taskbar on your main screen, and the second on all other screens.

These registry values are later read and translated into a hide bit and combine bit by explorer.exe. The function responsible for that translation looks like this:

```Assembly
0x7FF69EA67590:
explorer.exe+57590 - 48 89 5C 24 08        - mov [rsp+08],rbx
explorer.exe+57595 - 57                    - push rdi
explorer.exe+57596 - 48 83 EC 20           - sub rsp,20
explorer.exe+5759A - 48 8B F9              - mov rdi,rcx
explorer.exe+5759D - 33 DB                 - xor ebx,ebx
explorer.exe+5759F - B9 56000040           - mov ecx,40000056
explorer.exe+575A4 - FF 15 7E011700        - call qword ptr [explorer.exe+1C7728]
explorer.exe+575AA - 85 C0                 - test eax,eax
explorer.exe+575AC - 75 32                 - jne explorer.exe+575E0
explorer.exe+575AE - 4C 8D 4C 24 38        - lea r9,[rsp+38]
explorer.exe+575B3 - 89 5C 24 38           - mov [rsp+38],ebx
explorer.exe+575B7 - 4C 8B C7              - mov r8,rdi
explorer.exe+575BA - 48 8D 15 3F671700     - lea rdx,[explorer.exe+1CDD00]
explorer.exe+575C1 - 48 C7 C1 01000080     - mov rcx,80000001
explorer.exe+575C8 - E8 5BDAFBFF           - call explorer.exe+15028
explorer.exe+575CD - 8B 44 24 38           - mov eax,[rsp+38]
explorer.exe+575D1 - 83 E8 01              - sub eax,01
explorer.exe+575D4 - 74 17                 - je explorer.exe+575ED
explorer.exe+575D6 - 83 F8 01              - cmp eax,01
explorer.exe+575D9 - 74 05                 - je explorer.exe+575E0
explorer.exe+575DB - BB 03000000           - mov ebx,00000003
explorer.exe+575E0 - 8B C3                 - mov eax,ebx
explorer.exe+575E2 - 48 8B 5C 24 30        - mov rbx,[rsp+30]
explorer.exe+575E7 - 48 83 C4 20           - add rsp,20
explorer.exe+575EB - 5F                    - pop rdi
explorer.exe+575EC - C3                    - ret 
explorer.exe+575ED - BB 01000000           - mov ebx,00000001
explorer.exe+575F2 - EB EC                 - jmp explorer.exe+575E0
```

Now, we'd like to "Never combine, hide labels", one way to achieve this is to modify the bits set by one of the existing options, which is what I've done. Changing the instruction

```
explorer.exe+575ED - BB 01000000           - mov ebx,00000001
```
to
```
explorer.exe+575ED - BB 02000000           - mov ebx,00000002
```

This replaces the option `Combine when taskbar is full` with the behavior we want.
