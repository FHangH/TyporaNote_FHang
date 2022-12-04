# Linux



## 1.0

### 一、填空题

1．GNU的含义是**GNU's Not Unix**的递归缩写。

2．Linux一般有3个主要部分：**内核（kernel）**、**命令解释层（Shell或其他操作环境）**、**实用工具**。

3．**/etc/sysconfig/network**文件主要用于设置基本的网络配置，包括主机名称、网关等。

4．一块网卡对应一个配置文件，配置文件位于目录**/etc/sysconfig/network-scripts**中，文件名以**ifcfg-**开始。

5．**/etc/resolv.conf**文件是DNS客户端用于指定系统所用的DNS服务器的IP地址。

6．POSIX是**便携式操作系统接口**的缩写，重点在规范核心与应用程序之间的接口，这是由美国电气与电子工程师学会（IEEE）所发布的一项标准。

7．当前的Linux常见的应用可分为**企业应用**与**个人应用**两个方面。

8．Linux的版本分为**内核版本**和**发行版本**两种。

9．安装Linux最少需要两个分区，分别是**swap交换分区**  ，**/（根）**分区。

10．Linux默认的系统管理员账号是**root**。

### 二、选择题

1．Linux最早是由计算机爱好者（  ）开发的。

  A．Richard Petersen

  **B．Linus Torvalds** 

  C．Rob Pick

  D．Linux Sarwar

2．下列中（  ）是自由软件。

A．Windows XP

B．UNIX

**C．Linux**

D．Windows 2008

3．下列中（  ）不是Linux的特点。

  A．多任务

 **B．单用户** 

 C．设备独立性  

 D．开放性

4．Linux的内核版本2.3.20是（  ）的版本。

**A．不稳定**   

B．稳定的  

C．第三次修订  

D．第二次修订

5．Linux安装过程中的硬盘分区工具是（  ）。

  A．PQmagic       

B．FDISK      

 C．FIPS             

**D．Disk Druid**

6．Linux的根分区系统类型可以设置成（  ）。

  A．FATl6         

B．FAT32        

**C．ext4**              

D．NTFS

7．以下哪个命令能用来显示server当前正在监听的端口? （  ）

  A．ifconfig          

B．netlst         

C．iptables       

**D．netstat**

8．以下哪个文件存放机器名到IP地址的映射? （  ）

  A．**/etc/hosts**      B．/etc/host      C．/etc/host.equiv     D．/etc/hdinit

9．Linux系统提供了一些网络测试命令，当与某远程网络连接不上时，就需要跟踪路由查看，以便了解在网络的什么位置出现了问题，请从下面的命令中选出满足该目的的命令。（  ）

  A．ping           B．ifconfig       **C．traceroute**       D．netstat

 

### 三、补充表格

请将nmcli命令的含义列表补充完整。

| nmcli connection show          | 显示所有连接                                  |
| ------------------------------ | --------------------------------------------- |
| nmcli connection show --active | 显示所有活动的连接状态                        |
| nmcli connection show "ens33"  | 显示网络连接配置                              |
| nmcli device status            | 显示设备状态                                  |
| nmcli device show ens33        | 显示网络接口属性                              |
| nmcli connection add help      | 查看帮助                                      |
| nmcli connection reload        | 重新加载配置                                  |
| nmcli connection down test2    | 禁用test2的配置，注意一个网卡可以有多个配置。 |
| nmcli connection up test2      | 启用test2的配置                               |
| nmcli device disconnect ens33  | 禁用ens33网卡，物理网卡                       |
| nmcli device connect ens33     | 启用ens33网卡                                 |

 

 

### 四、简答题（部分）

1．简述Linux的体系结构。

**Linux是由内核、bootloader、文件系统，Shell和应用程序构成**



2．使用虚拟机安装Linux系统时，为什么要先选择稍后安装操作系统，而不是去选择RHEL 7系统镜像光盘？

**在配置界面中若直接选择了RHEL 7系统镜像，则VMware Workstation虚拟机会使用内置的安装向导自动进行安装，最终安装出来的系统跟我们后续进行实验所需的系统环境会不一样。**



3．简述RPM与Yum软件仓库的作用。

**RPM是为了简化安装的复杂度，而Yum软件仓库是为了解决软件包之间的依赖关系。**



4．安装Red Hat Linux系统的基本磁盘分区有哪些?

 **/swap, /, /boot, /var, /tmp, /home, /usr**



