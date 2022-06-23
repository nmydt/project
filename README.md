# 数据可视化课设
1，动漫评分数据分析与可视化

[可视化地址预览](https://nmydt.gitee.io/project/cartoon/html/)

2，IT行业招聘数据分析与可视化

[可视化地址预览](https://nmydt.gitee.io/project/IT/html/)

## 一 动漫评分数据分析与可视化

### 1.1 数据抓取
[BilibiliSpider](cartoon/BilibiliSpider.ipynb)

将抓取文件上传到${HIVE_HOME}/mydata目录下

数据格式为：[{"ssid": "41410", "cartoon": "\u95f4\u8c0d\u8fc7\u5bb6\u5bb6", "views": 63330785, "coins": 622944, "follow": 6570653, "series_follow": 6571233, "danmakus": 280498, "likes": 1442697, "favorite": 177344, "favorites": 6570661, "reply": 138687},{...}]
### 1.2 Hive表创建与导入
[Hive表字段信息](HiveTableInfo.txt)

#### 1.2.1 创建cartoon_info表并导入数据

```sql
CREATE EXTERNAL TABLE Json(
 data string
)
```
加载数据到Json表中备用
```sql
load data local inpath 'mydata/infos_total.json' overwrite into table Json;
```
创建cartoon_info表
```sql
drop table if exists cartoon_info;
CREATE EXTERNAL TABLE cartoon_info(
`ssid` string,
`cartoon` string,
`views` bigint,
`coins` int,
`follow` int,
`series_follow` int,
`danmakus` int,
`likes` int,
`favorite` int,
`favorites` int,
`reply` int,
`share` int,
`cover` string,
`url` string,
`episodes` int,
`count` int,
`is_finish` int,
`pub_time` TIMESTAMP,
`media_tags` string,
`voice_actor` string,
`score` float
)
stored as parquet
location '/warehouse/cartoon_info';
```
使用Json解析插入数据，详情请看: [Hive之Json解析(普通Json和Json数组)](https://blog.csdn.net/a12355556/article/details/124565275)
```sql
insert overwrite table cartoon_info

select json_tuple(json,'ssid' ,'cartoon' ,'views' ,'coins' ,'follow' ,'series_follow' ,'danmakus' ,'likes' ,'favorite' ,'favorites' ,'reply' ,'share' ,'cover' ,'url','episodes' ,'count' ,'is_finish' ,'pub_time','media_tags','voice_actor','score') from (
select explode(split(regexp_replace(regexp_replace(data,'\\[|\\]',''),'\\}\\, \\{','\\}\\;\\{' )  ,'\\;'))  as json from Json
)a;
```
#### 1.2.2 创建cartoon_comments表
```sql
CREATE EXTERNAL TABLE Json2(
 data string
);
```
加载数据到Json2表中备用
```sql
load data local inpath 'mydata/comments_total.json' overwrite into table Json2;
```
创建cartoon_comments表并导入数据
```sql
drop table if exists cartoon_comments;
CREATE EXTERNAL TABLE cartoon_comments(
`mid` string,
`uname` string,
`ssid` string,
`message` string,
`like` int,
`dt` timestamp
)
stored as parquet
location '/warehouse/cartoon_comments';
```
使用Json解析插入数据，详情请看: [Hive之Json解析(普通Json和Json数组)](https://blog.csdn.net/a12355556/article/details/124565275)
```sql
insert overwrite table cartoon_comments

select json_tuple(json,'mid' ,'uname' ,'ssid' ,'message' ,'like' ,'dt' ) from (select explode(split(regexp_replace(regexp_replace(data,'\\[|\\]',''),'\\}\\, \\{','\\}\\;\\;\\;\\{' )  ,'\\;\\;\\;')) as json from Json2)a;
```
## 二 IT行业招聘数据分析与可视化

### 1.1 数据抓取

[ITJobSpider](IT/ITJobSpider.ipynb)

`1，需要登录拉勾网！！请注意替换个人Cookie且Cookie中不要有中文，否则会报错；如果Cookie不生效，请打开拉勾网其他页面获取Cookie.`

`2，若报错请打开拉勾网查看是否需要验证`

将抓取文件上传到${HIVE_HOME}/mydata目录下

### 2.1 Hive表创建与导入

[Hive表字段信息](src/HiveTableInfo.txt)

```sql
CREATE EXTERNAL TABLE Json3(
 data string
)
```
加载数据到Json3表中备用
```sql
load data local inpath 'mydata/jobsInfo.json' overwrite into table Json3;
```
#### 2.1.1 创建jobs_info表并导入数据
```sql
drop table if exists jobs_info;
CREATE EXTERNAL TABLE jobs_info(
`job` string,
`keyword` string,
`place` string,
`requirement` string,
`salary` string,
`tags` string,
`welfare` string,
`pubtime` date
)
stored as parquet
location '/warehouse/jobs_info';
```
使用Json解析插入数据，详情请看: [Hive之Json解析(普通Json和Json数组)](https://blog.csdn.net/a12355556/article/details/124565275)
```sql
insert overwrite table jobs_info

select json_tuple(json,'job' ,'keyword' ,'place' ,'requirement' ,'salary' ,'tags' ,'welfare' ,'pubtime') from (
select explode(split(regexp_replace(regexp_replace(data,'\\[|\\]',''),'\\}\\, \\{','\\}\\;\\{' )  ,'\\;'))  as json from Json3
)a;
```
## 3，数据分析与可视化

### 3.1 Pyhive连接Hive教程:

[Python安装sasl,thrift,thrift-sasl 并连接PyHive](https://blog.csdn.net/a12355556/article/details/124580555)

连接代码: [Pyhive](pyhive.ipynb)

### 3.2 数据分析与可视化

安装必要的包
```shell
pip install pandas==0.23.4
pip install pyecharts==1.9.1
pip install matplotlib==3.5.1
pip install numpy==1.18.5
pip install jieba==0.42.1
pip install squarify==0.4.3
```

1，动漫评分数据分析与可视化
数据分析代码：[bilibili](cartoon/bilibili.ipynb)

代码包含了["玫瑰图","词云图","雷达图","散点图","漏斗图","环图","条形图","树形图","火柴杆图","子图"]共10个类型的图，包含了4个matplotlib图以及6个pyecharts图的简单分析。

2，IT行业招聘数据分析与可视化
数据分析代码：[IT](IT/IT.ipynb)

代码包含了["玫瑰图","词云图","象形图","散点图","漏斗图","环图","条形图","树形图","火柴杆图","子图"]共10个类型的图，包含了4个matplotlib图以及6个pyecharts图的简单分析。
