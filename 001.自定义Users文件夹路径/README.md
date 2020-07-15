# 自定义Users文件夹路径

将默认的用户文件夹（C:\\Users）移动到指定位置，本文选择移动到 D 盘。
> 用户文件夹主要上缓存类文件，如下载、文档、音乐、图片、视频等

### 优势
1. 防止系统盘损坏后用户文件丢失
2. 避免备份系统时系统不够“纯净”
3. 越来越多的用户文件可能导致系统盘容量不足

### 原理
将原文件（夹）移到指定位置，再在原位置类似 Linux 一样创建一个软连接链接到指定位置
注意：指定位置所在磁盘的**文件系统类型必须是NTFS**

### 实操（分3种情形）
#### 1.新系统正在安装时

在安装 Win7/Win10 的过程中，要求输入用户名及密码的时候，先不如输入任何信息，按 `Shift` + `F10` 打开 DOS 窗口(命令行窗口)，执行以下命令：

```DOS
## 复制C:\Users下所有文件(包含子文件夹)到D:\Users
robocopy "C:\Users" "D:\Users" /E /COPYALL /XJ
## 删除C:\Users文件夹 
rmdir "C:\Users" /S /Q 
## 创建(目录)软连接 C:\Users 指向 D:\Users
mklink /J "C:\Users" "D:\Users"
```

然后关闭 DOS 窗口，按正常流程继续安装 Windows 直至完成。如此安装的 Windows 所有“用户文件夹”(User Special Folder)的内容都已经被设置在D盘。

#### 2.已使用过的系统 - 维护模式

开机时按 `F8` 键，选择 *Repair your computer/维护模式* ，回车进入。连续点 *Next*/*OK* 两次后，选择 *Command Prompt* （命令提示符），然后执行以下命令：
```DOS
## 复制C:\Users下的所有文件到D:\Users
## 参数说明：此命令为Windows的“强健文件拷贝”命令。
##      /E 表示拷贝文件时包含子目录（包括空目录）
##      /COPYALL 表示拷贝所有文件信息
##      /XJ 表示不包括Junction points（默认是包括的）
##      /XD "C:\Users\Administrator" 表示不包括指定的目录,此处指定目录为："C:\Users\Administrator"
robocopy "C:\Users" "D:\Users" /E /COPYALL /XJ /XD "C:\Users\Administrator"
## 删除C:\Users文件夹 
## 参数说明：此命令删除指定目录。
##      /S 删除指定目录及其中的所有文件。用于删除目录树。
##      /Q 安静模式。删除时不询问。  
rmdir "C:\Users" /S /Q   
## 创建(目录)软连接 C:\Users 指向 D:\Users
## 参数说明：此命令创建符号连接。
##      /J 连接类型为目录连接
mklink /J "C:\Users" "D:\Users"
```
执行完成后，重启系统即可生效。

#### 3.已使用过的系统 - 超级管理员

> 使用超级管理员用户来操作，其前提是之前未使用过默认的超级管理员用户（即 Administrator），若之前直接使用超级管理员用户，则本方案无效。

首先关闭所有应用程序，然后右键我的电脑-管理-系统工具-本地用户和组-用户，在右侧找到 Administrator 右键-属性，取消勾选 *账户已禁用* ，确定。

然后注销当前用户，以 Administrator 登录，打开命令提示符，执行以下命令：

```DOS
robocopy "C:\Users" "D:\Users" /E /COPYALL /XJ /XD "C:\Users\Administrator"
```

注销 Administrator ，重新用你的用户名登录系统，而后跟上面启用超管账户类似，去禁用 Administrator。

然后以管理员身份打开一个DOS窗口，输入以下命令:
```DOS
rmdir "C:\Users" /S /Q
mklink /J "C:\Users" "D:\Users"
```
最后重启电脑即可生效。
