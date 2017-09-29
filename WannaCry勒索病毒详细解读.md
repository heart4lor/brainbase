# WannaCry勒索病毒详细解读


## 一、病毒概况

WannaCry病毒利用前阵子泄漏的方程式工具包中的“永恒之蓝”漏洞工具，进行网络端口扫描攻击，目标机器被成功攻陷后会从攻击机下载WannaCry病毒进行感染，并作为攻击机再次扫描互联网和局域网其他机器，形成蠕虫感染，大范围超快速扩散。

病毒母体为mssecsvc.exe，运行后会扫描随机ip的互联网机器，尝试感染，也会扫描局域网相同网段的机器进行感染传播。此外，会释放敲诈者程序tasksche.exe，对磁盘文件进行加密勒索。

病毒加密使用AES加密文件，并使用非对称加密算法RSA 2048加密随机密钥，每个文件使用一个随机密钥，理论上不可破解。

[![1.png](http://image.3001.net/images/20170519/14951961091000.png!small)](http://image.3001.net/images/20170519/14951961091000.png)

## 二、病毒详细分析

### 1、mssecsvc.exe行为

1）开关

病毒在网络上设置了一个开关，当本地计算机能够成功访问[http://www.iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com](http://www.iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com/)时，退出进程，不再进行传播感染。目前该域名已被安全公司接管。

[![2.png](http://image.3001.net/images/20170519/14951961298782.png!small)](http://image.3001.net/images/20170519/14951961298782.png)

2）蠕虫行为

通过创建服务启动，每次开机都会自启动。

[![3.png](http://image.3001.net/images/20170519/14951961498625.png!small)](http://image.3001.net/images/20170519/14951961498625.png)

从病毒自身读取MS17_010漏洞利用代码，playload分为x86和x64两个版本。

[![4.png](http://image.3001.net/images/20170519/14951961667463.png!small)](http://image.3001.net/images/20170519/14951961667463.png)

创建两个线程，分别扫描内网和外网的IP，开始进程蠕虫传播感染。

[![5.png](http://image.3001.net/images/20170519/14951961787385.png!small)](http://image.3001.net/images/20170519/14951961787385.png)

对公网随机ip地址445端口进行扫描感染。

[![6.png](http://image.3001.net/images/20170519/14951961899825.png!small)](http://image.3001.net/images/20170519/14951961899825.png)

对于局域网，则直接扫描当前计算机所在的网段进行感染。

[![7.png](http://image.3001.net/images/20170519/14951962036599.png!small)](http://image.3001.net/images/20170519/14951962036599.png)

感染过程，尝试连接445端口。

[![8.png](http://image.3001.net/images/20170519/14951962143115.png!small)](http://image.3001.net/images/20170519/14951962143115.png)

如果连接成功，则对该地址尝试进行漏洞攻击感染。

[![9.png](http://image.3001.net/images/20170519/1495196229755.png!small)](http://image.3001.net/images/20170519/1495196229755.png)

3）释放敲诈者

[![10.png](http://image.3001.net/images/20170519/14951962486487.png!small)](http://image.3001.net/images/20170519/14951962486487.png)

### 2、tasksche.exe行为（敲诈者）

解压释放大量敲诈者模块及配置文件，解压密码为WNcry@2ol7

[![11.png](http://image.3001.net/images/20170519/14951962627531.png!small)](http://image.3001.net/images/20170519/14951962627531.png)

首先关闭指定进程，避免某些重要文件因被占用而无法感染。

[![12.png](http://image.3001.net/images/20170519/14951962753811.png!small)](http://image.3001.net/images/20170519/14951962753811.png)

遍历磁盘文件，避开含有以下字符的目录。

```
\ProgramData\Intel\WINDOWS\Program Files\Program Files (x86)\AppData\Local\Temp\Local Settings\Temp
```

This folder protects against ransomware. Modifying it will reduce protection

[![13.png](http://image.3001.net/images/20170519/14951962908365.png!small)](http://image.3001.net/images/20170519/14951962908365.png)

同时，也避免感染病毒释放出来的说明文档。

[![14.png](http://image.3001.net/images/20170519/14951963051857.png!small)](http://image.3001.net/images/20170519/14951963051857.png)

病毒加密流程图：

[![15.png](http://image.3001.net/images/20170519/14951963223674.png!small)](http://image.3001.net/images/20170519/14951963223674.png)

遍历磁盘文件，加密以下178种扩展名文件：

```
.doc, .docx, .xls, .xlsx, .ppt, .pptx, .pst, .ost, .msg, .eml, .vsd, .vsdx, .txt, .csv, .rtf, .123, .wks, .wk1, .pdf, .dwg, .onetoc2, .snt, .jpeg, .jpg, .docb, .docm, .dot, .dotm, .dotx, .xlsm, .xlsb, .xlw, .xlt, .xlm, .xlc, .xltx, .xltm, .pptm, .pot, .pps, .ppsm, .ppsx, .ppam, .potx, .potm, .edb, .hwp, .602, .sxi, .sti, .sldx, .sldm, .sldm, .vdi, .vmdk, .vmx, .gpg, .aes, .ARC, .PAQ, .bz2, .tbk, .bak, .tar, .tgz, .gz, .7z, .rar, .zip, .backup, .iso, .vcd, .bmp, .png, .gif, .raw, .cgm, .tif, .tiff, .nef, .psd, .ai, .svg, .djvu, .m4u, .m3u, .mid, .wma, .flv, .3g2, .mkv, .3gp, .mp4, .mov, .avi, .asf, .mpeg, .vob, .mpg, .wmv, .fla, .swf, .wav, .mp3, .sh, .class, .jar, .java, .rb, .asp, .php, .jsp, .brd, .sch, .dch, .dip, .pl, .vb, .vbs, .ps1, .bat, .cmd, .js, .asm, .h, .pas, .cpp, .c, .cs, .suo, .sln, .ldf, .mdf, .ibd, .myi, .myd, .frm, .odb, .dbf, .db, .mdb, .accdb, .sql, .sqlitedb, .sqlite3, .asc, .lay6, .lay, .mml, .sxm, .otg, .odg, .uop, .std, .sxd, .otp, .odp, .wb2, .slk, .dif, .stc, .sxc, .ots, .ods, .3dm, .max, .3ds, .uot, .stw, .sxw, .ott, .odt, .pem, .p12, .csr, .crt, .key, .pfx, .der
```

程序中内置两个RSA 2048公钥，用于加密，其中一个含有配对的私钥，用于演示能够解密的文件，另一个则是真正的加密用的密钥，程序中没有相配对的私钥。

[![16.png](http://image.3001.net/images/20170519/1495196341978.png!small)](http://image.3001.net/images/20170519/1495196341978.png)

病毒随机生成一个256字节的密钥，并拷贝一份用RSA2048加密，RSA公钥内置于程序中。

[![17.png](http://image.3001.net/images/20170519/14951963543280.png!small)](http://image.3001.net/images/20170519/14951963543280.png)

构造文件头，文件头中包含有标志、密钥大小、RSA加密过的密钥、文件大小等信息。

[![18.png](http://image.3001.net/images/20170519/14951963703249.png!small)](http://image.3001.net/images/20170519/14951963703249.png)

使用CBC模式AES加密文件内容，并将文件内容写入到构造好的文件头后，保存成扩展名为.WNCRY的文件，并用随机数填充原始文件后再删除，防止数据恢复。

[![19.png](http://image.3001.net/images/20170519/14951963846170.png!small)](http://image.3001.net/images/20170519/14951963846170.png)

完成所有文件加密后释放说明文档，弹出勒索界面，需支付价值数百美元不等的比特币到指定的比特币钱包地址，三个比特币钱包地址硬编码于程序中。

> 115p7UMMngoj1pMvkpHijcRdfJNXj6LrLn
>
> 12t9YDPgwueZ9NyMgw519p7AA8isjr6SMw
>
> 13AM4VW2dhxYgXeQepoHkHSQuy6NgaEb94

[![20.png](http://image.3001.net/images/20170519/14951963993125.png!small)](http://image.3001.net/images/20170519/14951963993125.png)

### 3、解密程序

病毒解密程序中内置了其中一个公钥的配对私钥，可以用于解密使用该公钥加密的几个文件，用于向用户“证明”程序能够解密文件，诱导用户支付比特币。

[![21.png](http://image.3001.net/images/20170519/14951964152401.png!small)](http://image.3001.net/images/20170519/14951964152401.png)

[![22.png](http://image.3001.net/images/20170519/14951964288189.png!small)](http://image.3001.net/images/20170519/14951964288189.png)

此后，程序判断本地是否存在“00000000.dky”文件，该文件为真实解密所需私钥文件。若存在，则通过解密测试文件来检测密钥文件是否正确。

[![23.png](http://image.3001.net/images/20170519/14951964428736.png!small)](http://image.3001.net/images/20170519/14951964428736.png)

若正确，则解密。若错误或不存在，病毒将程判断解压后的Tor目录下是否存在taskhsvc.exe。若不存在，则生成该文件，并且调用CreateProcessA拉起该进程：

[![24.png](http://image.3001.net/images/20170519/14951964551367.png!small)](http://image.3001.net/images/20170519/14951964551367.png)

该程序主要为tor匿名代理工具，该工具启动后会监听本地9050端口，病毒通过本地代理通信实现与服务器连接。

在点击“Check Payment”按钮后，由服务端判断是否下发解密所需私钥。若私钥下发，则会在本地生成解密所需要的dky文件。

[![25.png](http://image.3001.net/images/20170519/14951964741337.png!small)](http://image.3001.net/images/20170519/14951964741337.png)

而后，程序便可利用该dky文件进行解密。不过，到目前为止，未曾有解密成功的案例。

### 4、文件列表及作用

> b.wnry: 中招敲诈者后桌面壁纸
>
> c.wnry: 配置文件，包含洋葱域名、比特币地址、tor下载地址等
>
> r.wnry: 提示文件，包含中招提示信息
>
> s.wnry: zip文件，包含Tor客户端
>
> t.wnry: 解密后得到加密程序所需的dll文件
>
> u.wnry: 解密程序（敲诈者主界面）
>
> f.wnry: 可免支付解密的文件列表
>
> WNCYRT: 加密过程中生成的，有的被随机擦写过、有的是原文件、有的只对文件头进行了处理
>
> WNCRY: 加密后的文件

## 三、Wanacry加解密过程深入分析

勒索软件一个核心重点是其加解密过程，接下来我们对加解密和擦写逻辑进行详细分析。

### 1、文件加密

> 1）文件是使用AES进行加密的。 
>
> 2）加密文件所使用的AES密钥（AESKEY）是随机生成的，即每个文件所使用的密钥都是不同的。 
>
> 3）AESKEY通过RSA加密后，保存在加密后文件的文件头中。 
>
> 4）敲诈者会根据文件类型、大小、路径等来判断文件对用户的重要性，进而选择对重要文件的先行进行加密并擦写原文件，对认为不太重要的文件，仅作删除处理。

### 2、文件删除及擦写逻辑

> 1）对于桌面、文档、所有人\桌面、所有人\文档四个文件夹下小于200M的文件，均进行加密后擦写原文件。 
>
> 2）对于其他目录下小于200M的文件，不会进行擦写，而是直接删除，或者移动到back目录（C盘下的“%TEMP%”文件夹，以及其他盘符根目录下的“$RECYCLE”文件夹）中。 
>
> 3）移动到back后，的文件重命名为%d.WNCRYT，加密程序每30秒调用taskdl.exe对back目录下的这些文件定时删除。 
>
> 4）对于大于200M的文件，在原文件的基础上将文件头部64KB移动到尾部，并生成加密文件头，保存为“WNCRYT”文件。而后，创建“WNCRY”文件，将文件头写入，进而按照原文件大小，对该“WNCRY”文件进行填充。

### 3、文件擦写方案

> 1）先重写尾部1k。 
>
> 2）判断大小后重写尾部4k。 
>
> 3）从文件头开始每256k填充一次。 
>
> 4）填写的内容为随机数或0×55。

### 4、详细加密流程

总结密钥及加密关系大致如下：

[![26.png](http://image.3001.net/images/20170519/14951965492643.png!small)](http://image.3001.net/images/20170519/14951965492643.png)

1）程序加密过程中，动态加载的dll里含有两个公钥（KEYBLOB格式），这里分别记为PK1和PK2。

其中PK2用于加密部分文件的AESKEY，并将该部分加密文件的路径存放在f.wnry中，这些文件是可以直接解密的（DK2也存在与可执行文件中）。

而PK1用于加密DK3（见后文介绍）。 

[![27.png](http://image.3001.net/images/20170519/1495196567164.png!small)](http://image.3001.net/images/20170519/1495196567164.png)

2）解密程序u.wnry（即界面程序@WanaDecryptor@.exe）中包含有一个私钥（KEYBLOB格式），该私钥与PK2配对，记为DK2。该DK2用于解密f.wnry中记录的文件。 

[![28.png](http://image.3001.net/images/20170519/14951965808532.png!small)](http://image.3001.net/images/20170519/14951965808532.png)

3）程序在每次运行时，会随机生成一组公私钥，用于。记为PK3、DK3。其中PK3保存在本地，主要用于加密各文件加密时随机生成的AESKEY。而DK3则由PK1加密后保存在本地，文件名为00000000.eky。

### 5、解密过程

1）f.wnry中记录的文件，主要使用DK2完成解密。 

2）其余文件若需要解密，则需点击check payment后，通过Tor将本地00000000.eky、00000000.res文件信息上传到服务端，由服务端使用作者自己保存的DK1解密后，下发得到DK3，本地保存为00000000.dky。 

3）得到DK3后，即可完成对磁盘中其余文件的解密。

### 6、分析及调试验证

将tasksche.exe拖到ida里可以发现，其加解密用到的是系统自带API，通过GetProcAddress来获取地址动态调用的。

[![29.png](http://image.3001.net/images/20170519/14951966001236.png!small)](http://image.3001.net/images/20170519/14951966001236.png)

于是，就可以以此为突破口来进行分析。

1）密钥分发过程

a. 导入密钥DK2，存放于00154830

[![30.png](http://image.3001.net/images/20170519/14951966177067.png!small)](http://image.3001.net/images/20170519/14951966177067.png)

b. 解密t.wnry

[![31.png](http://image.3001.net/images/20170519/14951966302510.png!small)](http://image.3001.net/images/20170519/14951966302510.png)

解密得到的t.wnry实则为一个DLL文件，该文件负责文件加密操作。

c. DLL线程导入PK1，存放于0016BE18

[![32.png](http://image.3001.net/images/20170519/14951966493651.png!small)](http://image.3001.net/images/20170519/14951966493651.png)

d. 生成PK3、DK3

[![33.png](http://image.3001.net/images/20170519/14951966631740.png!small)](http://image.3001.net/images/20170519/14951966631740.png)

e. 导出PK3

[![34.png](http://image.3001.net/images/20170519/14951966744553.png!small)](http://image.3001.net/images/20170519/14951966744553.png)

导出后，会调用writefile在本地生成00000000.pky。

[![35.png](http://image.3001.net/images/20170519/14951966895384.png!small)](http://image.3001.net/images/20170519/14951966895384.png)

f. 使用PK1加密DK3

[![36.png](http://image.3001.net/images/20170519/14951967021693.png!small)](http://image.3001.net/images/20170519/14951967021693.png)

由上图可以匹配PK3和DK3。加密后，会调用writefile在本地生成00000000.eky。 

至此，验证了密钥生成关系。

2）文件加密

a. 导入PK3，放到0016D950

[![37.png](http://image.3001.net/images/20170519/14951967198340.png!small)](http://image.3001.net/images/20170519/14951967198340.png)

b. 导入PK2，放到0016E510

[![38.png](http://image.3001.net/images/20170519/14951967363146.png!small)](http://image.3001.net/images/20170519/14951967363146.png)

c. 生成AESKEY，并使用PK3加密

生成随机AESKEY：

[![39.png](http://image.3001.net/images/20170519/14951967538411.png!small)](http://image.3001.net/images/20170519/14951967538411.png)

使用PK3加密：

[![40.png](http://image.3001.net/images/20170519/14951967699640.png!small)](http://image.3001.net/images/20170519/14951967699640.png)

d. 使用PK2加密

[![41.png](http://image.3001.net/images/20170519/14951967899621.png!small)](http://image.3001.net/images/20170519/14951967899621.png)

使用PK2加密后的文件，会写入f.wnry中：

[![42.png](http://image.3001.net/images/20170519/14951968089748.png!small)](http://image.3001.net/images/20170519/14951968089748.png)

选择使用PK2还是PK3，是随机的：

[![43.png](http://image.3001.net/images/20170519/14951968227866.png!small)](http://image.3001.net/images/20170519/14951968227866.png)

3）文件删除和擦写条件

调试时发现，敲诈者会对一些特定的文件再加密后对原文件进行擦写（填充随机数，为了防止数据回复）。操作首先找到擦写文件的逻辑，可以知道要擦写文件，有两个必要条件：

条件1：传入的this+902即back目录必须不为空。 

[![44.png](http://image.3001.net/images/20170519/14951968416897.png!small)](http://image.3001.net/images/20170519/14951968416897.png)

条件2：传入给上层函数的参数a4，需要等于4。 

[![45.png](http://image.3001.net/images/20170519/14951968542828.png!small)](http://image.3001.net/images/20170519/14951968542828.png)

往上层函数追溯，发现，该参数来自于函数10002940。 

分别有3和4，两个参数： 

[![46.png](http://image.3001.net/images/20170519/14951968717869.png!small)](http://image.3001.net/images/20170519/14951968717869.png)

而再往上层，该参数取决于函数10002E70。

通过分析，得知该函数逻辑如下，会对文件列表做四部处理：

> 第一步：对文件类型为类型一（见后文）进行处理，做大小判断，大于200M返回3，小于1k返回1，其余大小则返回4。类型二的文件，均返回1。 
>
> 第二步：对剩余文件中，所有类型一的文件均返回1。而对其他类型文件，做大小判断，大于200M的文件返回3，大于1k小于200M的文件返回4，小于1k的文件返回1。 
>
> 第三步：对剩余文件，返回4。 
>
> 第四步：对wncryt文件，返回2，wncyr、exe、dll文件，返回1。

至于为什么要这么做，可能是考虑擦写效率问题，为了尽可能多的加密、擦写文件，耗时的文件、小于1k不重要的文件最后处理。其中，返回为1的，不做处理；返回为2的，做删除处理；返回为3的：生成加密文件头，而后做文件头转移处理，创建为“WNCRYT”文件，而后生成假的加密文件“WNCRY”，往里填充随机数；返回为4的，做完整文件加密处理。 

其中判断文件类型的代码在：

[![47.png](http://image.3001.net/images/20170519/14951968974966.png!small)](http://image.3001.net/images/20170519/14951968974966.png)

类型一：

```
doc、docx、xls、xlsx、ppt、pptx、pst、msg、、vsd、vsdx、txt、csv、rtf、123、wks、wk1、pdf、dwg、onetoc2、snt、jpeg、jpg
```

类型二：

```
docb、docm、dot、dotm、dotx、xlsm、xlsb、xlw、xlt、xlm、xlc、xltx、xltm、pptm、pot、pps、ppsm、ppsx、ppam、potx、potm、edb、hwp、602、sxi、sti、sldx、sldm、sldm、vdi、vmdk、vmx、gpg、aes、arc、paq、bz2、tbk、bak、tar、tgz、gz、7z、rar、zip、backup、iso、vcd、bmp、png、gif、raw、cgm、tif、tiff、nef、psd、ai、svg、djvu、m4u、m3u、mid、wma、flv、3g2、mkv、3gp、mp4、mov、avi、asf、mpeg、vob、mpg、wmv、fla、swf、wav、mp3、sh、class、jar、java、rb、asp、php、jsp、brd、sch、dch、dip、pl、vb、vbs、ps1、bat、cmd、js、asm、h、pas、cpp、c、cs、suo、sln、ldf、mdf、ibd、myi、myd、frm、odb、dbf、db、mdb、accdb、sql、sqlitedb、sqlite3、asc、lay6、lay、mml、sxm、otg、odg、uop、std、sxd、otp、odp、wb2、slk、dif、stc、sxc、ots、ods、3dm、max、3ds、uot、stw、sxw、ott、odt、pem、p12、csr、crt、key、pfx、der
```

如此，第二个条件已理清楚，剩下第一个，在分析时发现：程序运行后，首先对桌面、文档、所有人\桌面、所有人\文档四个文件夹进行处理。

[![48.png](http://image.3001.net/images/20170519/14951969194257.png!small)](http://image.3001.net/images/20170519/14951969194257.png)

而对这几个文件夹处理的时候，back目录为空。对其他目录进行处理的时候，会进行判定：

[![49.png](http://image.3001.net/images/20170519/14951969351080.png!small)](http://image.3001.net/images/20170519/14951969351080.png)

a. 处理C盘其他目录时，back目录为%temp% 

b. 处理其他盘符时，back目录为各盘符下的回收站目录，若回收站不存在，则创建

由此，可以得到整体逻辑如下：

a. 对于桌面、文档、所有人\桌面、所有人\文档四个文件夹下的文件，文件类型为类型一、类型二的文件，大小小于200M的，均进行加密后擦写原文件，大于200M的文件不做加密也不做擦写操作，仅作转移文件头处理，后生成随机数填充的WNCRY文件。 

b. 对于其他目录下文件：由于back目录不为空，不会对原文件进行擦写，而是直接删除，或者移动到back目录中。移动到back后，的文件命名为%d.WNCRYT，程序taskdl.ext会对back目录下的这些文件定时删除。同样，大于200M的文件不做加密也不做擦写操作，仅作转移文件头处理。

4）文件擦写方式

擦写方式主要是对加密后的原文件进行重写。根据代码可知，其主要填充数据有两种：随机数、0×55。

[![50.png](http://image.3001.net/images/20170519/14951969584477.png!small)](http://image.3001.net/images/20170519/14951969584477.png)

而擦写过程为：

> a. 先重写尾部1k 
>
> b. 判断大小后重写尾部4k 
>
> c. 从文件头开始每256k填充一次

5）解密通信

敲诈者会在Tor目录下释放并拉起taskhsvc.exe，该工具启动后会监听本地9050端口，病毒通过本地代理连接实现与Tor服务器的通信。

[![51.png](http://image.3001.net/images/20170519/149519697981.png!small)](http://image.3001.net/images/20170519/149519697981.png)

在点击“Check Payment”按钮后，上传本地res和eky文件。

[![52.png](http://image.3001.net/images/20170519/14951969893445.png!small)](http://image.3001.net/images/20170519/14951969893445.png)

若私钥下发，则会在本地生成解密所需要的dky文件。

[![53.png](http://image.3001.net/images/20170519/14951970002202.png!small)](http://image.3001.net/images/20170519/14951970002202.png)

有精力可以再分析一下和Tor通信的协议，说不定，能有些啥出来呢~

[![54.png](http://image.3001.net/images/20170519/14951970128991.png!small)](http://image.3001.net/images/20170519/14951970128991.png)

[![55.png](http://image.3001.net/images/20170519/14951970278671.png!small)](http://image.3001.net/images/20170519/14951970278671.png)

6）RES文件解析

解密同时需要res文件和eky，就来分析一下res文件格式。通过分析，可得其格式大致如下：

[![56.png](http://image.3001.net/images/20170519/14951970398667.png!small)](http://image.3001.net/images/20170519/14951970398667.png)

红色框为八字节随机生成，应该是用于唯一性判定；蓝色框中为敲诈者首次运行时间，即首个文件加密时间；橘色框中为最后一个文件加密时间；紫色框中为加密文件总数；褐色框中记录的是加密文件总大小。 

有这些信息后，作者应该可以初步对非正常用户进行排除。当然，具体如何判定，还不得而知。

## 四、变种及关联样本

**![57.png](http://image.3001.net/images/20170519/14951970537349.png!small)**

WannCry变种情况：腾讯电脑管家从MD5维度监控到数百个变种，其中很多是因为网络传输过程中尾部数据损坏导致，还有一部分变种多为在原始样本上做修改，如：Patch、加壳、伪造签名、再打包等方式，还没有监测到真正源码级变化的变种。

相关样本：

1、永恒之蓝漏洞传播的其他样本：Onion敲诈者、挖矿样本和UIWIX样本。

2、Onion敲诈者针对服务器，对EXE，DLL，PHP等文件都进行加密，勒索金额较高，6个比特币。

3、挖矿样本是一个“门罗币”的挖矿程序，中毒后会通过IP策略阻止其他攻击者对中毒机器的攻击。

4、UIWIX技术难度更高，文件不落地在内存中执行，这两个相关样本都不具备蠕虫功能。

中毒情况：虽然不断出现变种，整体接触样本和中毒用户数量呈现明显下降趋势，电脑管家能够及时查杀和防御。

典型变种详细情况

| **时间**      | **变种**                                   | **描述**                                   | **当前危害**                     | **感染数量** |
| ----------- | ---------------------------------------- | ---------------------------------------- | ---------------------------- | -------- |
| 5月12日       | WannaCry原始样本                             | 域名开关地址：iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com。  比特币钱包地址：  115p7UMMngoj1pMvkpHijcRdfJNXj6LrLn       12t9YDPgwueZ9NyMgw519p7AA8isjr6SMw13AM4VW2dhxYgXeQepoHkHSQuy6NgaEb94 | 域名开关已经打开，对于能联网的机器不具备感染和传播能力。 | 数万       |
| 5月13日       | “想妹妹”变种                                  | 主要通过加壳实现在傀儡进程notepad.exe中执行病毒代码，逃避杀软检测。  | 传播能力弱                        | 较少       |
| 域名开关修改变种    | 开关域名改为：ifferfsodp9ifjaposdfjhgosurijfaewrwergwea.com | 域名开关已经打开，对于能联网的机器不具备感染和传播能力。             | 较少                           |          |
| 5月14日       | Excel文件图标变种                              | 把图标改为excel文件图标，增加欺骗性。  ** **             | 传播能力弱                        | 较少       |
| Patch域名开关变种 | Patch域名开关并产生传播的变种样本                      | 具有一定的传播能力                                | 较少                           |          |
| 5月15日       | “黑吃黑”变种                                  | 出现“黑吃黑”版变种，修改3个比特币收款帐号，并修改开关域名，而且受害者支付的比特币被黑，无法获取解密私钥，  域名开关：ayylmaoTJHSSTasdfasdfasdfasdfasdfasdfasdf.com  比特币钱包地址：  15nzzRpAsbgd1mmoqQRtiXxN49f4LcmTh4  18ucAGbkgCkU61F6yPMD19dZRUBBHyDGRV  1M9sgF4zhpusQA82rtTbrcZGKD5oBrSW5t | 域名开关已经打开，对于能联网的机器不具备感染和传播能力。 | 较少       |
| 虚假签名变种      | 出现通过添加虚假数字签名来尝试免杀的样本                     | 传播能力较弱                                   | 较少                           |          |
| 域名开关patch变种 | 再次出现多个修改开关域名的变种样本，使得样本可以在联网情况下进行感染传播     | 具有一定的传播能力                                | 较少                           |          |
| 5月16日       | 伪装知名安装包变种                                | 出现伪装成知名软件安装包的样本，增加欺骗性                    | 具有一定的传播能力                    | 较少       |

其他关联样本

| **时间** | **名称**               | **描述**                                   | **当前危害**          | **感染数量** |
| ------ | -------------------- | ---------------------------------------- | ----------------- | -------- |
| 最早4月份  | 发现挖矿木马通过“永恒之蓝”漏洞传播   | 该挖矿木马广度较大，黑客通过批量扫描445端口植入该木马，木马本身不具有自动传播功能，此外木马运行后会通过策略关闭445端口，阻止“永恒之蓝”类似攻击手法的木马进入系统 | 没有自传播能力，用永恒之蓝漏洞传播 | 数万       |
| 最早4月份  | ONION敲诈者通过“永恒之蓝”漏洞传播 | 敲诈者ONION经分析，也是通过“永恒之蓝”漏洞进行传播，黑客通过批量扫描445端口进行传播感染的，本身没有蠕虫传播能力，该木马最早于4月18号出现，是利用该漏洞传播最早的木马之一 | 没有自传播能力，用永恒之蓝漏洞传播 | 较少       |
| 5月9号   | UIWIX敲诈者通过“永恒之蓝”漏洞传播 | 敲诈者UIWIX经分析，也是通过“永恒之蓝”漏洞进行传播，该木马全程无文件落地，较难以防御和查杀，不过该木马也是黑客通过批量扫描445端口进行传播感染的，本身没有蠕虫传播能力 | 没有自传播能力，用永恒之蓝漏洞传播 | 较少       |

## **五、安全建议**

腾讯电脑管家管家提供以下安全建议：

1、关闭445、139等端口，方法详见：<http://mp.weixin.qq.com/s/7kArJcKJGIZtBH1tKjQ-uA>

2、下载并更新补丁，及时修复漏洞（目前微软已经紧急发布XP、Win8、Windows server2003等系统补丁，已经支持所有主流系统，请立即更新）；

3、安装腾讯电脑管家，管家会自动开启主动防御进行拦截查杀；

4、对于已经中毒的用户，可以使用电脑管家文件恢复工具进行恢复，也可以使用电脑管家提供的解密工具尝试解密，地址（<http://t.cn/RaEHKZq>）。

5、建议用户使用微云，10G免费空间日常备份。

## **六、比特币支付以及解密流程**

1、 中毒后用户电脑里面文档会被加密为后缀为.wncry的文件。

[![58.png](http://image.3001.net/images/20170519/1495197089797.png!small)](http://image.3001.net/images/20170519/1495197089797.png)

2、弹出提示框提示用户付款比特币地址。

[![59.png](http://image.3001.net/images/20170519/14951971089740.png!small)](http://image.3001.net/images/20170519/14951971089740.png)

3、购买比特币往这个比特币钱包付款以后，通过病毒的洋葱网络的匿名通信通道把付款的钱包地址发给病毒作者。

[![60.png](http://image.3001.net/images/20170519/14951971245729.png!small)](http://image.3001.net/images/20170519/14951971245729.png)

4、作者如果匿名网络在线，点击“check payment”会收到回复的确认消息，这个时候用于解密的密钥会通过匿名网络发送回来。

[![61.png](http://image.3001.net/images/20170519/14951971351910.png!small)](http://image.3001.net/images/20170519/14951971351910.png)

5、然后选择解密就可以解密文件了。

[![62.png](http://image.3001.net/images/20170519/14951971491389.png!small)](http://image.3001.net/images/20170519/14951971491389.png)

[![63.png](http://image.3001.net/images/20170519/14951971564537.png!small)](http://image.3001.net/images/20170519/14951971564537.png)

6、解密完成后是仍然有可能重新中毒的，病毒并没有标签解密过的机器。

## **七、比特币和洋葱网络**

### 1、 比特币介绍

1）比特币

比特币（BitCoin）的概念最初由中本聪在2009年提出，根据中本聪的思路设计发布的开源软件以及建构其上的P2P网络。比特币是一种P2P形式的数字货币。点对点的传输意味着一个去中心化的支付系统。

与大多数货币不同，比特币不依靠特定货币机构发行，它依据特定算法，通过大量的计算产生，比特币经济使用整个P2P网络中众多节点构成的分布式数据库来确认并记录所有的交易行为，并使用密码学的设计来确保货币流通各个环节安全性。P2P的去中心化特性与算法本身可以确保无法通过大量制造比特币来人为操控币值。基于密码学的设计可以使比特币只能被真实的拥有者转移或支付。这同样确保了货币所有权与流通交易的匿名性。比特币与其他虚拟货币最大的不同，是其总数量非常有限，具有极强的稀缺性。该货币系统曾在4年内只有不超过1050万个，之后的总数量将被永久限制在2100万个。

2）货币特点

a. 完全去处中心化，没有发行机构，也就不可能操纵发行数量。其发行与流通，是通过开源的p2p算法实现。

b. 匿名、免税、免监管。

c. 健壮性。比特币完全依赖p2p网络，无发行中心，所以外部无法关闭它。

d. 无国界、跨境。跨国汇款，会经过层层外汇管制机构，而且交易记录会被多方记录在案。但如果用比特币交易，直接输入数字地址，点一下鼠标，等待p2p网络确认交易后，大量资金就过去了。不经过任何管控机构，也不会留下任何跨境交易记录。

（参考资料：<http://baike.baidu.com/item/%E6%AF%94%E7%89%B9%E5%B8%81/4143690>）

### **2、洋葱网络介绍**

洋葱网络是一种在计算机网络上进行匿名通信的技术。通信数据先进行多层加密然后在由若干个被称为洋葱路由器组成的通信线路上被传送。每个洋葱路由器去掉一个加密层，以此得到下一条路由信息，然后将数据继续发往下一个洋葱路由器，不断重复，直到数据到达目的地。这就防止了那些知道数据发送端以及接收端的中间人窃得数据内容。

（参考资料：<http://baike.baidu.com/item/%E6%B4%8B%E8%91%B1%E7%BD%91%E7%BB%9C>）

*** 本文作者：腾讯电脑管家（企业帐号），转载请注明来自FreeBuf.COM**