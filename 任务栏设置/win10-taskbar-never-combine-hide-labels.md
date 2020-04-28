## 设置Win10任务栏按钮为 从不合并，隐藏标签

> 本文为翻译 ，适当做了些意译，根据东西思维差异作出一定改动。
> 原文：[Make win10 taskbar buttons Never combine, hide labels](https://gist.github.com/blole/428d67218642379489fe)

微软官方在`设置`->`个性化`->`任务栏`页面的下拉菜单`合并任务栏按钮`中并未提供`从不合并，隐藏标签`选项，而是提供了以下3个选项：

可用选项 | 注册表值	| hide bit | combine bit
-|:-:|:-:|:-:
始终合并按钮 | 0 | 1 | 1
任务栏已满时 | 1 | 0 | 1
从不 | 2 | 0 | 0

> 译者注：为便于理解，两个表头未予翻译，下同。若实难理解，可参考：hide bit = 隐藏（控制）位、combine bit = 合并（控制）位。
> 也可通过右键单击任务栏并选择`任务栏设置`来访问`合并任务栏按钮`

![从不](https://github.com/SoyaDokio/CustomizeOS-Windows/blob/master/res/img/%E4%BB%8E%E4%B8%8D.png)

![始终合并按钮](https://github.com/SoyaDokio/CustomizeOS-Windows/blob/master/res/img/%E5%A7%8B%E7%BB%88%E5%90%88%E5%B9%B6%E6%8C%89%E9%92%AE.png)

通过选择该下拉菜单的选项，将会修改以下注册表项（修改值参考上表）：
```
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced\TaskbarGlomLevel
```
和
```
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced\MMTaskbarGlomLevel
```

前者控制主显示器上的任务栏，后者控制所有其它显示器上的任务栏。
> 译者注：单显示器用户看第一个就好

这些注册表值随后会被 explorer.exe 进程读取并转换为 hide bit 和 combine bit 。负责该转换的函数如下所示：

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

现在，我们的需求是`从不合并，隐藏标签`，一种实现方法是修改下拉菜单`合并任务栏按钮`中某一现有选项的 hide bit 和 combine bit，这是我目前的选择：
将指令

```
explorer.exe+575ED - BB 01000000           - mov ebx,00000001
```
修改为
```
explorer.exe+575ED - BB 02000000           - mov ebx,00000002
```

这一操作会使下拉菜单`合并任务栏按钮`中`任务栏已满时`选项的实际功能变更为`从不合并，隐藏标签`功能。
<br><br><br>
可实现`从不合并，隐藏标签`的第三方工具：[7+ Taskbar Tweaker](http://rammichael.com/7-taskbar-tweaker) （官方支持中文）