5．Red Hat Linux系统支持的文件类型有哪些？

**纯文本文件, 二进制文件, 数据格式的文件, 目录文件, 连接文件**



6．RHEL 7系统采用了systemd作为初始化进程，那么如何查看某个服务的运行状态？

**执行命令“systemctl status 服务名.service”可查看服务的运行状态，其中服务名后的.service可以省略。**

 

## 2.7  练习题

### 一、填空题

1．在Linux系统中命令区分大小写。在命令行中，可以使用**Tab**键来自动补齐命令。

2．如果要在一个命令行上输入和执行多条命令，可以使用**分号**来分隔命令。

3．断开一个长命令行，可以使用  **” \ “**  ，以将一个较长的命令分成多行表达，增强命令的可读性。执行后，Shell自动显示提示符 **">"** ，表示正在输入一个长命令。

4．要使程序以后台方式执行，只需在要执行的命令后跟上一个 **"&"** 符号。

### 二、选择题

1．（  ）命令能用来查找在文件TESTFILE中包含4个字符的行。

  A．grep '???? ' TESTFILE                

B．grep '…. ' TESTFILE

  **C．grep '^????$' TESTFILE**           

D．grep '^….$ ' TESTFILE

2．（  ）命令用来显示/home及其子目录下的文件名。

  A．ls -a /home     

**B．ls -R /home**    

C．ls -l /home 

D．ls -d /home

3．如果忘记了ls命令的用法，可以采用（  ）命令获得帮助。

  A．？ls           

B．help ls        

**C．man ls**        

D．get ls

4．查看系统当中所有进程的命令是（  ）。

  A．ps all          

B．ps aix             

C．ps auf            

**D．ps aux** 

5．Linux中有多个查看文件的命令，如果希望在查看文件内容过程中用光标可以上下移动来查看文件内容，则符合要求的那一个命令是（  ）。

  A．cat            

B．more         

**C．less**          

D. head

6．（  ）命令可以了解您在当前目录下还有多大空间。

  A．df                

 B．du  /        

**C．du  .**         

D．df .

7．假如需要找出 /etc/my.conf 文件属于哪个包（package），可以执行（  ）命令。

  A．rpm -q /etc/my.conf               

B．rpm -requires /etc/my.conf 

  **C．rpm -qf /etc/my.conf**              

D．rpm -q | grep /etc/my.conf 

8．在应用程序启动时，（  ）命令设置进程的优先级。

  A．priority           

 **B．nice**          

C．top           

D．setpri

9．（  ）命令可以把f1.txt复制为f2.txt。

  A．cp f1.txt | f2.txt                   

B．cat f1.txt | f2.txt 

  **C．cat f1.txt > f2.txt**                 

D．copy f1.txt | f2.txt

10．使用（  ）命令可以查看Linux的启动信息。

   A．mesg –d      

**B．dmesg**        

C．cat /etc/mesg   

D．cat /var/mesg



## 3.3 练 习 题

一、填空题

1．由于核心在内存中是受保护的区块，因此我们必须通过**shell**将我们输入的命令与Kernel沟通，以便让Kernel可以控制硬件正确无误地工作。

2．系统合法的shell均写在**/etc/shells**文件中。 

3．用户默认登录取得的shell记录于**/etc/passwd**的最后一个字段。 

4．bash的功能主要有**命令编辑功能；命令与文件补全功能；命令别名设置功能；作业控制、前台与后台控制；程序化脚本；通配符**等。 

5．shell变量有其规定的作用范围，可以分为**全局变量**与**局部变量**。

6．**set**可以观察目前bash环境下的所有变量。

7．通配符主要有***、?、[]**等。

8．正则表示法就是处理字符串的方法，是以**行**为单位来进行字符串的处理的。

9．正则表示法通过一些特殊符号的辅助，可以让使用者轻易地**查找、删除、替换**某个或某些特定的字符串。

10．正则表示法与通配符是完全不一样的。**通配符（wild card）**代表的是bash操作接口的一个功能，但正则表示法则是一种字符串处理的表示方式。 



二、简述题

1．vim的3种运行模式是什么？如何切换？

**命令模式、插入模式、底行模式**

2．什么是重定向？什么是管道？什么是命令替换？

**把命令执行的结果重新输入到一个文件中;**

**将一个命令的输出，传给另外一个命令，作为另外一个命令的输入;**

