# **安全网关项目产品代码仓库**
## **简介**
&emsp;&emsp;安全网关项目产品侧仓库，主要存放应用与安全网关设备上的应用、内核等相关功能的代码。
##  **目录结构**    
<pre>
.                               安全网关产品源代码主目录
├── Common                      产品侧和平台侧公共头文件
├── docs                        Readme 文档相关资源存放目录
├── Platform                    平台侧工程目录
│   ├── build                   平台侧系统构建 Makefile 目录
|   ├── common                  平台侧公共代码目录
│   ├── modules                 平台侧内核态工程源代码目录
│   │   ├── cfgrcv
│   │   ├── conntrack_api
│   │   │   ├── api
│   │   │   ├── demoA
│   │   │   └── test
│   │   ├── netlink_api
│   │   └── pdelivery
│   └── user                    平台侧用户态工程源代码目录
│       ├── cfg2kernel
│       ├── netlink_uapi
│       └── pdeliv_u
├── Product                     产品侧工程目录
│   ├── build                   产品侧系统构建 Makefile 目录
│   ├── common                  产品侧公共代码目录
│   ├── modules                 产品侧内核态工程源代码目录
│   │   └── demo                内核态态应用程序 demo 目录
│   └── user                    产品侧用户态工程源代码目录
│       └── demo                用户态应用程序 demo 目录
└── kernel                      内核源码存放目录
</pre>
	
>#### **注意事项：**
>1. 内核态功能相关代码存放于 modules 目录下  
>2. 用户态功能相关代码存放与 user 目录下
>3. 每个独立的功能以子目录方式存放到适当的目录下
>4. user 与 modules 目录中共有的源代码和头文件以及配置文件必须存放到 common 目录中 
>5. 不得把相同功能、相同定义、相同配置的内容独立放到各自的 user 和 modules 目录中

## **编译方法**
#### 1. 从仓库获取最新代码  
<code>git clone git@git.komect.net:ISG/secogateway.git</code>  
或  
<code>git pull</code>  

#### 2. 安装必要软件(UBuntu)  
<code>sudo ./fsl-qoriq-glibc-x86_64-fsl-toolchain-aarch64-toolchain-2.4.1.sh
sudo apt-get -y install git u-boot-tools device-tree-compiler autoconf curl flex 
sudo apt-get -y install automake dh-autoreconf libssl-dev openssl libpcap-dev bc
sudo apt-get -y install python-pip qemu-utils libncurses5-dev python-crypto bison
</code>  

#### 3. 安装 linux 内核源代码  
<code>sudo mkdir -p /opt/fsl-kernel /opt/fsl-kernel/arm64 /opt/fsl-kernel/x86
sudo chmod 644 /opt/fsl-kernel -R
cp ./kernel/linux-4.14.83 /opt/fsl-kernel/x86 -rf
cp ./kernel/linux-4.14.83 /opt/fsl-kernel/arm64 -rf
</code>  

#### 4. 设置环境变量  
在 ~/.bashrc 文件末尾加上以下几行配置  
<code>export HUACHENG_LINUX_KERNEL=/opt/fsl-kernel/x86/linux-4.9.140
export HUACHENG_ARM64_KERNEL=/opt/fsl-kernel/arm64/linux-4.9.140
export SDKTARGETSYSROOT=/opt/fsl-qoriq/2.4.1/sysroots/aarch64-fsl-linux
export PATH=/opt/fsl-qoriq/2.4.1/sysroots/x86_64-fslsdk-linux/usr/bin:/opt/fsl-qoriq/2.4.1/sysroots/x86_64-fslsdk-linux/usr/sbin:/opt/fsl-qoriq/2.4.1/sysroots/x86_64-fslsdk-linux/bin:/opt/fsl-qoriq/2.4.1/sysroots/x86_64-fslsdk-linux/sbin:/opt/fsl-qoriq/2.4.1/sysroots/x86_64-fslsdk-linux/usr/bin/../x86_64-fslsdk-linux/bin:/opt/fsl-qoriq/2.4.1/sysroots/x86_64-fslsdk-linux/usr/bin/aarch64-fsl-linux:/opt/fsl-qoriq/2.4.1/sysroots/x86_64-fslsdk-linux/usr/bin/aarch64-fsl-linux-musl:$PATH
source ~/.bashrc
</code>

#### 5. 编译内核  
+ x64_86  
<code>cd /opt/fsl-kernel/x86/linux-4.9.140 && make -j</code>
+ arm64  
<code>cd /opt/fsl-kernel/arm64/linux-4.9.140 && unset LDFLAS</code>		
修改 Makefile 文件第 257、258 两行  
<code>ARCH ?= $(SUBARCH)
CROSS_COMPILE ?= $(CONFIG_CROSS_COMPILE:"%"=%)</code>  
为  
<code>ARCH ?= arm64
CROSS_COMPILE ?= aarch64-fsl-linux-</code>  
然后运行 make 命令进行编译

