---
layout: post
category: 硬件
keywords:
tags: hardware
title: HP-2560P黑苹果折腾记
description:
---


> 导读：最近换了新笔记本（HP Envy x360），旧笔记本便被淘汰下来了。之前在学校时一直是台式机+笔记本的搭配，对性能的要求也不是很高，所以一直都是装着deepin一直在当开发机用。现在沦为二奶机了，而且感觉最近升级的deepin15.6后也不太合胃口，索性便在坛友那买了块SSD，折腾折腾黑苹果，体验了一把睾贵的Mac OS。话不多说，下面是大致的安装过程...

```
· 硬件配置：HP-2560P( CPU：i7-2620m 显卡：HD3000集显 网卡：Intel 82579LM(有线) + BCM943224HMS(无线) )
· 软件配置：macOS 10.13.5 ， clover 4558
· 引导方案：UEFI + GPT + Clover + macOS单系统
```

- 前言
- 安装步骤
  - 1.制作启动盘
    - 获取镜像
    - 制作启动U盘
    - 配置plist
  - 2.安装
  - 3.固化Clover
  - 4.升级Clover(可选)
  - 5.安装驱动
    - 有线网卡
    - 电源管理
    - 声卡
    - 无线网卡
    - 触摸板
  - 6.洗白(注入三码)
  - 7.其他优化
    - 解决每次开机时都是最大亮度
    - 设置开机直接进入系统
    - 修改终端中显示的计算机名
    - 修改按键映射
    - 修复无法睡眠的问题
- 结语
- 相关资源下载
  - 工具包
  - EFI分区备份
- 参考网站


## 安装步骤
安装黑苹果的步骤其实和你平常装Windows差不了多少，如果你熟悉Windows的安装的话，安装黑苹果的问题应该也不会很大。总结下来不外乎下面几步：制作启动盘、U盘启动进行安装、安装驱动、po解/洗白。

### 制作启动盘
有两种方案可以制作启动盘：1.使用原版镜像(门槛高)；2.使用别人制作好的镜像(自带clover,门槛低)；为了使安装过程简单一点，我选择了第2种方案，优点是所有操作可以在Windows下完成，无需Mac虚拟机或者白苹果，对新手而言可能会简单一点。