**通知所有人**

3．Shell变量有哪两种？分别如何定义？

**私有变量: A1="1234” delcare A2="2345"**

**环境变量：A1="1234" export A1** 

4．如何设置用户自己的工作环境？

**~/.profile**

 

## 4.8 练习题

一、填空题

1．Linux操作系统是**多用户多任务**的操作系统，它允许多个用户同时登录到系统，使用系统资源。

2．Linux系统下的用户账户分为两种：**普通用户**和**超级用户（root）**。

3．root用户的UID为　**0**　，普通用户的UID可以在创建时由管理员指定，如果不指定，用户的UID默认从**500**开始顺序编号。

4．在Linux系统中，创建用户账户的同时也会创建一个与用户同名的组群，该组群是用户的**主组群**。普通组群的GID默认也从**500**开始编号。

5．一个用户账户可以同时是多个组群的成员，其中某个组群是该用户的**主组群**（私有组群），其他组群为该用户的**附属组群**（标准组群）。

6．在Linux系统中，所创建的用户账户及其相关信息（密码除外）均放在**/etc/passwd**配置文件中。

7．由于所有用户对/etc/passwd文件均有**读取**权限，为了增强系统的安全性，用户经过加密之后的口令都存放在**/etc/shadow**文件中。

8．组群账户的信息存放在**/etc/group**文件中，而关于组群管理的信息（组群口令、组群管理员等）则存放在**/etc/gshadow**文件中。

 

二、选择题	

1．哪个目录存放用户密码信息？（  ）

  **A．/etc**            

B．/var          

C．/dev          

D．/boot

2．请选出创建用户ID 是200、组ID是1000、用户主目录为/home/user01的正确命令。（  ）

  A．useradd -u:200 -g:1000 -h:/home/user01 user01

  B．useradd -u=200 -g=1000 -d=/home/user01 user01

  **C．useradd -u 200 -g 1000 -d /home/user01 user01**

  D．useradd -u 200 -g 1000 -h /home/user01 user01

3．用户登录系统后首先进入下列哪个目录?（  ）

  A．/home                          

B．/root的主目录 

  C．/usr                            

**D．用户自己的家目录**

4．在使用了shadow口令的系统中，/etc/passwd和/etc/shadow两个文件的权限正确的是（  ）。

  A．-rw-r----- , -r--------       

B．-rw-r--r-- , -r--r--r—

  **C．-rw-r--r-- , -r--------**       

D．-rw-r--rw- , -r-----r—

5．下面哪个参数可以删除一个用户并同时删除用户的主目录？（  ）

  A．rmuser –r       

B．deluser –r     

**C．userdel –r**     

D．usermgr -r

6．系统管理员应该采用哪些安全措施?（  ）

  A．把root密码告诉每一位用户         

  B．设置telnet服务来提供远程系统维护

  **C．经常检测账户数量、内存信息和磁盘信息**  

  **D．当员工辞职后，立即删除该用户账户**

7．在/etc/group中有一行students::600:z3,14,w5，表示有多少用户在student组里？（  ）

  **A．3**              

B．4            

C．5            

D．不知道

8．下列的哪些命令可以用来检测用户lisa的信息？（  ）

  **A．finger lisa**                             

 **B．grep lisa /etc/passwd**   

  C．find lisa /etc/passwd                   

 D．who lisa



## 5.4 练习题

### 一、选择题

\1. 假定kernel支持vfat分区，下面哪一个操作是将/dev/hda1，一个Windows分区加载到/win目录？（ D）

A. mount  -t windows  /win  /dev/hda1 

B. mount  -fs=msdos /dev/hda1  /win

C. mount  -s  win   /dev/hda1 /win  

**D. mount –t  vfat  /dev/hda1  /win**

\2. 请选择关于/etc/fstab的正确描述。（ B ）

A. 启动系统后，由系统自动产生。

**B. 用于管理文件系统信息。**

C. 用于设置命名规则，是否使用可以用TAB来命名一个文件。

D. 保存硬件信息。  

\3. 存放Linux基本命令的目录是什么（ A）

**A. /bin**   

B. /tmp   

C. /lib  

D. /root

\4. 对于普通用户创建的新目录，哪个是缺省的访问权限？（ A ）

**A. rwxr-xr-x**  

B. rw-rwxrw-   

C. rwxrw-rw-  

