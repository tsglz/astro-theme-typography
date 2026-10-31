---
title: TeslaCrypt样本分析
pubDate: 2026-08-12
categories: ['样本分析']
description: '简单分析一个 TeslaCrypt 家族的样本'
slug:
---

参考文章：https://blog.csdn.net/qq_42814021/article/details/122536584

样本 hash：83e62221df7ba94f18fc9b63fec82285d8694c0c0a2542c0196752dc81524a10

这个样本是之前测试项目的时候选用的一个比较简单的样本，因为涉及到了反沙箱导致项目测试出现了一些问题，这里我手动分析排查了一下问题，这里留档记录，很适合给新人练手。

项目最开始就是一个沙箱检测。通过 COM 接口返回值进行第一次反沙箱检测，设置非法调用 GetClassID(NULL)：

- 真实 Windows：GetClassID 检测到 NULL → 返回 E_POINTER
- 沙箱/模拟器：COM stub 返回其他内容 → 不匹配 → ExitProcess(-1)

<img src ="/TeslaCrypt/anti-sandbox-1.png">

然后通过 DirectShow Filter Graph 做环境真实性探测：

<img src ="/TeslaCrypt/anti-sandbox-2.png">

这个样本的敏感内容在文件中是加密处理的，只有在使用的时候会进行解密。

<img src ="/TeslaCrypt/decrypt.png">

使用了 aes 加密，密钥为：

> AES-256 key: 7abe8ba3a5fc704269791f472e7961f729b73f99b92ce2487cb263094c163900

解密内容对应如下 5 块：

```
sub_402170 解密
    ├─ dword_45F4D8  → 要加密的文件扩展名白名单
    ├─ dword_45F4DC  → 遍历/跳过规则 + 勒索文件名模板
    ├─ dword_45F4E0  → 主行为配置（提权、清影拷、持久化、杀进程…）
    ├─ dword_45F4E4  → C2 URL / HTTP 协议字段
    └─ dword_45F4E8  → 勒索信模板（XXXXXXXXXXXXX 占位符）
```

解密出的内容如下：

