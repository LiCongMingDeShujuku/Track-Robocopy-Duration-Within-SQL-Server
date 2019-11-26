![CLEVER DATA GIT REPO](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/0-clever-data-github.png "李聪明的数据库")

# 跟踪SQL Server中的Robocopy的持续时间
#### Track Robocopy Duration Within SQL Server
**发布-日期: 2015年11月27日 (评论)**

![#](images/##############?raw=true "#")

## Contents

- [中文](#中文)
- [English](#English)
- [SQL Logic](#Logic)
- [Build Info](#Build-Info)
- [Author](#Author)
- [License](#License) 


## 中文

这是一些会跟踪Robocopy持续时间的SQL逻辑。我为ETL进程创建了这个，它基本上采用本地数据库备份并将其复制到另一个数据库服务器。然后，我在临时表中收集文件副本的持续时间的报告。



## English
Here’s some SQL logic that will track Robocopy Durations. I created this for an ETL process that basically takes a local database backup and copies it to another database server. I then collect the duration in a temp table for reporting of the file copy.

---
## Logic
```SQL
use master;
set nocount on
 
if object_id('tempdb..#robocopy_duration') is not null
    drop table #robocopy_duration
 
create table #robocopy_duration
(
    time_start      datetime
,   time_finish     datetime
,   notes           varchar(max)
)
 
declare @time_start datetime = ( select getdate())
insert into #robocopy_duration ([time_start])
values (@time_start)
 
exec master..xp_cmdshell 'ROBOCOPY "E:\LOAD_ETLS_SOURCE_BACKUPS"  "\\MyDestinationServer\E$\LOAD_ETLS_SOURCE_BACKUPS" LOAD_ETLS_MyDatabase_01.BAK /ETA /Z /XO /R:2 /W:3 /IS /B /COPYALL /NP /LOG:"E:\LOAD_ETLS_SOURCE_BACKUPS\robocopy_log_for_LOAD_ETLS_MyDatabase_01.log"'
exec master..xp_cmdshell 'ROBOCOPY "E:\LOAD_ETLS_SOURCE_BACKUPS"  "\\MyDestinationServer\E$\LOAD_ETLS_SOURCE_BACKUPS" LOAD_ETLS_MyDatabase_02.BAK /ETA /Z /XO /R:2 /W:3 /IS /B /COPYALL /NP /LOG:"E:\LOAD_ETLS_SOURCE_BACKUPS\robocopy_log_for_LOAD_ETLS_MyDatabase_02.log"'
 
declare @begin_time     datetime = ( select max([time_start]) from #robocopy_duration )
declare @time_finish    datetime = ( select getdate() )
update  #robocopy_duration
set     [time_finish]   = @time_finish where [time_start] = @begin_time
 
declare         @get_robocopy_log   table (robocopy_output varchar(max))
insert into     @get_robocopy_log select * from openrowset(bulk N'e:\load_etls_source_backups\robocopy_log_for_load_etls_MyDatabase_01.log', single_blob) as grl
 
update #robocopy_duration
    set notes = ( select robocopy_output from @get_robocopy_log )
    where [time_finish] = @time_finish
 
select
    'time_start'    = left([time_start], 19) + ' ' + datename(dw, [time_start])
,   'time_finish'   = left([time_finish], 19) + ' ' + datename(dw, [time_finish])
,   'duration'      =
         cast(datediff(second, [time_start], [time_finish])/60/60%24 as nvarchar(50)) + ' hr ' +
         cast(datediff(second, [time_start], [time_finish])/60%60 as nvarchar(50)) + ' mn ' + 
         cast(datediff(second, [time_start], [time_finish])%60 as nvarchar(50)) + 's'
,   'notes'         = notes
from
    #robocopy_duration
order by
    [time_start] asc


```
Of course you could always query the Log output file directly from within SQL Server with this:

当然，你始终可以直接从SQL Server中查询日志输出文件：

```SQL
declare         @get_robocopy_log   table (robocopy_output varchar(max))
insert into     @get_robocopy_log select * from openrowset(bulk N'e:\load_etls_source_backups\robocopy_log_for_load_etls_MyDatabase_01.log', single_blob) as the_robocopy_log
 
select robocopy_output from @get_robocopy_log

```
This could be edited further using bulk insert and the appropriate switches to collect only the text you want during the import.

这里可以使用批量插入和相应的开关进一步编辑，并仅在导入期间收集所需的文本。


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

- **李聪明的数据库 Lee's Clever Data**
- **Mike的数据库宝典 Mikes Database Collection**
- **李聪明的数据库** "Lee Songming"

[![Gist](https://img.shields.io/badge/Gist-李聪明的数据库-<COLOR>.svg)](https://gist.github.com/congmingshuju)
[![Twitter](https://img.shields.io/badge/Twitter-mike的数据库宝典-<COLOR>.svg)](https://twitter.com/mikesdatawork?lang=en)
[![Wordpress](https://img.shields.io/badge/Wordpress-mike的数据库宝典-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

---
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Lee Songming](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/1-clever-data-github.png "李聪明的数据库")