D. rwxrwxrw-

\5. 如果当前目录是/home/sea/china，那么“china”的父目录是哪个目录？（ A）

**A. /home/sea**   

B. /home/   

C. /   

D. /sea

\6. 系统中有用户user1和user2，同属于users组。在user1用户目录下有一文件file1，它拥有644的权限，如果user2想修改user1用户目录下的file1文件，应拥有（ B）权限？ 

A. 744   

**B. 664**   

C. 646   

D. 746

\7. 在一个新分区上建立文件系统应该使用命令（ C ）

A. fdisk   

B. makefs   

**C. mkfs**   

D. format

\8. 用ls –al 命令列出下面的文件列表，问哪一个文件是符号连接文件？（ D）

A. -rw------- 2 hel-s users  56 Sep 09 11:05 hello

B. -rw------- 2 hel-s users  56 Sep 09 11:05 goodbey

C. drwx----- 1 hel  users 1024 Sep 10 08:10 zhang

**D. lrwx----- 1 hel users 2024  Sep 12 08:12  cheng**

\9. Linux文件系统的目录结构是一棵倒挂的树，文件都按其作用分门别类地放在相关的目录中。现有一个外部设备文件，我们应该将其放在（C ）目录中。

A. /bin   

B. /etc   

**C. /dev**   

D. lib 

\10. 如果umask设置为022，缺省的创建的文件的权限为：（ D ）

A. ----w--w- 

B.  –rwxr-xr-x 

C. r-xr-x--- 

**D. rw-r--r--**

### 二、填空题

\1. 文件系统（File System）是磁盘上有特定格式的一片区域，操作系统利用文件系统**保存和管理**文件。

\2. ext文件系统在1992年4月完成。称为**扩展文件系统**，是第一个专门针对Linux操作系统的文件系统。Linux系统使用**ext2/ext3/ext4**文件系统。

\3. **ISO 9660**是光盘所使用的标准文件系统。

\4. Linux的文件系统是采用阶层式的树状目录结构，在该结构中的最上层是根目录**“/”**。

\5. 默认的权限可用**umask**命令修改，用法非常简单，只需执行**“umask 777”**命令，便代表屏蔽所有的权限，因而之后建立的文件或目录，其权限都变成000。

\6. 在Linux系统安装时，可以采用**Disk Druid、RAID和LVM**等方式进行分区。除此之外，在Linux系统中还有**fdisk、cfdisk、parted**等分区工具。

\7. RAID（Redundant Array of Inexpensive Disks），中文全称是**独立磁盘冗余阵列**，用于将多个廉价的小型磁盘驱动器合并成一个**磁盘阵列**，以提高存储性能和**容错**功能。RAID可分为**软RAID**和**硬RAID**，软RAID通过软件实现多块硬盘**冗余**。

\8. LVM（Logical Volume Manager）的中文全称是**逻辑卷管理器**，最早应用在IBM AIX系统上。它的主要作用是**动态分配磁盘分区**及调整磁盘分区大小，并且可以让多个分区或者物理硬盘作为**一个逻辑卷（相当于一个逻辑硬盘）**来使用。

\9. 可以通过**索引节点数和磁盘块区数**来限制用户和组群对磁盘空间的使用。

 

### 三、简答题

1． RAID技术主要是为了解决什么问题呢？

**答：RAID技术可以解决存储设备的读写速度问题及数据的冗余备份问题。**

 

2． RAID 0和RAID 5哪个更安全？

**答：RAID 0没有数据冗余功能，因此RAID 5更安全。**

 

3．位于LVM最底层的是物理卷还是卷组？

**答：最底层的是物理卷，然后在通过物理卷组成卷组。**

 

4． LVM对逻辑卷的扩容和缩容操作有何异同点呢？

**答：扩容和缩容操作都需要先取消逻辑卷与目录的挂载关联；扩容操作是先扩容后检查文件系统完整性，而缩容操作为了保证数据的安全，需要先检查文件系统完整性再缩容。**

 

5． LVM的快照卷能使用几次？

**答：只可使用一次，而且使用后即自动删除。**

 

6． LVM的删除顺序是怎么样的？

**答：依次移除逻辑卷、卷组和物理卷。**

 

## 6.4 练习题

一、选择题

\1. TCP/IP中，哪个协议是用来进行IP地址自动分配的？（ C ）

A. ARP 

B.  NFS 

