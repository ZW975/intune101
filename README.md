# 苹果，安卓，MAC设备注册失败要怎么办

## 对于苹果设备

苹果设备一切的前提条件是需要 apple MDM push certificate。
一般情况下，比如说像BYOD这样的情况，如果是download management profile失败的话，大概率是网络的问题，我们就去建议用户换一个网络试试，或者说看看其他设备在相同的网络下是否也都是一样的情况。如果说是install management profile这个阶段失败或者说检查设置这里超过十分钟的话，可以看看这台设备是否以前注册过Intune。如果说是一台全新的设备，那么可以去收集一下company portal log。可以通过company portal app里的setting and help去上传日志，上传完之后会有一个incident id，拿到id之后可以去后台（powerlift）用这个id把log导出来，然后在log里搜关键词WPJ（work place joined）就可以看到一些相关的信息

ADE也是可以收company portal log的，但假设你选的认证方式是modern authentication的话，然后在输入Microsoft账号密码的阶段就报错了的话，这时候我们就可以去收一个console log。收这个log需要用数据线把设备和mac连接，然后用mac的一个内置应用console。打开console然后点左上角的actions，把debug和info message勾选上，然后点record，然后把问题复现之后再stop。这时候会出现很多log的内容，因为它不能直接导出来，所以我们可以把这些内容复制粘贴到TXT文档上拿去分析。这里也可以用Windows来连接设备去收log，不过会用到一个第三方软件叫xlog，所以一般还是推荐客户去用mac来收。

## 对于安卓设备

像Device Administrator的话首先要确保device platform restriction创建了一条block掉AE的restriction，并且把这条的优先级调到最高，否则在同等条件下，AE的优先级永远高于DA，只有在将AEblock掉或者因为GMS的原因AE不可用的时候，DA才会变成首先注册到intune 的方式。如果DA出现问题也是可以去通过company portal的incident id来收log的。

android BYOD设备不需要恢复出厂设置，可以直接下载company porta登录intune账号，然后下载配置文件，这个注册出现了问题也可以收company portal log。
如果说是AE的一些设备，像dedicated，fully managed和corporate owned，这些没有company portal这个应用，不过他们可以通过Microsoft Intune这个应用来收log，从这里收上来的log和从company portal上收上来的log没有什么区别。如果说是在扫QR code的阶段出现问题话，就得搜debug log，这个和console log的形式差不多，也会用到一个USB，然后会需要在settings里的developer options里边复现问题然后收report再发到邮箱里。
除此之外对于dedicated设备，如果是在展台模式下，在manage home screen中的展示的应用，首先需要添加到google 商店中，并且要新建一个android平台的device configuration，选择manage home screen需要展示的app，可以调节这些app 的layout。
manage home screen出现了问题，那么侧拉多次会出现一个弹窗，可以收MHS的log。

## 对于MAC设备

MAC是不能直接在store里下company portal的，它可以通过官方的一个网页下载，这个网页链接在Microsoft的文档里有给到https://go.microsoft.com/fwlink/?linkid=853070。然后下载下来之后再去收company portal log就行了。

# windows BYOD注册
首先确定windows版本，crtl+R winver
Windows10，1607及以后版本的在Windows store中下载company portal,连接到组织账号，出错的话可以收company portal log。
Windows10，1511及之前版本的在设置中，account-add a work or school account-sign in，之后的版本也可以用这种方式注册，这属于automatic enrollment的BYOD，首先要在entra ID portal中查看mobility，MDM user scope有没有打开为all
但如果是家庭版本的设备，只能register到entra，在setting中输入组织账号，首先去entra中看有没有 Microsoft entra registered，然后再回到intune portal中查看状态，如果注册不成功会报错，在详情页上得到error code。
或者也可以在event viewer里面device management provider -admin，查看到注册成功或失败的event id。

# Autopilot失败怎么办

## 在device preparation阶段失败

如果说是在Secure hardware这里失败的话，说明设备的某些硬件要求没达到标准，这里更多的是针对self-deploying和pre-provisioned的设备，他们有一个tpm2.0的要求

如果说是在join organization's network这里失败的话，说明设备没法join到AAD，这里也是针对self-deploying和pre-provisioned的设备，标准的user-driven在进入到ESP这个界面时就已经完成了这个操作了。这里失败的话就得去Azure portal上看Device settings，里面有一个Users may join devices to Microsoft Entra，这里是否设置成了all，设置成some的话这台设备是否有包含在里面。另外可以看maximum number of devices per user，这里是否超过了限制，一般来说recommend的是20台，不过用户也可以自定义到1-unlimited。

