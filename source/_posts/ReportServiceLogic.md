---
title: 上报服务代码逻辑
date: 2019-05-20 10:49:47
tags:
password: Shui
abstract: 上报服务代码逻辑
---


# ProvinceReport工程说明
该文档建议使用支持MarkDown的文档编辑器或装有MarkDown插件的浏览器打开

## 目录

1. 工程概述  
1.1 SVN  
1.2 MinistryReportService  
1.3 RealEstateQueueService  
1.4 RealEstateReportService  
1.5 RealEstateReport  
1.6 RealEstateStorage  
1.7 ReportResponseService  
1.8 上报工具  
2. 项目说明  
2.1 [MinistryReportService](#MiniInfo)  
2.2 [RealEstateQueueService](#QueueInfo)  
2.3 [RealEstateReport](#RealInfo)  
2.4 [RealEstateReportService](#ReportInfo)  
3. [部署](#Test)


## 工程概述

### SVN
内网地址
- http://192.168.27.71:8191/svn/ProvinceReport/ProvinceReport

外网地址：
- http://61.183.129.187:8191/svn/ProvinceReport

### 1.MinistryReportService 
不动产省厅报部服务，主要用于湖南省厅从省级数据库中获取可上报数据，生成xml报文，提交给部里接收端。
>- 四川绵阳市，新疆昌吉市，湖南邵阳市，将省级服务部署在市级用于接收县里报文，同时上报给省级接收端

### 2.RealEstateQueueService
不动产省级接收端服务，用于湖南省厅接收市县报文，并对报文进行解析，校验，入库，生成并反馈响应报文等操作。

### 3.RealEstateReportService
不动产市县上报服务，用于现有不动产统一登记系统所在项目，对登记数据进行抽取和上报，提交报文给部里接收端或其他省市级报文接收端。

### 4.RealEstateReport
不动产市县上报主动态库，用于不动产统一登记系统和`RealEstateReportService`服务中。
* 在不动产登记系统中，用作上报相关页面的控制器，响应页面请求和进行其他逻辑处理。
* `RealEstateReportService`内，完成数据抓取，报文组织，签名，校验，上报等任务。
* 该库同时被`RealEstateQueueService` 和 `MinistryReportService` 服务引用，以使用其部分通用功能。

### 5.RealEstatePlatform
省级平台相关工程，工程内 `SJBDCPlatformTransfer `项目在用，其他项目已经不再维护。`SJBDCPlatformTransfer`为类库，用于`MinistryReportService `服务和不动产省级平台（`RealEstateStatisticsPro`项目）。

### 6.RealEstateStorage
原不动产上报系统 WCF 服务的主动态库，后来用 SFTP 替代 WCF + MSMQ 消息队列后不再维护。
该项目不再维护。

### 7.ReportResponseService
原不动产上报响应报文接收服务，后来将接收功能作为线程，增加到RealEstateReportService中。
该项目不再维护。

### 上报工具
此部分的工具都只是为了解决某一次的需求设计出来的。也许你永远也用不到里面的东西

#### 1.DeleteOldData
删除历史数据

#### 2.DisposeSavedRepXml
手动处理响应报文，这个是湖南某个地区只能下载响应报文，无法处理的情况下做的，后来又通过上报服务把功能修复了。但是这个项目也没有删除。

#### 3.DYAQDataCollect
某一次省厅需要统计各市县的抵押权相关信息，而做出来的工具。

#### 4.GetRepMsg
手动获取响应报文并入库。

#### 5.MonitorNetstatTable
监测Report_NetStat表是否存在缺省值。

#### 6.ReportCommonFunction
通用方法

#### 7.ReportDataBaseModel
通用模型

#### 8.ZhongZhiZJK
推送中智的中间库，每次省厅获取数据后，要把数据推送到中智的数据库中。采用Oracle的DBMS_ChangeNotification 包。

## 项目说明

<span id="MiniInfo"/>

### MinistryReportService
**要点一**
```
<!--上报地区 0：报SFTP；1：报湖南省厅WCF；2：WebService；3：HTTP Request；-->
<add key="ReportAreaCode" value="0" />
```
服务根据配置参数`ReportAreaCode` 选择启动对应的省级上报线程，代码段如下。


```C# 
/*
项目：MinistryReportService
类文件：MinistryReportService.cs
行号：47
*/
protected override void OnStart(string[] args)
        {
            // TODO: 在此处添加代码以启动服务。
            var reportAreaCode = GetConfigString(ConfigurationManager.AppSettings["ReportAreaCode"]);
            switch (reportAreaCode)
            {
                case "1":					// WCF 方式已经弃用
                    _thdReport = new Thread(UploadRealEstateDataForHNWCF);
                    _thdReport.Start();
                    break;
                case "2":					//目前应该没有地方在用这个
                    _thdReport = new Thread(UploadRealEstateDataMultiThread_WS);
                    _thdReport.Start();
                    break;
                case "3":					//四川绵阳市部署的省级上报服务在使用 上报到四川省厅。
                    _thdReport = new Thread(UploadRealEstateDataMultiThread_HTTP);
                    _thdReport.Start();
                    break;
                default:					//湖南省厅报部的方式，分为如下 4 个线程
                    _thdReport = new Thread(UploadRealEstateDataMultiThread_SFTP);		//完成报文生成，并写入服务器本地报文缓存目录
                    _BizMsgToRemote = new Thread(LocalBizMsgToRemote);		//扫描本地报文缓存目录，并将报文发送至部里 SFTP 接收端
                    _thdResponse = new Thread(GetRepMsgFrmSftp);		//扫描 SFTP 前置机，下载响应本地至本地响应报文缓存目录
                    _RepMsgToDB = new Thread(UpdateRepMsgToDB);		//扫描本地响应报文缓存目录，将响应报文解析入库
                    _thdReport.Start();
                    _BizMsgToRemote.Start();
                    _thdResponse.Start();
                    _RepMsgToDB.Start();
                    break;
            }
        }
```

**要点二**
```
<!--不动产登簿与数据接入日志上报配置 value=1，开启运行日志-->
<add key="OpenDebugLog" value="1"/>
<!--日志记录方式，默认txt，1：数据库-->
<add key="StoreLogDataBase" value="1"/>　
　　
```
开启服务运行日志后，默认将日志保存在本地文件中，可以通过设置	`StoreLogDataBase` 的值将日志保存在数据库中，便于查询，服务每天自动删除7天前的日志数据，代码位置如下。
``` C#
/*
项目：RealEstateQueueService
类文件：RealEstateQueueService.cs
行号：146
*/
if (CommonFunction.GetAppConfigSetting("StoreLogDataBase") == "1" && DateTime.Now.Hour.ToString() == "12")
   {
       Task.Factory.StartNew(() =>
       {
            var iDb = DbAccessFactory.CreateInstance();
            iDb.ExecuteSql("delete from queuesvcdebuglog where time < sysdate - 7");
        });
    }

```

**要点三** 
装载可上报行政区的集合 `List<EstateDataTransfer.AreaReady>()`，并将集合作为参数传入线程池，用于给空闲线程派发任务。
```C#
/*
项目：MinistryReportService
类文件：MinistryReportService.cs
行号：680
*/
var iDb = DbAccessFactory.CreateInstance(strConnectString, strDataType);
            var strSql = @"select areacode from sysareacode where flag = '1'";
            var strAreacode = iDb.GetFirstColumn(strSql).ToString();

            strSql = string.Format(@"select distinct  t.areacode  from sysareacode t where t.areacode 
            like '{0}____' and t.areacode not like '____00'", strAreacode.Substring(0, 2));

            dataSet = iDb.GetDataSet(strSql);
            var areaList = new List<EstateDataTransfer.AreaReady>();
            areaList.AddRange(from DataRow dr in dataSet.Tables[0].Rows
                              select new EstateDataTransfer.AreaReady
                              {
                                  Areacode = dr[0].ToString(),
                                  Isready = true
                              });


            
```
线程池只能接收 object 对象作为参数，这里用自定义参数类 `TaskInfo`
```c#
/*
项目：MinistryReportService
类文件：MinistryReportService.cs
行号：709
*/
var o = new TaskInfo(strConnectString, strDataType, sftpHost, sftpUserName, sftpUserPwd, sftpUpRemotePath, areaList);
```
服务内使用线程池 `ThreadPool.QueueUserWorkItem` 来实现多线程上报，ThreadPool类在微软官方文档上是已经被弃用的，建议改用Task类来实现。
```C#
/*
项目：MinistryReportService
类文件：MinistryReportService.cs
行号：710
*/
while (true)
      {
         var hnReportStartTime = Convert.ToInt32(ConfigurationManager.AppSettings["HNReportStartTime"]);
         var hnReportEndTime = Convert.ToInt32(ConfigurationManager.AppSettings["HNReportEndTime"]);
         var allTimeRun = Convert.ToInt32(ConfigurationManager.AppSettings["AllTimeRun"]);
	     var isTimeLegal = CheckTime(hnReportStartTime, hnReportEndTime);
         if (isTimeLegal || allTimeRun == 1)
         {
		       EstateDataTransfer.AreaReady ar = new EstateDataTransfer.AreaReady();
	           if (!ar.AnyAreaAvailable(areaList))
               {
                    WriteLog("no AnyAreaAvailable,sleep 10s");
                    Thread.Sleep(10000);
                    continue;
               }
               if (_stopThreadPool || maxCadaScanThreadNum <= _currentCadaScanTreadNum) continue;
               var callback = new WaitCallback(ScanAndSave_SFTP);

               ThreadPool.SetMaxThreads(maxCadaScanThreadNum, maxCadaScanThreadNum);
               ThreadPool.SetMinThreads(minCadaScanThreadNum, minCadaScanThreadNum);
               ThreadPool.QueueUserWorkItem(callback, o);
               Interlocked.Increment(ref _currentCadaScanTreadNum);
          }
          else
          {
               WriteLog("非上报时间区间，暂停十分钟");
               Thread.Sleep(600000);//不在正常上报时间内，暂停十分钟
          }
	}
```

**要点四**
SFTP连接方法使用的开源库 WinSCP ，建议去官方主页了解该库的使用方法。
>- 其中 FileMask 在文件数量过大的时候可能会导致 SFTP 连接超时，当时是 SFTP 中同一目录下报文数量大于 10w 出现了这个情况。
>- WinSCP 的 FileMask 、递归复制、断点续传特性都非常有用。
>- SSH 连接用完后要手动关闭，或者实现IDisposable接口来释放，不然会导致 sftp 服务器连接过载导致连接出错

<span id="QueueInfo"/>

### RealEstateQueueService
**要点一**  
`RealEstateQueueService` 同 `MinistryReportService` 一样，用多个线程任务共同完成报文接收和反馈。
```C#
/*
项目：RealEstateQueueService
类文件：RealEstateQueueService.cs
行号：36
*/
protected override void OnStart(string[] args)
        {
            msgLogFunc.OnStart();	//登记簿日志接收
            // TODO: 在此处添加代码以启动服务。
            if (CommonFunction.GetAppConfigSetting("queueType", "0") == "1")
            {
                _thdMain = new Thread(new ThreadStart(CadaScan_SFTP));	//从SFTP下载报文到QueueService服务本地
                _thdPersistence = new Thread(new ThreadStart(Persistence_SFTP));	//从QueueService服务本地解析入库，并生成本地响应报文
                _thdRepMinister = new Thread(new ThreadStart(RepScanMinister_SFTP));	//从MinisterReportService服务返回响应报文到SFTP 市县RepMsg目录
                _thdRepPutBackToCounty = new Thread(new ThreadStart(RepPutBackToCounty));	//从QueueService服务返回响应报文到SFTP 市县RepMsg目录
            }
            else
            {
                _thdMain = new Thread(new ThreadStart(CadaScan));
            }
        }
```

**要点二**  
`DownloadFromSftpQueue`方法中下载市县上报报文时先用 `Tamir.SharpSsh.jsch` 动态库查询了sftp前置机中data/sftp/路径下的各个公司的家目录的名称(zondy01,zhongzhi01等），然后通过`WinScp`的文件夹批量传输进行了报文下载。
```C#
/*
项目：RealEstateQueueService
类文件：QueueSftpProcess.cs
行号：128
*/
//遍历公司家目录，行号：155
foreach (Tamir.SharpSsh.jsch.ChannelSftp.LsEntry item1 in items1)
                {
                    string name1 = item1.getFilename();
                    if (name1 == "." || name1 == "..")//公司名称
                    {
                        continue;
                    }
//下载报文，行号：180
					#region sftp 下载操作
                    sftp.LongOpen();
                    var result = sftp.Get(remotePath, localPath, fileMask, true);
                    sftp.Close();
                    #endregion
```

**要点三**  
到官网学习WinScp的使用方法，了解其`FileMask`、`TransferResumeSupportState`等特性;
虽然在`RealEstateReport.WinScpClient`类中有`LongOpen`方法，试图重复使用打开的sftp会话，但是实际使用过程中会导致问题，要在用完后手动调用`Close`方法关闭session。

**要点四**  
注意`ExcludeTableKey`类，其目的是在QueueService服务多线程批量处理同一批报文时，对于不同报文间出现的相同表相同主键的数据举行手动过滤，以此避免多线程同时入库时将同表同主键数据同时判断为insert导致数据库插入错误。
原则是将同一批入库的数据中同表同主键数据视作相同的数据，无须重复入库。

**要点五**  
`BatDownFile`方法是QueueService服务数据入库和逻辑维护的主要方法。该方法需要做如下事情：
1、维护QueueList表；
2、读取，解析，校验xml报文，检查数据主键字段值是否为空，排除数据主键冲突，并将报文数据写入数据库
3、生成响应报文，并保存到服务器本地
4、维护Reportoffset表
5、维护ReportStatus表
>- ps: QueueList表主键是消息ID，ReportOffset主键是BizMsgID，ReportStatus表主键是BizMsgID

**要点六**  
`RepScanMinister_SFTP`线程是从报部服务获取部里的响应报文发送给市县的。其逻辑为从transferstate表中找出已经得到部里响应的报文ID和响应报文存放路径，并从ReportStatus表中找到该条数据对应的公司家目录名称(downpath)，将部里的响应报文返回给市县。

**要点七**  
市县校验使用`Schema校验`，使用XSD文件来校验报文的属性是否存在、元素节点是否存在等等。  
省厅比市县多了`Schematron校验`，使用.sch文件，内部使用Schematron文件规范，用来进行计算属性的值是否满足某种规则。    
```C#
项目：NMatrix.Schematron
说明：这是采用Schematron1.5标准的项目工程，没有找到1.6版本的。1.6版本自带了变量的定义，更为方便，因此修改了1.5标准的工程，加了自定义变量的功能。

```
```C#
文件：SchemaLoader.cs
行号：196
foreach (var i in asserts.Current.GetAttribute("vali", string.Empty).ToString().Split(','))
{
	vali.Add(i);
}
//此处将自定义的vali变量代表的属性的名称添加到对象中


文件：SyncEvaluationContext.cs
行号：235

foreach (var i in assert._vali)
{
	var a = context.GetAttribute(i, context.NamespaceURI);
	expr = !string.IsNullOrEmpty(a) ? expr.Replace(string.Format(@"va-{0}-va", i), a) : expr;
	mess = !string.IsNullOrEmpty(a) ? mess.Replace(string.Format(@"va-{0}-va", i), a) : mess;
}
//此处将自定义变量的值查询到，然后替换文件中的va-{}-va字符串为具体的值

//<assert ... vali='BDCDYH' test='va-BDCDYH-va'>
```
<span id="RealInfo" />

### RealEstateReport
**要点一**
这个工程生成RealEstateReport.dll，该动态库在市县不动产Web，市县ReportService，省级QueueService，省级MinistryReportService服务中均有引用。可以把这个工程当作是站点系统，同时被其他工程引用了类库。

**要点二**  
上报系统有部分嵌入到了不动产系统中，需要在不动产登簿的时候，调用上报的方法
> SendRealEstateStorage  
> SendRealEstateStorageSPF  

，更新SEQ表，这样服务才可以上报内容。  
同时还有页面嵌入到了不动产站点中，ReportManage，上报管理页面，用于查询上报的状态，信息，重新编号，重新上报之类的，重新编号是指重新生成报文ID，然后重新上报。  
另外还为上报服务设计了配置修改页面，可以在页面上修改**上报服务**的配置，仅限于上报服务，ReportConfigManage页面。同时还有一套前端设计的修改配置页面，站点是27.11的F：/站点/上报/dist。如果不理解前端项目如何开发，可以只使用嵌入不动产系统中的，前端系统是没有放到SVN中的。

**要点三**

<span id="ReportInfo" />

### RealEstateReportService
**要点一**  
本工程是各市县上报业务数据，上报登簿数据，接收业务响应报文，以及上报其他信息的功能。其他支线功能有：枝江上报流程报文、上报状态信息、补报报文功能。
```C#
文件：RealEstateReportService.cs 51行

			//上报登簿日志
            _MsgLogFunc.OnStart();
            //上报业务数据
            if (runRealEsatateReport == "1")
            {
                WriterLog("AutoRealEstateReport start");
                _thdReport = new Thread(AutoRealEstateReport);
                _thdReport.Start();
            }
            //接收响应报文
            if (StrResponseInThis == "1")
            {
                WriterLog("ReportResponseFunc start");
                _responseFunc = new ReportResponseFunc();
                _responseFunc.OnStart();
            }
            //郑州状态报文
            if (runRealEsatateReport == "1" && RecMethod == "1" && RecInfo == "1")//WebService && 郑州
            {
                WriterLog("AutoZhengZhouStatus start");
                _thdZzStatus = new Thread(AutoZhengZhouStatus);
                _thdZzStatus.Start();
            }
            //枝江流程报文
            if (runRealEsatateReport == "1" && ExtZhijiang == "1")
            {
                WriterLog("ProBizReport start");
                _thdProReport = new Thread(ProBizReport);
                _thdProReport.Start();
            }
			//湖南各市县上报状态EXCEL,用于统计信息
            if (ReportStatusExcel == "1")
            {
                WriterLog("ReportStatusExcel start");
                _thReportStatusExcel = new Thread(ReportStatus);
                _thReportStatusExcel.Start();
            }
			//存在这种登簿后但是没有上报的业务
            //处理登簿未上报的业务，更改日志表的状态，让上报线程去报，这里只是改状态
            if (runReportRegisData == "1" || runReportRegisData == "2")
            {
                WriterLog("ReportRegisData start");
                _thReportRegisData = new Thread(ReportRegisData);
                _thReportRegisData.Start();
            }
```

**要点二**  

```AutoRealEstateReport``` 作为上报业务数据的线程，内部的功能逻辑按照以下的流程来实现：  

1. 查找SEQ_REPORT找可以上报的业务
> ReportAble：是否可以上报  
> Flag：当前数据是否有效  

2. 确定组装、匹配规则
> ReportPracticeDict：国标的业务与不动产业务的对应，国标业务需要组装什么表  
> ReportTable：国标规则的表，有哪些字段，关键字是什么  

3. 组装数据  
4. 根据不同的发送方式发送
> SFTP 湖南地区基本只用这个      
> WebService  
> HTTP  
> FTP  

**要点三**  

上报服务针对衡阳地区的分库做了适应功能，主要是对iDB的获取与设置。分库后，主库中```DBS_INFO```存储了分库的信息，然后设计了```iDBHelper```类来进行iDB的更新与获取。  
```C#
dCreater//主库加分库的IDB记录器
dIndicator//指示器，记录哪一个线程目前采用的是哪一个分库
```
在分库的版本中，确保所有分库都经历过上报行为。

> 一个主库，对应多个分库，通过DBS_INFO表记录分库的信息

分库的关键做法是：  
1. 设置iDbHelper，在初始化的时候加载分库信息到对应对象。  
2. 通过单例开发模式，在线程运行中，对iDbHelper进行遍历，进行数据上报、接受等。  
3. 在每次循环完成之后，更新iDbHelper对应线程的标识符，让iDbHelper下一次返回的是下一个分库的信息   
4. 所有循环完成后，重置对应线程标识符。

**要点四**  

```ReportResponseFunc``` 是获取响应报文的线程，可能有的地区现在还部署了很久之前的Response服务，以前是作为服务存在的，现在更改为了作为线程存在。

接收响应注意事项：  
1. 如果是```SFTP下载``` ，则会首先根据配置中的信息，将涉及的所有行政区开头的报文都下载下来。'所以一定要非常注意，避免某地区下载了不是它的报文'，当初是发生了报文积压，有十万单位的报文积压在这里，导致每个地方下载都很慢。所以就先下载后处理。  
2. 可能有的区县代码代表两个地区，这种地区则根据数据库中的报文ID和前置机的报文名称列表进行匹配，只下载数据库中有的。

**要点五**  

登簿日志上报，登簿日志记录了每天的登簿业务数量，具体内容参考**《登簿日志上报方案.pdf》**。<span style='color:red'>省厅也有需要往部里报登簿日志的要求，所以省厅也会部署一个上报服务ReportService，但是会关闭所有开关，只打开登簿日志上报的开关。</span>
```C#
数据库：MiniAccessReportProc存储过程，每天方法调用这个存储过程来获取数据
```

<span id="ZJKInfo" />

### ZhongZhiZJK
**要点一**  
推送中智中间库，目前推送的思路是采用`DBMS_ChangeNotification` 	包，记录发生变化的数据，然后通过iDB实时更新。
> Dbms_ChangeNotification包是Oracle的基础包  
> 具体的使用教程在上报代码项目/ZhongZhiSJZJK/Readme/DBMS.CHANGE.NOTIFICATION.sql中  

**要点二**  
通过将发生变化（增删改）的数据记录到nfrowchanges表，记录了操作类型table_operation、表名table_name、唯一标志row_id，然后通过rowid找到具体的数据。  
因为是通过rowid改变的，所以要注意！！！<span style='color:red'>如果涉及到备份还原、迁库之类的行为，Oracle不会存储原来的Rowid，而是会产生新的</span>  ，所以再每次迁库或其他可能造成rowid变化的行为之前，要保证现有的nfrowchanges都有变化都已经推送完毕。

<span id="Install" />

## 部署
部署一套上报系统，需要以下内容：  
1. 不动产站点（用于模拟各地登簿、历史上报）  
2. 市县上报服务ReportService  
3. 省厅入库服务QueueService  
4. 省厅报部服务MinistryService  
5. 不动产数据库  
6. 省厅数据库  
7. （可选）Linux前置机，大部分情况下，可以只模拟到发送存档的过程。  

<span id="Test" />

### 测试

#### 测试上报服务
1. 修改SEQ表的记录，让某一条报文处于可上报状态
> update seq_report set reportable = '1' where ywh = '' and flag = '1' and reportdirection = 'default'  

2. 开启上报服务，观察日志信息

#### 测试入库服务
1. 将报文放到入库服务/data/sftp/mysftp/BizMsg 中  
2. 开启QueueService服务