```
==== dword_45F4D8 0x435f80 len 0x810 ====
hex[:48]: 2e007200330064003b002e007000740078003b002e007000650066003b002e007300720077003b002e00780033006600
fields(192):
  [0] '.r3d'
  [1] '.ptx'
  [2] '.pef'
  [3] '.srw'
  [4] '.x3f'
  [5] '.der'
  [6] '.cer'
  [7] '.crt'
  [8] '.pem'
  [9] '.odt'
  [10] '.ods'
  [11] '.odp'
  [12] '.odm'
  [13] '.odc'
  [14] '.odb'
  [15] '.doc'
  [16] '.docx'
  [17] '.kdc'
  [18] '.mef'
  [19] '.mrwref'
  [20] '.nrw'
  [21] '.orf'
  [22] '.raw'
  [23] '.rwl'
  [24] '.rw2'
  [25] '.mdf'
  [26] '.dbf'
  [27] '.psd'
  [28] '.pdd'
  [29] '.pdf'
  [30] '.eps'
  [31] '.jpg'
  [32] '.jpe'
  [33] '.dng'
  [34] '.3fr'
  [35] '.arw'
  [36] '.srf'
  [37] '.sr2'
  [38] '.bay'
  [39] '.crw'
  [40] '.cr2'
  [41] '.dcr'
  [42] '.ai'
  [43] '.indd'
  [44] '.cdr'
  [45] '.erf'
  [46] '.bar'
  [47] '.hkx'
  [48] '.raf'
  [49] '.rofl'
  [50] '.dba'
  [51] '.db0'
  [52] '.kdb'
  [53] '.mpqge'
  [54] '.vfs0'
  [55] '.mcmeta'
  [56] '.m2'
  [57] '.lrf'
  [58] '.vpp_pc'
  [59] '.ff'
  [60] '.cfr'
  [61] '.snx'
  [62] '.lvl'
  [63] '.arch00'
  [64] '.ntl'
  [65] '.fsh'
  [66] '.itdb'
  [67] '.itl'
  [68] '.mddata'
  [69] '.sidd'
  [70] '.sidn'
  [71] '.bkf'
  [72] '.qic'
  [73] '.bkp'
  [74] '.bc7'
  [75] '.bc6'
  [76] '.pkpass'
  [77] '.tax'
  [78] '.gdb'
  [79] '.qdf'

==== dword_45F4DC 0x436790 len 0x80 ====
hex[:48]: 2e0063006f006d003b005200450063006f007600450052003b00250073005c002b00250073002b00250073002b002500
fields(10):
  [0] '.com'
  [1] 'REcovER'
  [2] '%s\\+%s+%s+%s'
  [3] '.png'
  [4] '.txt'
  [5] '.html'
  [6] 'A:\\'
  [7] 'B:\\'
  [8] '\\*.*'
  [9] 'recove'

==== dword_45F4E0 0x436810 len 0x460 ====
hex[:48]: 770069006e006400690072003b00250073005c00730079007300740065006d00330032005c0063006d0064002e006500
fields(30):
  [0] 'windir'
  [1] '%s\\system32\\cmd.exe'
  [2] '/c start "" "'
  [3] 'runas'
  [4] 'vssadmin.exe'
  [5] ' Delete Shadows /All /Quiet '
  [6] 'open'
  [7] '%s\\%s'
  [8] '%s\\%s.exe'
  [9] 'SeDebugPrivilege'
  [10] 'Software\\%s'
  [11] 'data'
  [12] 'S-1-5-18\\'
  [13] 'S-1-5-18\\Software\\MSVCC\\'
  [14] '%X%X%X%X%X%X%X%X'
  [15] 'Software\\MSVCC\\'
  [16] 'FIX'
  [17] '\\+recover+file.txt'
  [18] '%s\n%s\n%S\n%d\n'
  [19] 'image/png'
  [20] 'Software\\Microsoft\\Windows\\CurrentVersion\\Run'
  [21] '%s\\SYSTEM32\\CMD.EXE /C START "" "'
  [22] 'SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Policies\\System'
  [23] 'EnableLinkedConnections'
  [24] 'taskmg'
  [25] 'regedi'
  [26] 'procex'
  [27] 'msconfi'
  [28] 'cmd'
  [29] 'reg add HKEY_CURRENT_USER\\Software\\Microsoft\\Windows\\CurrentVersion\\Run /v %s /t REG_SZ  /d "%s" /f'

==== dword_45F4E4 0x438f10 len 0x210 ====
hex[:48]: 687474703a2f2f766964656f616d696e70726f64756b74696f6e2e64652f706c7567696e732f62696e7374722e706870
fields(16):
  [0] 'http://videoaminproduktion.de/plugins/binstr.php'
  [1] 'http://clubsaintandre.fr/images/binstr.php'
  [2] 'http://affiliateproductes.com/binstr.php'
  [3] 'http://ptgp.pl/tmp/binstr.php'
  [4] 'http://strategicdisaster.info/wordpress/wp-content/plugins/binstr.php'
  [5] 'http://minteee.com/images/binstr.php'
  [6] 'Sub=%s&dh=%s&addr=%s&size=%lld&version=4.0&OS=%ld&ID=%d&inst_id=%X%X%X%X%X%X%X%X'
  [7] '0324532423723948572379453249857'
  [8] 'Mozilla/5.0 (Windows NT 6.3 rv:11.0) like Gecko'
  [9] 'Content-Type: application/x-www-form-urlencoded'
  [10] '*/*'
  [11] 'Crypted'
  [12] 'Ping'
  [13] 'data=%s'
  [14] 'POST'
  [15] 'INSERTED'

==== dword_45F4E8 0x436c70 len 0x9a0 ====
hex[:48]: 2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d
utf16 preview: 'ⴭⴭⴭⴭⴭⴭⴭⴭⴭⴭⴭⴭⴭⴭⴭⴭⴭⴭⴭⴭⴭⴭⴭⴭⴭⴭ\u0a0d低⁔余剕䰠乁啇䝁㽅唠䕓栠瑴獰⼺琯慲獮慬整朮潯汧\u2e65潣൭ഊ圊慨❴\u2073桴\u2065慭瑴牥眠瑩\u2068潹牵映汩獥ിഊ夊畯\u2072慤慴眠獡猠捥牵摥甠楳杮愠猠牴湯\u2067湥牣灹楴湯眠瑩\u2068卒㑁㤰⸶\u0a0d獕\u2065桴\u2065楬歮搠睯\u206e敢潬⁷潴映湩\u2064摡楤楴湯污椠普牯慭楴湯漠\u206e桴\u2065湥牣灹楴湯欠祥\u2073獵湩\u2067卒ⵁ〴㘹栠瑴獰⼺支\u2e6e楷楫数楤\u2e61牯⽧楷楫刯䅓⡟牣灹潴祳瑳浥\u2029\u0a0d\u0a0d桗瑡攠慸瑣祬琠慨⁴敭湡㽳\u0a0d\u0a0d瑉洠慥獮琠慨⁴湯愠猠牴捵畴慲\u206c敬敶\u206c潹牵映汩獥栠癡\u2065敢湥琠慲獮潦浲摥⸠夠畯眠湯琧戠\u2065扡敬琠\u206f獵\u2065\u202c敲摡Ⱐ猠敥漠\u2072潷歲眠瑩\u2068桴浥愠祮潭敲⸠\u0a0d湉漠桴牥眠牯獤琠敨⁹牡\u2065獵汥獥\u2073\u202c潨敷敶\u2072\u202c桴牥\u2065獩愠瀠獯楳楢楬祴琠\u206f敲瑳牯\u2065'
ascii preview: b"----------------------------------------------------\r\nNOT YOUR LANGUAGE? USE https://translate.google.com\r\n\r\nWhat's the matter with your files?\r\n\r\nYour data was secured using a strong encryption with RSA4096.\r\nUse the link down below to find additional information on the encryption keys using RSA-40"
```