如果说是在register device for mobile management这里失败的话，说明设备没法enroll到intune，这里就可以去通过azure或者intune portal上看是否MDM user scope是设置成all的，如果设置成some了的话，这台设备是否包含在内。另外还可以在intune portal上看device platform restriction,看是否block了MDM或者设置了version要求。同时可以看看device limit restriction是否超过限制了，非DEM的情况下，一个standard user最多是可以enroll15台。另外还可以看看这些都检查完之后，可以让客户再去敲一个MDMDiagnosticstool -area Autopilot -cab的命令去收集log，收到之后我们在cab包里主要看的是provier-admin的event log，打开之后看里面是否有event id 75或76，失败的话我们更多的是找76，找到了的话它旁边可能会给到一些有用的error code或者error message，我们就能进一步去看看是什么原因。如果这两个log我们都没看到的话，那说明这个注册可能根本就没触发，我们可就需要进一步跟客户去确认是否是网络或者设备层面的原因。网络的话可以看用户在相同的网络下其他设备是否有注册成功的，另外用户换一个网能否解决问题。如果还是不成功的话，可以尝试用wireshark抓包，来看是不是intune endpoint url的连接有问题。设备层面的话得达到autopilot的prerequisites，比如说如果用户的设备是home edition的话那就是没有办法做autopilot的。

## 在device setup阶段失败

如果说是在**security policies**这个阶段失败的话，说明我们设置在intune portal上的policy可能出问题了。我们可以找用户先确认一下最近是否有修改过policy的一些设置，如果有的话我们的范围就能迅速缩小。我们也可以去intune portal上找到这台设备，然后点进这台设备里去看它的configuration policy或者compliance policy的state是怎样的，这样我们就能快速锁定具体是哪一条policy出了问题。一般来说是看configuration，因为compliance的成功失败不会直接影响到autopilot的成功或失败，不过它可以作为一个参考看是哪里没有compliant。然后configuration policy这边我们可以找到出问题的策略，然后点进去看设置是否是正确的，比如说可能有些策略是要求设备重启的话，就有可能会导致autopilot的失败。还有可以看configuration assignment这边是下发给user还是device，因为有些CSP要求可能只能下发给user，或者说有些只能下发给device。另外我们也需要让客户去敲一个MDMDiagnosticstool area autopilot cab的命令去把log收上来，收上来之后我们就可以去打开provider-admin的event log，在这里面通过policy的关键词去搜索，看是否出现一些error message和error code去锁定具体原因。
registry dump里搜索policy manager的关键词，找到这些policy的路径后可以直接去注册表看对应的policy的data值或者value值是怎样的，来确认我们intune是否部署成功。如果能找到这条policy，并且policy设置的值跟我们部署的是一样的，那可能跟我们Intune的关系就不是很大。

如果说是在**certificate profiles**这个阶段失败的话，说明SCEP profile的install可能出现问题了，我们就需要看对应的profile设置和assignment是否正确。

如果说是在**network connections**这个阶段失败的话，就需要去确认一下网络层面的原因。试着更换一个网络，或者可以用`netsh trace start scenario=internetClient capture=yes maxSize=2048` 这个命令检查网络状况
 `netsh trace stop` 停止检查。确认是网络的问题可以转交给网络组解决问题。

如果说是在**Apps**这个阶段失败的话，说明我们在intune上部署的app要么下载不成功，要么安装不成功。我们得先去确认具体是由哪一个app导致的失败，这个我们可以去intune portal上看到一些app失败的信息。也可以看看是否有LOB和win32的app在一起推，因为在autopilot阶段一起推的话他们installer的组件会起冲突，也可能会导致一个失败。用shift+f10唤出命令行，用`mdmdiagnosticstool. exe -area AutoPilot -cab %temp%|%COMPUTERNAME%.cab` 收cab包，然后跑一个`Get-AutopilotDiagnostics cab`的命令去获取一条autopilot的时间线，这里我们也可以看到具体是在哪一个app上失败的以及失败的时间。总之确认好是哪个app后就要去看对应的信息

**LOB app**，首先确定是安装问题还是下载问题

 - 下载问题
 在路径`C：\Windows\System32\config\systemprofile\AppData\Local\mdm`下没有找到下载的msi安装包可以判定为下载失败