**C.  DHCP** 

D. DDNS



\2. DHCP租约文件默认保存在（ D ）目录中。

A. /etc/dhcpd  

B. /var/log/dhcpd 

C. /var/log/dhcp  

**D. /var/lib/dhcp**



\3. 配置完DHCP服务器，运行（ AB ）命令可以启动DHCP服务。

**A. service dhcpd start**  

**B. /etc/rc.d/init.d/dhcpd start** 

C. start dhcpd      

D. dhcpd on

 

二、填空题

1．DHCP工作过程包括**DHCP Discover  DHCP offer  DHCP Request  DHCP Acknowledge 4**种报文。

2．如果DHCP客户端无法获得IP地址，将自动从**169.254.0.0/16**地址段中选择一个作为自己的地址。

3．在Windows环境下，使用**ipconfig**命令可以查看IP地址配置，使用**ipconfig/release**命令可以释放IP地址，使用**ipconfig/renew**命令可以续租IP地址。

4．DHCP是一个简化主机IP地址分配管理的TCP/IP标准协议，英文全称是**Dynamic Host Configuration Protocol**，中文名称为**动态主机配置协议**。

5．当客户端注意到它的租用期到了**50％**以上时，就要更新该租用期。这时它发送一个    **DHCP Request**信息包给它所获得原始信息的服务器。

6．当租用期达到期满时间的近 **87.5％** 时，客户端如果在前一次请求中没能更新租用期的话，它会再次试图更新租用期。

7．配置Linux客户端需要修改网卡配置文件，将BOOTPROTO项设置为**dhcp**。



三、实践题

架设一台DHCP服务器，并按照下面的要求进行配置：

**1．为192.168.203.0/24建立一个IP作用域，并将192.168.203.60~192.168.203.200范围内的IP地址动态分配给客户机。**

**2．假设子网的DNS服务器的IP地址为192.168.0.9，网关为192.168.203.254，所在的域为jnrp.edu.cn，将这些参数指定给客户机使用。**

 

## 7.7  练 习 题

一、填空题

1．在Internet中计算机之间直接利用IP地址进行寻址，因而需要将用户提供的主机名转换成IP地址，我们把这个过程称为**域名解析**

2．DNS提供了一个**分级**的命名方案。

3．DNS顶级域名中表示商业组织的是**com**。

4．**A**表示主机的资源记录，**CNAME**表示别名的资源记录。

5．写出可以用来检测DNS资源创建的是否正确的两个工具**ping  nslookup**。

6．DNS服务器的查询模式有**递归查询  转寄查询**。

7．DNS服务器分为四类：**主DNS服务器（Master或Primary） 辅助DNS服务器（Slave或Secondary） 转发DNS服务器 惟高速缓存DNS服务器（Caching-only DNS server）**。

8．一般在DNS服务器之间的查询请求属于**转寄查询**。

二、选择题

1．在Linux环境下，能实现域名解析的功能软件模块是（  ）。

  A．apache         

B．dhcpd            

**C．BIND**        

D．SQUID 

2．www.163.com是Internet中主机的（  ）。

  A．用户名             

B．密码         

C．别名         

**D．IP地址**    

E. FQDN

3．在 DNS服务器配置文件中A类资源记录是什么意思? （  ）

  A．官方信息                       

B．IP地址到名字的映射  

  **C．名字到IP地址的映射**            

D．一个name server的规范

4．在Linux DNS系统中，根服务器提示文件是（   ）。	

  A．/etc/named.ca                    

**B．/var/named/named.ca** 

  C．/var/named/named.local          

 D．/etc/named.local

5．DNS指针记录的标志是（  ）。

  A．A       

 **B．PTR**     

C．CNAME

D．NS

6．DNS服务使用的端口是（  ）。

  **A．TCP 53**        

B．UDP 54       

C．TCP 54       

D．UDP 53

7．以下哪个命令可以测试DNS服务器的工作情况？（  ）。

  **A．dig**            

**B．host**          

**C．nslookup**     

D．named-checkzone

8．下列哪个命令可以启动DNS服务？（  ）

  **A．systemctl start named**             

**B．systemctl restart named**

  C．service dns start                  

D．/etc/init.d/dns start

9．指定域名服务器位置的文件是（  ）。

  A．/etc/hosts                        

B．/etc/networks  

  **C．/etc/resolv.conf**                   

D．/.profile