#### 6. 构建系统
+ 构建  
<code>cd secogateway  
make OPT=clean && make</code>
</code>    
+ 清理  
<code>make OPT=clean</code>
+ 安装  
<code>make OPT=install [DIR=../_install]</code>  
DIR 环境变量指定了最终安装默认，默认为 ../_install。支持绝对路径以及相对路径。
+ 安装目录结构  
<pre>
_install
└── debug               系统构建类型 debug/release
    ├── targets         构建可执行程序、驱动安装目录
    │   ├── ARM64       ARM64 平台的可执行程序、驱动
    │   └── LINUX       Linux 平台的可执行程序、驱动
    └── targets.debug   构建可执行程序、驱动的对应调试符号信息安装目录
        ├── ARM64       ARM64 平台的可执行程序、驱动调试符号信息
        └── LINUX       ARM64 平台的可执行程序、驱动调试符号信息
</pre>
+ 编译信息  
执行 make 命令进行编译结束后，./Common/compile.h 文件中保存了当前系统编译信息，可以在代码中直接引用。  
<code>#define sGATE_COMPILE_DATE "2019-05-22"
#define sGATE_COMPILE_TIME "11:19:58"
#define sGATE_COMPILE_MAJOR "20190522"
#define sGATE_COMPILE_SUB "111958"
#define sGATE_COMPILE_BY "hx"
#define sGATE_COMPILE_HOST "hx-ubuntu"
#define sGATE_GIT_TAGS "sGATE1.0-20181213-stable-159-g1b4c69da2-dev"
#define sGATE_GIT_VERS "1b4c69da2d9cd6d133075f2fd96fbe0ac220fb72"
</code>  
sGATE_GIT_TAGS 记录了当前源码在 gitlab 服务器上面的分支信息  
sGATE_GIT_VERS 记录了当前源码在 gitlab 服务器上面的版本信息  

#### 7. Windows 开发环境搭建

+ 方法一：下载克隆（或屏蔽）部分文件代码（**适用于不在windows下查看、编辑linux内核代码的开发人员**）
1. 说明
     **windows操作系统git下载克隆（或屏蔽）部分文件代码的方法** 
    使用以下代码的重点是，不能使用POWERSHELL/CMD，只能使用git bash，如果在POWERSHELL/CMD中操作，在最后一步命令的时候会报以下错误：
        <code>error: Sparse checkout leaves no entry on working directory</code>
2. 准备工作：启用git的sparse checkout功能（要求Cit 1.7+版本）
<code>mkdir secogateway
cd secogateway
git init
git remote add -f http://git.komect.net/ISG/secogateway.git
git config core.sparsecheckout true</code>
3. 设置检出的过滤规则：示例屏蔽kernel文件夹
<code>echo '/*' >> .git/info/sparse-checkout #这一步不可省略
echo '!/kernel/' >> .git/info/sparse-checkout</code>
4. 命令行 执行检出：（或使用TortoiseGit等图形化工具，执行检出）
<code>git checkout master</code>
------------
+ 方法二：解决 git 出错的
1. 下载并安装 [cygwin](http://cygwin.com/install.html)  
2. 运行安装程序 ![安装界面](./docs/img/1.PNG)
3. 在源选择界面中按图添加163源 [http://mirrors.163.com/cygwin/](http://mirrors.163.com/cygwin/) ![源设置界面](./docs/img/6.PNG)
4. 安装包选择界面中选择中 git 等必要命令(如果没有选择上，以后可以运行安装工具继续选择) ![选择软件包](./docs/img/7.PNG)
5. 点击 下一步 等待安装完成 ![安装完成](./docs/img/10.PNG)
6. 运行 cygwin，执行以下代码（假设 Windows 目录为 E:\my_projects）拉取服务器代码
<code>cd /cygdrive/e/my_projects
git clone [git@git.komect.net:ISG/secogateway.git](http://git.komect.net/ISG/secogateway)</code>
7. 如果 git clone 报错，请设置 ssh key
8. git clone 完成后，可以从 E:\my_projects\secogateway 访问文件。警告文件为内核源码，可以忽略 ![](./docs/img/A1.PNG)
9. 可以将 kernel/linux-4.14.83/ 添加到本地 .gitignore 文件中，但不要将 .gitignore 文件提交到服务器
------------
+ 方法三：将服务器目录映射到 Windows 驱动器
1. 安装 [sshfs](https://github.com/billziss-gh/sshfs-win)
2. 安装 [WinFsp](http://www.secfs.net/winfsp/)
3. 参照 [WinFsp文档](http://www.secfs.net/winfsp/) 进行配置