在注册表中定位到`HKLM\SOFTWARE\Microsoft\EnterpriseDesktopAPPManagement`，面向设备的应用定位到s-0000···>MSI，用appid定位到下载失败的msi应用；面向用户的应用定位到userSID/MSI。用appid定位到下载失败的msi应用
再次下载msi文件可以将EnforcementRetryCount 和 EnforcementRetryIndex 设置为 （0），并将 LastError 和 Status 设置为空字符串。
 - 安装问题
 已经在路径下发现了msi的安装包，那说明是安装的问题，可以用这个命令手动静默安装，并且附带一个日志文件。`msiexec /I <包名>.msi /l*v %temp%\msiverbose.txt /qn`

比如说如果是**Office**软件的话，我们可以去看Windows Temp或者System Temp里看有没有tmp.exe和一个.xml的文件，如果有的话证明我们的Intune是部署下来了的，更多的可能就是安装问题，安装的话这里就需要找office的同事去问问可能是什么原因。如果说是没有下载下来的话，更多的可能就是网络原因，可以换一个网络尝试一下，如果换了网络还是不行的话，那可能就是office endpoint的连接有问题了，这里就只能说去尝试用wireshark来抓包看。

然后如果说是**store new**或者**win32**的app的话，我们可以去看IME Log。因为这个log信息比较多，我们可以先在intune portal上拿到app id之后在这里面搜就会更快一些。对于没安装上的app，问题更多可能是出现在applicability，download，unzipping，和installation这四个阶段。applicability的话就是检测设备的一些硬件或者操作系统是否符合用户在intune portal上设置的，比如说某个app会要求用户是32还是64bit，最低的operating system是多少，或者说要求一些disk space，physical memory或者processor需要达到多少。只有这个阶段通过了才能进入到下载阶段，下载阶段的话就得看这个下载是否已经开始，如果开始了的话，我们应该是能在ime content里找到一个incoming的folder，这里面应该也会有一个bin的文件，我们也可以搜关键词，比如说start downloading来确认是否开始下载。如果显示下载报错，我们可以搜索关键词download url来让客户手动下载看是否能下载下来，如果手动也不能下下来那可能跟网络有一定的关系。然后download阶段如果没问题的话我们就去看unzip阶段，我们可以通过搜索关键词start unzipping来确认是否解压的过程开始了，如果解压失败的话在这条信息的附近应该也是能看到一些原因的。比如说如果显示unauthorized access什么什么的就说明用户可能在这个解压的过程中把那个zip包点开从而导致了一个解压失败。这个阶段是在一个叫staging的文件夹进行的，如果解压成功了的话他会把文件转移到Windows IME Cashe的文件夹里，我们也可以去这两个地方确认到底卡在了哪一步。然后unzip检查完没问题的话就要去看installation，这里用户可以去event log看看跟installer相关的是否有报错以及相关原因，或者说是用户可以尝试跑命令看是否能手动安装下来，像store new的话就是跑一个winget install的组件，这里的话会需要一个package identifier，这个package identifier可以在intune portal的app property里找到。不过在跑之前呢，我们也可以跑一个winget info的命令来确认一下winget或者说是package manager的版本，因为v1.7以前的版本话是有问题的，也可能会导致报错。如果手动跑winget的命令都安装失败的话，那可能跟intune的关系就不大了。然后如果是win32的app的话，我们就是需要用到pstool来模拟一个system context然后去跑命令看是否能手动安装。如果手动也没法安装下来的话可能就需要去找app developer了。如果说是已经安装上了的app出现报错，那大概率是post-install detection这个阶段出问题了。这时候就需要先去确认用户设置的detection rule是什么。是script还是manual configure，如果是script的话，我们要去IMElog里搜exit code，看最后的value是否是0，如果不是0的话，那可能script本身也需要再去看一看。如果是manual configure，并且detection method设置的是file or folder exists的话，我们就要去检查，第一是否路径是否是正确的，其次检测的文件与文件名是否是正确的，我们也可以搜path does not exist的关键词来找到相关信息。如果detection method设置的是date modified，string或者size之类的，我们就可以搜value does not match的关键词来找到相关信息。