样本通过内置模块 id 来调用相应 dll，对应表如下：

| ID   | DLL                 |
| ---- | ------------------- |
| 1    | kernel32.dll（PEB） |
| 2    | advapi32.dll        |
| 3    | user32.dll          |
| 4    | ws2_32.dll          |
| 5    | ntdll.dll           |
| 6    | winsta.dll          |
| 7    | shell32.dll         |
| 8    | wininet.dll         |
| 9    | urlmon.dll          |
| 15   | Gdi32.dll           |
| 16   | gdiplus.dll         |
| 17   | crypt32.dll         |
| 18   | SHLWAPI.dll         |

样本中还用到了 api hash 技术，如下图所示（sub_4013B0：API hash 函数）：

<img src ="/TeslaCrypt/api-hash-1.png">

函数内部：

<img src ="/TeslaCrypt/api-hash-2.png">

sub_406EB0：结束进程的线程，结束进程名中代码 taskmg、regedi、procex、msconfi、cmd 的进程：

<img src ="/TeslaCrypt/api-hash-3.png">

样本中主要功能如下图所示：

<img src ="/TeslaCrypt/main-function.png">

红框中功能如下表，只有先解密了配置文件才能确定这些函数中的功能，加密+反沙箱导致了项目中的分析结果部分无法正常显示。

| 调用       | 功能                                                         |
| ---------- | ------------------------------------------------------------ |
| sub_40FB60 | 生成随机串；从 Shell32 取系统字符串；缓存桌面/公共文档等特殊目录（CSIDL 59/16/25） |
| sub_402940 | 身份/密钥初始化：读或写注册表 MSVCC 等；若是首次则 sub_402980 生成 RSA/会话材料，并设 dword_45F50C = 1 |
| sub_40FF10 | 持久化：写 HKCU\...\Run（随机值名 +++xxxx，命令用配置里的 CMD /C START 启动自身）；再写 HKLM\...\Policies\System\EnableLinkedConnections = 1（映射盘对提权进程可见） |
| sub_40FBE0 | 在「我的文档」写下加密的 +recover+file.txt（配置 +0x44），内容含 ID、公钥十六进制、机器名、PE CheckSum 等 |
| sub_402430 | 用注册表里的 ID 格式化成 %X%X...，塞进勒索信模板（替换 XXXXXXXXXXXXX）→ 内存中的完整勒索文案 |
| sub_40FE60 | 初始化 GDI+，按配置 image/png 相关逻辑准备位图（勒索信配图一类） |

其中 sub_40FF10：
(HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run)这部分内容是从之前的解密内容中拼出来的

以及 sub_40FBE0：
+recover+file.txt 这个名字也是从解密内容中提取的

因为分析这个样本只是为了定位其他项目中因为反沙箱等因素没有成功获取信息的问题，这个样本中部分内容没有着重分析，比如样本中还存在自拷贝、结束当前进程后重新启动，多线程执行功能等内容，这里不多赘述。

由于这个样本过于简单，个人认为最有价值的部分在利用 COM 接口反沙箱的部分。