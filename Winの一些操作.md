# Windowsのcmd命令

### 网络操作:

1. ipconfig命令：

   - ipconfig -all		查看网络信息

   - ipconfig -release	释放ip/tcp

   - ipconfig -renew	重新获取ip/tcp

   - ipconfig -flushdns	刷新dns缓存

     

2. netsh命令设置ip，dns，网关等信息tsh interface ip set address "eth0" static 192.168.10.33 255.255.255.0 192.168.10.2

- netsh interface ip set address "eth0" dhcp	动态获取ip等信息
- netsh interface ip set dnsserver "eth0" static 8.8.8.8	#静态设置DNS服务器

### 用户管理：

1. 查看用户

> net user		#查看当前用户
>
> wmic useraccount get name,sid		#获取计算机中所有用户名称和sid，sid结尾为500开始一般为管理员账户，1000为普通用户账户。

2. 账户管理

   ```
   #各帐户所在注册表的路径
   \HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList\
   #创建一个用户
   net user 名字 /add		
   #创建账户的同时配置密码，注意该方式的密码为明文。
   net user 名字 密码 /add	
   net user 名字 /add *		手动输入密码
   #删除账户
   net user 名字 /del 	
   #修改指定账户的密码
   net user 名字 密码		
   #创建隐藏用户，隐藏后net user无法查看到该账户
   net user 名字$ 密码 /add	
   #将用户添加至管理员组，即提权
   net localgroup administrators 名字 /add	
   ```


3. Windows 组的管理：

   - 概念：一组用户的集合，组中的用户具备所属组的权限

   - ```
     net localgroup name /add	创建组
     net localgroup 组名 用户名 /add	将指定用户添加至指定组
     net localgroup 组名 用户名 /del	从指定组移除
     
     ```


### NTFS权限

一、NTFS权限

1. 文件系统

   - Windows

     - 早期Windows上使用FAT16或FAT32
     - 目前Windows基本使用的都是NTFS
       - ACL（访问控制列表，设置相应权限）
       - EFS（加密文件系统，使用BitLocker进行加密）
       - 压缩及磁盘配额（需要事先开启相应windows功能）
     - ReFS文件系统

     早期FAT系统不支持单个文件超过4G，需要转换文件系统：

     ```
     #在cmd中将FAT转换为NTFS且不影响文件内容,G:为目标盘符
     convert G:/fs:ntfs
     ```

   - Linux

     - swap：交换文件系统，将磁盘部分空间划分至内存使用，及虚拟内存。
     - ext4：

二、NTFS系统文件权限

1. 设置文件权限：

- 读取
- 写入
- 附加数据
- 删除
- 执行

2. 文件夹权限：

- 列出文件夹
- 创建文件夹
- 创建文件
- 删除
- 删除子文件夹和文件

#### NTFS权限规则

一、权限规则

- 权限的累加
  - 个人所有权限和所属组的权限
- 拒绝权限
  - 拒绝权限优先级最高
- 继承权限
  - 文件夹或文件的访问控制列表默认情况下会继承上级文件夹的权限。
- 特殊权限
  - 读取权限
    - 指读取访问控制列表的权限，与文件（夹）读取权限不同
    - 针对于用户想要访问某个文件的内容，此权限必须拥有
  - 更改权限
    - 用户是否可以修改文件或文件夹访问控制列表，易造成不安全因素
    - 如想更改，前提是能读取
  - 取得所有权
    - 能够修改文件或文件夹的所有者
    - 前提必须有读取和修改权限

#### 本地安全策略

一、打开方式

- 开始菜单······>管理工具······>本地安全策略

- 使用命令

  ```powershell
  secpol.msc
  ```

- 从本地组策略进入（本地组策略包含了本地安全策略）

  ```powershell
  gpedit.msc
  ```

二、账户策略

1. 密码策略
2. 账户锁定策略

三、本地策略

1. 审核策略
2. 用户权限分配
3. 安全选项

#### 文件共享

1. 共享权限

   - 一般设置为everyone完全控制

2. 要求为多方处于同一局域网下（使用同一网络的IP地址）

   - 直接可以Ping通对方

3. 访问共享

   ```
   \\192.168.10.53		#192.168.10.53为共享服务器地址
   \\WIN-h2j2k3h4g51		#WIN-h2j2k3h4g51为服务器主机名
   ```

#### 组策略应用

一、

1. 打开组策略：	

```
	gpedit.msc
```

2. 更新组策略：

   ```
   gpupdate.msc
   ```



#### 注册表

一、注册表结构

1. 子树
   - HKEY_LOCAL_MACHINE ：记录关于计算机系统的信息，包括硬件和操作系统数据
   - HKEY_USERS ：记录关于动态加载的用户配置文件和默认配置文件的信息
   - HKEY_CURRENT_USER：HKEY_USERS的子树，它指向"HKEY_USERS\当前用户的安全ID"包含当前以交互方式登录的用户的用户配置文件
   - HKEY_CURRENT_CONFIG：HKEY_LOCAL_MACHINE的子树，指向HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Hardware Profiles\Current包含在启动时有本地计算机系统使用的硬件配置文件的相关信息加载的设备驱动程序，显示时要使用的分辨率
   - HKEY_CLASSES_ROOT：HKEY_CURRENT_USER的子树，包含用于各种OLE技术和文件类关联数据的信息
2. 项
   - 可以简单地理解为文件夹，项中可以包含项和值
3. 值
   - 每个注册表项或子项都可以包含称为值的数据
   - 部分值用于某个用户的信息
   - 部分值应用于计算机所有用户的信息
   - 值有三部分组成（名称，类型，数据）

二、注册表基本操作

1. 创建项
2. 创建值
   - 字符串值（REG_SZ）：固定长度的文本字符串
   - 二进制值（REG_BINARY）：原始二进制数据，多数硬件组件信息都以二进制数据存储
   - DWORD值（REG_DWORD）：数据由4字节长的数来表示，设备驱动程序和服务的很多参数都是这种类型
   - QWORD值（REG_QWORD）：数据由8字节长的数来表示
   - 多字符串值（REG_MULTI_SZ）：多重字符串，包含列表或多值的值通常为该类型
   - 可扩充字符串值（REG_EXPAND_SZ）：长度可变的数据串，该数据类型包含在程序或服务使用该数据使得解析的变量
3. 修改，删除，重命名值

三、注册表维护

1. 注册表遭到破坏后的常见现象
   - 无法启动系统
   - 无法运行或正常运行合法的应用程序
   - 找不到启动系统或运行应用程序所需的文件
   - 没有访问应用程序的权限
   - 不能正确安装或装入驱动程序
   - 不能进行网络连接
   - 注册表条目有错误
2. 遭破坏的原因
   - 应用程序错误：在安装某些软件后，可能出现彼此之间的冲突
   - 驱动程序不兼容
   - 硬件老化等
   - 误操作：误操作是最常见的原因
3. 备份与还原
   - 在regedit中进行导入与导出即可
4. 锁定与解锁注册表
5. 优化注册表