# GPO注册成功与否
GPO第一步要确认hybrid环境有没有配置好，在命令提示符下输入dsregcmd /status得到当前设备状态。
并且要确保mdm URL那几栏是有内容的，如果URL是空的，虽然能够在intune portal上看到设备，但是mdm和owner都是空的，这个时候还没有完全完成inune注册，需要确认的是，在server上的active directory domain and trusts下添加组织的UPN，然后将user的UPN更改为组织UPN，确保user的upn和portal中user的upn是一致的。用portal中user的完整名称登录到客户端设备。
然后我们可以查看task scheduler中是否创建了一个GPO enroll的任务，如果没有的话尝试gpupdate /force，并且重新启动设备。
如果还没有检查，server端的创建的组策略是否link到了正确的OU，并且组策略在administrat templates>windows components>MDM下打开了enable automatic MDM enrollment，选择了user credential
最后去event viewer中查看enroll的最终结果。
如果想要看到完整的组策略报告，还可以在命令行输入gpresult /r。

# conditional access

 1. 看先决条件是否满足（Microsoft entra ID p1 licenses，或Microsoft 365 business premium licenses），在启用conditional access的时候需要先关闭intune默认的保护策略。
 2. 看sign-in logs最长可以看到一个月前的
 3. Correlation ID快速确定是哪一次登陆失败，从设备登陆页面的报错提示可以看，网页版是interactive，app内部是non-interactive，app内部报错无法查看详细信息那么就用时间点及时查看。
 4. 由于cap是entra的服务所以找Entra Device ID
 5. 如果entra和intune中的compliant状态不一致看audit log，update的记录

# App deployment
## 安卓

 - AE
 通过google play推应用，稍微有点儿区别的就是dedicated中MHS的显示（上文有提到），用intune来收log
 - 其他
 LOB、web-link，可以上传到google play中再推到设备上
 DA就用android store app这种类型推应用，用company portal收log

## iOS

 - vpp
 确保vpp token的状态是健康的，在vpp token下推应用，可以选择user license或device license，区别在于如果是device license那么assign时选择available就是一个无效的操作
 - 其他

## Windows

 - win32
 - store （new）
 - LOB

# App protection& app configuration

## App protection