#### 获取镜像
这里推荐三个网站：
- [远景论坛](http://bbs.pcbeta.com/)
- [黑苹果社区](https://osx.cx/)
- [黑苹果乐园](https://imac.hk/)

可以在这三个网站中寻找名为 ` macOS ... With Clover` 的镜像(三个网站中镜像资源应该是同一个人发布的，md5值都一样...)，不过最恼人的是三个网站都需要注册用户才能下载，这里可以求助于万能的某宝，搜索" 网站名 + 代下载 "，几毛钱即可搞定。

#### 制作启动U盘
你需要准备的是：一个 ≥8G 的U盘，系统镜像、TransMac与DiskGenius(可以在本文的后面的工具包中找到)。

![所需工具]({{ '/resources' | append: page.id }}/images/所需工具.png)

- 首先，将U盘格式化，并用DiskGenius将其转换成GPT格式：
  
  ![将U盘转换成GPT模式]({{ '/resources' | append: page.id }}/images/将U盘转换成GPT模式.png)

- 使用管理员身份运行TransMac，在U盘右键菜单中选择 `Format Disk for Mac` ，将U盘格式化成Mac分区格式，后面的卷名可以随便填：
  
  ![格式化成Mac分区格式]({{ '/resources' | append: page.id }}/images/格式化成Mac分区格式.png)

- 格式化完成后，再次在U盘右键菜单中选择 `Restore with Disk Image` 项，将下载好的系统镜像写入到U盘中：
  
  ![将镜像写入到U盘中]({{ '/resources' | append: page.id }}/images/将镜像写入到U盘中.png)
  
  ![镜像写入中]({{ '/resources' | append: page.id }}/images/镜像写入中.png)

- 写入完成！
  
  ![镜像写入完成]({{ '/resources' | append: page.id }}/images/镜像写入完成.png)

#### 配置plist

Clover依靠EFI分区中的 `config.plist` 文件来配置macOS启动所需的参数，该文件位于： `U盘的EFI分区/EFI/CLOVER/` 文件夹下。其中的配置通常与你的电脑硬件配置有关，需要自行调整。所以前面的启动盘如果不进一步配置的话，后面虽然也能够进入Clover，但是引导安装时会出现这种情况：

![无法引导安装]({{ '/resources' | append: page.id }}/images/无法引导安装.jpg)

无法进入安装界面，自然也无法继续安装。

不过，像这种非原版的系统镜像通常会自带一些常用的config.plist文件，来适应一些常见的硬件。如果有适用与你硬件的配置文件的话可以拿来试试(重命名并替换掉原有 config.plist 文件)：

![常见硬件的plist配置文件]({{ '/resources' | append: page.id }}/images/常见硬件的plist配置文件.png)

不幸的是：我的电脑显卡为 hd3000，并不是默认支持。不过这也没关系，你可以到 [远景论坛](http://bbs.pcbeta.com/) 上面寻找与你机型相同的用户，看是否有人分享自己的 `config.plist` 文件，或者 `EFI分区打包` :

![上远景论坛寻找配置文件]({{ '/resources' | append: page.id }}/images/上远景论坛寻找配置文件.png)

我运气不错，找到一个直接给出度盘链接的用户。如果是需要登录才能下载的附件的话，依旧可以求助于万能的某宝，这里不展开细说。

提取其中的 `config.plist` 文件替换到U盘中，准备工作便做好了！


### 安装

HP-2560P对UEFI的支持比较奇怪，只支持 OS manager、光驱位UEFI以及以太网UEFI，所以选启动项时与其他教程有些不同，其他基本一样。

- 开机按F10，进入BIOS，找到并开启UEFI启动选项，保存：
  
  ![开启UEFI启动选项]({{ '/resources' | append: page.id }}/images/开启UEFI启动选项.jpg)

- 插入先前制作好的启动U盘，在电脑启动时按F9，进入 `Boot Options` 界面，选择 `Boot from EFI file` 项：
  
  ![选择启动选项]({{ '/resources' | append: page.id }}/images/选择启动选项.jpg)

- 依次选择进入  `"U盘/EFI/CLOVER"` 路径，找到 `CLOVERX64.efi` 文件，回车：
  
  ![找到CLOVERX64文件]({{ '/resources' | append: page.id }}/images/找到CLOVERX64文件.jpg)

- 此时将会进入Clover界面：
  
  ![进入到Clover界面]({{ '/resources' | append: page.id }}/images/进入到Clover界面.jpg)

- 选择上图中的 `Boot macOS Install from Install macOS High Sierra` 项，回车，接下来是一大堆命令行文本输出：
  
  ![启动盘的命令行输出]({{ '/resources' | append: page.id }}/images/启动盘的命令行输出.jpg)

- 其中的错误可以无视，因为有一些硬件还无法驱动，所以自然是一堆Error。稍等一会，便可进入启动盘的语言选择界面：
  
  ![启动盘的语言选择界面]({{ '/resources' | append: page.id }}/images/启动盘的语言选择界面.jpg)

- 选择简体中文。接下来，进入 "macOS实用工具" 中的 `磁盘工具` ，选择你的电脑硬盘，单击 `“抹掉”` ，用它来将硬盘格式化成Mac专用格式，名称可以随便填：
  
  ![macOS实用工具上的磁盘工具]({{ '/resources' | append: page.id }}/images/macOS实用工具上的磁盘工具.jpg)
  
  ![使用磁盘工具抹掉硬盘]({{ '/resources' | append: page.id }}/images/使用磁盘工具抹掉硬盘.jpg)

- 格式化完成后，退出磁盘工具，在 "macOS实用工具" 菜单中选择 `安装macOS` :
  
  ![macOS实用工具上的安装macOS]({{ '/resources' | append: page.id }}/images/macOS实用工具上的安装macOS.jpg)

- 许可协议之类的，看着选就行：
  
  ![安装界面的许可协议]({{ '/resources' | append: page.id }}/images/安装界面的许可协议.jpg)

- 等看到这个界面时，选择你先前使用 `磁盘工具` 抹掉后得到的盘，点安装即可：
  
  ![安装界面选择磁盘]({{ '/resources' | append: page.id }}/images/安装界面选择磁盘.jpg)

- 安装过程中会重启几次，不过由于HP-2560P对UEFI的支持不完全，所以重启时请自行按F9，选择 `Boot from EFI file`，重新使用 `CLOVERX64.efi` 来启动CLOVER，并使用 `Boot from <前面抹掉硬盘时设置的硬盘名>` 选项来继续安装。

- 接下来的步骤不一一讲述啦，无非是 等待安装、创建用户、选择区域语言、选择键盘布局之类的，自己看着选就行，由于我已经装好了所以就不重装演示了。


### 固化Clover

系统安装完成后，若要从硬盘中启动macOS，还需要使用U盘EFI分区中的Clover来进行引导，是不是有些麻烦？别担心，接下来我们将Clover转移到硬盘的EFI分区中去，后面便可脱离U盘启动。


- 在本文下方提供的工具包中找到 `Clover Configurator` ，将其拷贝到你的新系统中，打开它：
  
  ![打开CloverConfigurator]({{ '/resources' | append: page.id }}/images/打开CloverConfigurator.png)

- 点击左侧侧边栏中的 `Mount EFI` 项：
  
  ![侧边栏中的MountEFI项]({{ '/resources' | append: page.id }}/images/侧边栏中的MountEFI项.png)

- 在界面下方可以看见 `"EFI on EFI"` 这个分区，它位于你的电脑中。点击 `Mount Partition` 挂载该分区，然后点击 `Open Partition` 打开该分区：
  
  ![新系统的EFI分区]({{ '/resources' | append: page.id }}/images/新系统的EFI分区.png)

- 删除该分区中的 `EFI` 文件夹，使用U盘EFI分区中的 `EFI` 文件夹替换之：
  
  ![使用U盘EFI分区中的文件夹来替换]({{ '/resources' | append: page.id }}/images/使用U盘EFI分区中的文件夹来替换.png)

- 此后，便可脱离U盘，使用硬盘EFI分区中的Clover来引导系统了。


### 升级Clover(可选)

Clover的版本更新很快，系统镜像自带的Clover版本往往不高。有些kext驱动对Clover的版本有一定的要求，这时候便需要手动升级Clover版本。

- 到 [Clover EFI bootloader项目](https://sourceforge.net/projects/cloverefiboot/files/latest/download) 中下载最新版本的Clover，然后打开下载后得到的 `.pkg` 文件：
  
  ![打开Clover安装包]({{ '/resources' | append: page.id }}/images/打开Clover安装包.png)

- 继续，继续，继续，等到出现下面这一步时选择 `自定` ：
  
  ![Clover安装时选择自定]({{ '/resources' | append: page.id }}/images/Clover安装时选择自定.png)

- 勾选 `“安装Clover到EFI系统区”` 和 `“Drivers64UEFI”` 项，点击 安装 ：
  
  ![Clover安装时勾选必要选项]({{ '/resources' | append: page.id }}/images/Clover安装时勾选必要选项.png)

- 输入你的用户密码，之后便可升级完成：
  
  ![Clover安装完成]({{ '/resources' | append: page.id }}/images/Clover安装完成.png)

- 以后升级时依照上述步骤安装即可。


### 安装驱动

系统已经安装完成，接下来是驱动安装。经过上面的折腾以后，系统已经是基本可用的了，但是你会发现有很多硬件功能尚不可用，比如电池无法显示，亮度无法调节，声卡和无线网卡没有驱动等，下面我们将一一解决。

#### 有线网卡

我的有线网卡默认就是驱动了的，如果没有，可以试试使用 `MultiBeast` 来安装驱动，适用于macOS 10.13.5 的 Multibeast我已放到了文后的工具包中。

- 打开 MultiBeast ，点击 `Drivers → Network` ，勾选适用与你电脑硬件的驱动，点击 Build ：
  
  ![使用MultiBeast安装网卡驱动步骤1]({{ '/resources' | append: page.id }}/images/使用MultiBeast安装网卡驱动步骤1.png)

- 接着在右方选择你的硬盘名，点击 Install 即可，kext驱动将会被安装到 `/资源库/Extensions/` 文件夹下：
  
  ![使用MultiBeast安装网卡驱动步骤2]({{ '/resources' | append: page.id }}/images/使用MultiBeast安装网卡驱动步骤2.png)


#### 电源管理

默认系统安装完成后无法显示电池电量，且亮度调节也不正常。修复电源的常用方案是对DSDT文件打补丁，不过我目前还没有去仔细去研究过这块。现在我使用的是远景论坛同机型用户分享的 `DSDT.aml` 文件来修复这两个问题，不够完美但是基本够用。现在已经发现的问题有：
  1. 整体亮度过高了一点。
  2. 最低亮度时黑屏。
  3. 无法睡眠。（已解决，见: [修复无法睡眠的问题](#修复无法睡眠的问题)）

等有时间再去仔细研究了。

目前我的做法是：
  - 将打了补丁的 `DSDT.aml` 文件复制到 `EFI分区/EFI/CLOVER/ACPI/patched/` 目录下，其中EFI分区可使用上文中提到的 `Clover Configurator` 来挂载打开。
  - 重启电脑即可：
  
    ![电源管理已经能够使用了]({{ '/resources' | append: page.id }}/images/电源管理已经能够使用了.png)

如果你想进一步监控CPU温度、频率、功耗等信息的话，则需要对 `EFI分区/EFI/CLOVER/kexts/Other/` 目录下的 `FakeSMC.kext` 进行改动，并使用 `HWMonter` 软件来进行监控，下面是操作方法：
  - 到 [ OS-X-FakeSMC-kozlek项目](https://bitbucket.org/RehabMan/os-x-fakesmc-kozlek/downloads/) 中下载最新版本的 RehabMan-FakeSMC.zip 。
  - 解压，在目录下新建一个名为 `Plugins` 的文件夹，并将 `FakeSMC_ACPISensors.kext`，`FakeSMC_CPUSensors.kext`，`FakeSMC_GPUSensors.kext`，`FakeSMC_LPCSensors.kext` 四个文件移动到其中。
  - 重命名 `FakeSMC.kext` 文件为 `FakeSMC.kext.bak` ，在Mac下将把其显示为文件夹。然后将上面的 `Plugins` 文件夹移动到 `FakeSMC.kext.bak/Contents/` 目录下:
  
    ![处理FakeSMC]({{ '/resources' | append: page.id }}/images/处理FakeSMC.png)

  - 回到上级，然后将 `FakeSMC.kext.bak` 文件夹命名回 `FakeSMC.kext` 。
  - 将 `FakeSMC.kext` 文件复制到 `EFI分区/EFI/CLOVER/kexts/Other/` 目录下，然后重启电脑。
  - 打开压缩包中自带的 `HWMonitor.app` ，即可在状态栏中看见应用图标，可监控CPU温度、频率等信息：
  
    ![使用HWMonitor监控硬件信息]({{ '/resources' | append: page.id }}/images/使用HWMonitor监控硬件信息.png)


#### 声卡
由于没有事先记录声卡的具体型号，也没找到相关的资料，所以我决定先用 [VoodooHDA](https://sourceforge.net/projects/voodoohda/files/latest/download) 来试试看，没想到居然成功驱动了！外放正常，耳机正常，麦克风正常，睡眠后声音失效的问题可以通过安装 `CodecCommander.kext` 来解决，基本完美。

声卡驱动的安装方法为：
  - 到 [VoodooHDA项目](https://sourceforge.net/projects/voodoohda/files/latest/download) 中下载最新版本的VoodooHDA。
  - 将解压得到的 `VoodooHDA.kext` 复制到 `EFI分区/EFI/CLOVER/kexts/Other/` 目录下。
  - 到 [OS-X-EAPD-Codec-Commander项目](https://bitbucket.org/RehabMan/os-x-eapd-codec-commander/downloads/) 中下载最新版本的 RehabMan-CodecCommander.zip 。
  - 解压，将 Release文件夹中的 `CodecCommander.kext` 复制到 `EFI分区/EFI/CLOVER/kexts/Other/` 目录下。
  - 重启电脑后即可驱动声卡并解决睡眠声音问题：
    
    ![声卡已经正常驱动了]({{ '/resources' | append: page.id }}/images/声卡已经正常驱动了.png)


#### 无线网卡

> 无线网卡驱动的安装可参考：[wireless_broadcom](https://github.com/toleda/wireless_broadcom)

黑苹果对Intel无线网卡的支持并不好，建议还是换BCM网卡保平安。网上看见有几款网卡是可以直接免驱的，不过由于惠普本坑比的网卡白名单限制，可供选择的网卡型号并不多。所以我选择了惠普拆机的 `BCM943224HMS` 网卡，淘宝上十几块搞定。

无线网卡更换完成后，还需额外安装kext才能正常使用，下面是安装方法：
  - 到 [ OS-X-Fake-PCI-ID项目](https://bitbucket.org/RehabMan/os-x-fake-pci-id/downloads/) 中下载最新版本的 RehabMan-FakePCIID.zip 。
  - 解压，将 Release 文件夹中的 `FakePCIID.kext` 和 `FakePCIID_Broadcom_WiFi.kext` 复制到 `EFI分区/EFI/CLOVER/kexts/Other/` 目录下。
  - 重启电脑即可：
    
    ![无线网卡已经正常驱动了]({{ '/resources' | append: page.id }}/images/无线网卡已经正常驱动了.png)


#### 触摸板

系统装好以后在 `系统偏好设置 → 触摸板` 设置中无显示内容，估计是被当成鼠标了。下面我们来修复它：
  - 到 [OS-X-Voodoo-PS2-Controller项目](https://bitbucket.org/RehabMan/os-x-voodoo-ps2-controller/downloads/) 中下载最新版本的 RehabMan-Voodoo.zip 。
  - 解压，将 `org.rehabman.voodoo.driver.Daemon.plist` 文件复制到本机 `/资源库/LaunchDaemons` 路径下。
  - 将 Release 文件夹中的 `VoodooPS2Daemon` 复制到本机的 `/usr/bin/` 路径下。
  - 将 Release 文件夹中的 `VoodooPS2Controller.kext` 文件复制到 `EFI分区/EFI/CLOVER/kexts/Other/` 目录下。
  - 重启电脑即可。
    
    ![触摸板已经正常驱动了]({{ '/resources' | append: page.id }}/images/触摸板已经正常驱动了.png)


### 洗白(注入三码)

> 先决条件：确保你的有线网卡已经内建，并且BSD名称为en0 （在：左上角苹果标→关于本机→系统报告→网络 处可以看到）。

系统安装完成后大部分功能已经可用，但iMessage和FaceTime依旧无效。如果你需要这两项功能的话可以通过注入 `SerialNumber`，`SmUUID`，`Board Serial Number` 来修复。可以从白苹果中获取这三个码，或者使用工具来生成。由于我手头没有白苹果，所以使用的是工具生成的方法：
  - 首先，确保你没有在系统中登录iCloud (如果事先登录了的话请先在系统中退出，然后在iCloud中删除这个设备)。
  - 打开终端，执行命令：`cd ~/Library/Caches/`
  - 执行命令：`rm -R com.apple.iCloudHelper*`
  - 执行命令：`rm -R com.apple.imfoundation.IMRemoteURLConnectionAgent*`
  - 执行命令：`rm -R com.apple.Message*`
  - 执行切换路径的命令：`cd ~/Library/Preferences/`
  - 执行命令：`rm -R com.apple.iChat.*`
  - 执行命令：`rm -R com.apple.icloud.*`
  - 执行命令：`rm -R com.apple.ids.service*`
  - 执行命令：`rm -R com.apple.imagent.*`
  - 执行命令：`rm -R com.apple.imessage.*`
  - 执行命令：`rm -R com.apple.imservice.*`
  - 然后重启你的系统。
  - 重启后，使用 `Clover Configurator` 挂载EFI分区，然后用它打开 `EFI分区/EFI/CLOVER/` 路径下的 `config.plist` ，点击左侧的 `SMBIOS` 项:
    
    ![打开Clover的SMBIOS项]({{ '/resources' | append: page.id }}/images/打开Clover的SMBIOS项.png)
  
  - 点击上图中Macbook图片旁边的下拉图标，选择你的机型，将自动为你生成一个序列号：
    
    ![Clover自动生成序列号]({{ '/resources' | append: page.id }}/images/Clover自动生成序列号.png)

  - 复制上图中生成的序列号，进入 [EveryMac官网](https://everymac.com/ultimate-mac-lookup/) ，粘贴你的序列号并进行人机身份验证：
    
    ![上EveryMac检验生成的序列号]({{ '/resources' | append: page.id }}/images/上EveryMac检验生成的序列号.png)
  
  - 如果可以查询到相关的设备信息,则进行下一步，否则回到前两步中重新生成序列号：
    
    ![该序列号在EveryMac上检验成功]({{ '/resources' | append: page.id }}/images/该序列号在EveryMac上检验成功.png)
  
  - 然后上 [苹果保修查询](https://checkcoverage.apple.com) 这个网站，依旧填入上面的序列号，查询保修状态：
    
    ![到Apple官网查询保修状态]({{ '/resources' | append: page.id }}/images/到Apple官网查询保修状态.png)

  - 如果查询结果如下图所示，显示 "这个序列号无效"，那么恭喜你！可以将这个序列号用在你的黑苹果中。否则回到上面 用Clover自动生成序列号的这步。
    
    ![Apple官网显示该序列号无效]({{ '/resources' | append: page.id }}/images/Apple官网显示该序列号无效.png)

  - 在系统中打开终端，执行以下命令以随机生成UUID: `uuidgen`
  - 将生成的UUID复制，然后填入到之前 `Clover Configurator → SMBIOS` 界面上的 `SmUUID` 输入框中：
    
    ![填入随机生成的UUID]({{ '/resources' | append: page.id }}/images/填入随机生成的UUID.png)

  - 点击在菜单栏中的 `File → Save` ，然后重启系统。

  - 将本文后面提供的资源包中提供的的 `iMessageDebugv2` 复制到你的系统中，然后在 `iMessageDebug` 文件上 `右键→打开方式→终端` ，按 y 来在当前目录下创建一个文本输出副本(iMessageDebug.txt)。
  
  - 再次重启系统，并用右键打开 `iMessageDebug` 文件，比较这次的输出与iMessageDebug.txt文件中的内容是否相同，如果完全相同，则黑苹果洗白成功！你应该可以正常使用iMessage了。
    
    ![iMessage已经可以正常使用了]({{ '/resources' | append: page.id }}/images/iMessage已经可以正常使用了.png)


### 其他优化

#### 解决每次开机时都是最大亮度

删除 `EFI分区/EFI/CLOVER/drivers64UEFI/` 文件夹中的 ` EmuVariableUefi-64.efi` 文件，重启即可(该文件对集显没什么用处)。

#### 设置开机直接进入系统
开机后引导进入Clover界面，然后还要控制箭头选择从硬盘启动，是不是有些麻烦？下面我们使用 `Clover Configurator` 来设置Clover的超时时间与默认选项，以解决这个问题：
  - 挂载EFI分区，进入到 `EFI分区/EFI/CLOVER/` 路径下，然后备份 `config.plist` 为 `config.plist.bak` 。
  - 用 `Clover Configurator` 打开 `config.plist` ，然后点击左侧的 `Boot` 项。
  - 如下图所示，分别设置右侧的 `Default Boot Volume` 为 `LastBootedVolume` ，设置 `Timeout` 为 `0` ：

    ![设置开机直接进入系统]({{ '/resources' | append: page.id }}/images/设置开机直接进入系统.png)

  - 点击在菜单栏中的 `File → Save` ，重启即可看见效果。
  - 注：若开机后依旧停留在Clover界面中，没有进行倒计时。请尝试删除 `EFI分区/EFI/CLOVER/drivers64UEFI/` 文件夹中的 ` EmuVariableUefi-64.efi` 文件。
  - 注：如此做后，若在下次开机时要进入Clover，请在看见开机图标后按住方向键。

#### 修改终端中显示的计算机名

在终端中使用命令：`sudo scutil --set HostName 新的名字` 来修改它。

#### 修改按键映射

刚从Windows键盘布局中切换过来可能会感觉不太习惯，比如复制在macOS下就变成了Command + C (即Alt + C)，而 Control键的功能却比较类似Windows系统下的Win键，以至于现在每次按快捷键时都要先回想一下，而不是靠肌肉记忆。还好macOS系统切换按键映射还是挺方便的，如果你不想忍受的话可以切换过来，更加符合Windows下的按键思维。

  ![按键方案]({{ '/resources' | append: page.id }}/images/按键方案.png)

具体做法：
  - 点击 `系统偏好设置→键盘→修饰键...` 。
  - 通过每项的下拉栏设置按键绑定即可:
    
    ![修改按键方案]({{ '/resources' | append: page.id }}/images/修改按键方案.png)

#### 修复无法睡眠的问题
前面在 ["电源管理"](#电源管理) 一节中提到了笔记本无法睡眠的问题：当点击"睡眠"时，电脑循环往复的被唤醒，以至于根本无法进入睡眠。在网上找了一些资料，终于发现了问题所在，给 `DSDT.aml` 打上一个USB补丁即可。现在睡眠功能完成度已经比较高了(美中不足的是每两个小时会被唤醒一次，然后再次进入睡眠。日志看起来与RTC时钟有关，尚未找到解决方法)，日常使用已无大碍。

- 首先，介绍一条命令，你可以用它来寻找电脑被唤醒的原因：

  `log show |grep -i "wake reason" > ~/Desktop/log.txt （将导出到桌面）`

- 打USB补丁的方法：
  - 你需要用到的工具为： `MaciASL` ，你可以从[ OS-X-MaciASL-patchmatic
项目](https://bitbucket.org/RehabMan/os-x-maciasl-patchmatic/downloads/) 中下载最新版本的 RehabMan-MaciASL.zip , 解压获得此app 。
  - 打开此app，点击菜单栏中的 Preferences → Sources ，确保补丁源中已添加：http://raw.github.com/RehabMan/Laptop-DSDT-Patch/master 这个源：
    
    ![确保你已经添加RehabMan的Laptop补丁源]({{ '/resources' | append: page.id }}/images/确保你已经添加RehabMan的Laptop补丁源.png)
  
  - 使用 `Clover Configurator` 挂载EFI分区，将 `EFI分区/EFI/CLOVER/ACPI/patched/` 目录下的 `DSDT.aml` 文件备份为 `DSDT.aml.bak` 。
  - 使用 MaciASL 打开 `DSDT.aml` 文件。
  - 点击 `Patch` 按钮进入补丁选择界面：
    
    ![点击MaciASL中的Patch按钮]({{ '/resources' | append: page.id }}/images/点击MaciASL中的Patch按钮.png)
  
  - 在左侧的侧边栏中找到： `_RehabMan Laptop → [usb]USB3_PRW 0x6D(instant wake)` 这个补丁，双击，等待右侧加载出代码后，点 Applay 提交：
    
    ![选择USB3_PRW_0x6D补丁]({{ '/resources' | append: page.id }}/images/选择USB3_PRW_0x6D补丁.png) 
  
  - 完成后，点 Close 退出补丁界面，使用菜单栏中的 File → Save 保存文件即可，之后重启电脑检验是否成功。


## 结语
黑苹果的安装过程其实并不困难，难的是装驱动和后续的优化。借用Linux的一句话来说，就是：“macOS is free, only if your time is free.” ，要做到完美的话不仅要花费大量时间，而且还需要一些运气。我目前搞完后基本上可以日常使用了，但还有下面一些问题，等有时间再仔细研究了：
  - 最低亮度直接黑屏；
  - 光驱无法驱动；
  - 指点杆无法驱动；
  - VGA接口无法使用；
  - 自带读卡器无法驱动；
  - 其他尚未发现的问题...



## 相关资源下载

### 工具包
点 [这里](https://pan.baidu.com/s/1QLUyb3CMDhOCLSHQ_UMzcQ) 下载

### EFI分区备份
点 [这里](https://pan.baidu.com/s/1aVBqS-G7y3dAVPNGi2s3rA) 下载


## 参考网站
下面是安装过程中参考的一些网站：
  - [tonymacx86](https://www.tonymacx86.com/)
  - [远景论坛](http://bbs.pcbeta.com/)
  - [黑苹果乐园](https://imac.hk/)
  - [黑苹果社区](https://osx.cx/)
  - [RehabMan大神项目主页](https://bitbucket.org/RehabMan/)