首先确定有没有sign out ，sign in一次，手动sync查看配置是否生效，然后根据想要达到的最终需求查看policy配置页面的截图，看具体的设置有没有配对。App protection看到properties页面，policy managed apps分别是哪些，在data protection中想要生效的应用有没有被涵盖到，在assign的时候有没有分配到正确的用户，在设备上登录的user是不是在assign到的用户范围之中。确认完这些信息之后，如果没有问题那么去edge中收集MAM log，输入 about: intune help，得到MAM log然后搜索关键词查找到对应的值参照文档[查看应用保护策略日志 - Microsoft Intune |Microsoft学习](https://learn.microsoft.com/en-us/mem/intune/apps/app-protection-policy-settings-log)看看想要配置到的值是不是对应到了，比如说block应该是value=0等等，如果这个地方值对不上，那可能是intune portal本身出现了问题，联系产品组，如果是值已经对应上了，最终app还是没有生效，那就怀疑是app本身的问题，可能没有intune SDK集成，或者集成失效，这个时候去找对应的app 产品组。

## App configuration
同样的首先需要sign out ，sign in一次，手动sync查看配置是否生效
需要注意的是选择managed device，并且设备平台为iOS时，通过intune下发的应用需要绑上mamupn才能够被识别为managed。
一些常见的microsoft应用会有官方文档指导配置设置，如果是一些其他的应用可以找应用开发商看具体的配置要求。
依照官方文档或应用开发商提供的文档完成setting配置后，确认在assign的时候有没有分配到正确的用户，在设备上登录的user是不是在assign到的用户范围之中。
如果都没有问题，那就去收MAM log。


# configuration & compliance policy
首先都要确认intune portal中到底配置了哪些policy，policy的状态，还有policy的properties具体内容
 - 安卓设备
 主要看company portal log中的 OMADA log
 - iOS设备
 可以看managed profile中新添加了哪些settings
 - Windows设备
 知道是哪个policy失败之后可以去provider-admin看他的状态，然后去注册表根据这条policy的名字看具体的值

# RBAC
在intune中的role和entra ID上的role是两套并行的系统，intune上的权限更加细分，entra上的权限给的更加笼统。
像intune上面会细分，update、read等等权限，而entra上可能只有一个manage（yes/no）
如果已经在entra ID中分配了一个较大的权限，再在intune portal中细分权限不会生效，权限叠加只会按照最高权限的许可来执行

## role不生效时的troubleshooting流程

首先看role中monitor下的的my permission，如果已经配置了权限仍然没有显示的话，可以去查administrator licensing，可以尝试给user分配一个license，一般来说在没有license的情况下也是可以配置权限的，但这部分经常会出问题，可以和intune产品组反映。
  
## 范围标记（Scope Tag）  
  
**定义**：  
- 范围标记是一种标签，用于标记和区分不同的对象（如设备、配置文件等）。  
- 角色分配定义：
哪些用户被分配到角色
他们可以看到哪些资源
他们可以更改哪些资源。
您可以将自定义角色和内置角色分配给用户。
若要分配 Intune 角色，用户必须具有 Intune 许可证。 若要查看角色分配，请选择“Intune”>“租户管理”>“角色”>“所有角色”>选择角色>“分配”>选择分配。
成员：列出的 Azure 安全组中的所有用户都有权管理“作用域（组）”中列出的用户/设备。
作用域（组）：作用域组是用户或设备的 Microsoft Entra 安全组，或两者兼而有之，该角色分配中的管理员只能对其执行操作。例如，将策略或应用程序部署到用户或远程锁定设备。这些 Microsoft Entra 安全组中的所有用户和设备都可以由成员中的用户进行管理。
范围（标签）：成员中的用户可以看到具有相同范围标签的资源。
  
**作用**：  
- 范围标记主要用于角色分配和资源的可见性控制。  
- 管理员可以基于范围标记来控制哪些资源（如设备、配置文件等）对哪些用户或管理员可见和可管理。  
- 通过分配不同的范围标记，可以将不同的对象分配给不同的管理员，从而实现资源分离和管理权限分配。  
  
**使用场景**：  
- 在大型企业中，有不同的管理员负责不同部门的设备管理。可以通过范围标记将设备、配置文件等资源分配给特定的管理员，从而实现精细化的权限控制。  
- 例如，给销售部门的设备分配“Sales”范围标记，给IT部门的设备分配“IT”范围标记，然后相应地给不同的管理员分配不同的范围标记权限。  
  
**示例**：  
- 将某些策略和配置文件标记为“US-East”，使得只有负责US-East区域的管理员可以查看和管理这些策略和配置文件。  

# SCEP

## 证书申请流程

首先intune下发一个配置文件给设备，设备通过配置文件中NDES的URL找到NDES服务器，然后向NDES发送三条指令

 - GetCACAPS： get CA cap是验证CA是否适合颁布这张证书
 - GetCACERT： 用于验证证书链上的trust CA是否已经安装到设备上
 - PKIOperation

NDES Server拿到了这个request之后，会先把这个request递交给connector，connector要做三件事：receive, download, and verify request，request验证成功后， （NDES安装完成后，会自动安装一个application pool到你的server上），NDES上的IIS将request递交给CA，CA颁布证书。request一般只要成功递交到CA上，CA就会颁布证书，颁布证书也是通过http，在pki operation部分，通过IIS，将证书以下载的形式发布到device端。
## troubleshooting流程
Client端

 1. Android收company portal log
 2. IOS收console log，可看的内容比较少，搜SCEP相关的字段，通过关键词检索
 3. Windows端收intune event，log里会告诉你SCEP进行到哪一步，幸运的话可以看到失败的原因

client端看的比较少，一般来说还是主要收NDES server上的IIS log，承载来自client端的所有请求

NDES server
 1. 看NDES log，前面流程说到device会下发几个指令，get CA cap, get CA cert, pki operation，验证完CA cap和CA certificate，才会把request递交给intune certificate connector，C:\inetpub\logs\LogFiles\W3SVC1\u_exlog(check Device->NDES connection) 如果能在IIS log里看到get CA cap, get CA cert，post并且值都是200，就可以看CA connector的日志了
 2. 看application log，PPT P32页，除了上面的log，还可以看Event Log>Application log-NetworkdeviceenrollmentService，筛选出NDES相关的event，看下NDES相关是否有报错
 3. 最后看connector相关的日志，Event viewer>Application&EventService >Microsoft>Intune>CertificateConnector，要admin和operational两份



# Co-management注册不成功

我们先要找用户确认是注册未触发导致的失败，还是说是触发了但是enroll这个过程失败了。在更深入的检查之前，我们也可以先向用户确认一下prerequisites是否都满足，比如说configuration manager的版本是否支持，用户是否有EMS的license，如果是SCCM to Intune的话，那么device是否是hybrid aad joined的，如果是hybrid joined的话，再看看是否是pending status，如果是pending的话也是会导致注册失败的。MDM User scope如果不是All的话，也建议用户先开成All看看是否能成功。还有就是device restrictions这里也可以确认是否设限。

在确认完这些前提条件后，如果说是注册未触发的话，我们可以先去SCCM Console上看‘Enablement‘ tab里automatic enrollment in intune这一栏，是否有选择All或者是pilot，如果选择的是pilot的话，那么想要注册的device是否在pilot组里面。确认完之后，我们再去SCCM client去看configuration manager properties，我们可以先看configuration tab是否有一条autoenroll的policy，如果没有的话，也可以进一步确认是由于未触发导致的失败，这时候我们可以去旁边的actions tab里找到policy retrieval and evaluation cycle，然后点run now，看看点完之后policy有没有刷新出来。如果还是没有的话，我们去windows ccm里看handler log，因为有时候他并不是立即触发enrollment的，他会有一个up to 5 days的randomized time，对于老一点的版本，我们可以搜关键词queuing enrollment timer来看他具体是schedule到什么时候enroll的，如果是新一点的版本的话就是搜schedule time。还有一种情况可能是要等到下一个user log in了他才能触发enroll，这时候就只需要重启一下设备就能触发了。如果这些都做完之后还是没用的话，我们就可能需要找SCCM的同事去寻求帮助。

另外一种情况就是注册触发了但是失败了，这时候我们就可以去SCCM Client看configuration manager properties，比如说version低于5.00.9058的话肯定是会出问题的。我们也可以去configurations tab看auto enroll的这条policy，这里注册失败的话大概率会显示non-compliant的，我们可以尝试点evaluate去重新手动触发一次，如果重新触发还是失败了的话，我们可以去点旁边的view report，这里可能能给到我们一些有用的error code或者error description。如果说SCCM这边没问题的话，我们可以让用户去看provider-admin event logs，失败了的话就是看Event ID 76，看看具体报错的信息是什么来锁定原因。

# tools and logs
## DFM

 - 接case和客户联系用
 - 	DFM写第一封邮件
		自我介绍
		对这个case的理解
		需要收集的信息
附上outlook邮箱
 - 	电话号码区号
		+91 印度（印度有team负责）
		+1美国
		+855香港
+61澳大利亚

## kusto
会列出所有的policy和app 的状态（除win32和store（new））
Win32和store（new）的状态会在底下的intune management extension中显示出来
Device compliance的状态和原因
所有内容都只保持30天

## AC
搜索case id看一些基础的状态和成员
 - 	user、group的成员，Device ID是AAD 的device ID
		Is compliant
		Is managed
		Profile type
 - AAD中的audit log
 - Sign-in log

## Assist 365
 Device，搜索intune device id或者用户UPN
enrollment
MAM，user，app有没有安装，最后一次同步的时间

## powerlift
用companyportal的incident ID去搜
MAM log
MDM log

## logs
### MDM log
portal上device直接collect diagnostics(cooperate owned才会有)
### procmon
设备上所有的在运行程序的所有操作记录下来
### ODC
intune大礼包log。产品组喜欢用，跑命令安装脚本收日志
### syncml
intune策略到设备上，设备接收到的信息，收到的value返回的value，return 200成功，500失败（需要sync）
### setup
所有windows层面的日志（Windows update、intune enroll、SCCM等log）
### remote action
在portal上进行一个remote action操作之后，按F12会看到有一个API生成，向设备端发送远程指令。
移动设备平台，首先看portal上的指令到底有没有下发到设备上，可以在kusto中查看，如果说接收到了这样的一条指令但是仍然没有生效，那么就可以去收设备上的log，安卓debug log， iOS console log
Windows，可以看event viewer中的provider-admin，搜索远程操作的名称
PushNotification-platform
### MDM clean up

 - 在task scheduled中microsoft>windows>enterpriseMgmt，找到enrollment ID，删除enrollment ID 下所有的task，并且把这个enrollment ID 的文件夹删除
 - 在registry editor中用enrollment ID搜索删除键值
 路径是HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Enrollments
 ### Console&x-log
 iOS设备，在无法收集company portal log时，可以连接数据线到mac上在mac上打开console，action中勾选收集log
 ### F12 log
 intune portal上的报告或显示问题，尝试更换浏览器
 ### MHS（managed home screen）
 android dedicated在展台模式下侧拉多次出现log

github原创内容
