# prom+grafa primary

## 基础
1.   [亚马逊中译grafana文档](https://docs.aws.amazon.com/zh_cn/grafana/latest/userguide/v9-dash-variable-add.html#v9-dash-variable-add-filter)  
2.   [一篇比较详细的promql表达式文章](https://blog.51cto.com/root/3033762)  
3. [rancher集群检测推荐指标](https://docs.rancher.cn/docs/rancher2/cluster-admin/tools/cluster-monitoring/expression/_index#%E9%9B%86%E7%BE%A4-cpu-%E5%88%A9%E7%94%A8%E7%8E%87)  

4. **Query Editor：** 查询编辑器，用来指定获取哪一部分数据。类似于sql查询语句，但是其中的不同DataSource  获取数据的方式不同，写法便不同。可以选择并自动生成
5. instance -- Prometheus中，每一个暴露监控样本数据的HTTP服务称为一个实例Prometheus中，每一个暴露监控样本数据的HTTP服务称为一个实例
6. job -- 一组用于相同采集目的的实例，或者同一个采集进程的多个副本则通过一个一个任务(Job)进行管理一组用于相同采集目的的实例，或者同一个采集进程的多个副本则通过一个一个任务(Job)进行管理
7. 标签(label)反映了当前样本的特征维度
8. 以`__`作为前缀的标签，是系统保留的关键字，只能在系统内部使用
10. Expression 是一种基于 PromQL 的查询方式，它允许你使用 PromQL 表达式来查询 Prometheus 中的指标，从而创建更复杂和灵活的查询；常用于警报功能。

11. **Metrics数据类型**： 
	1.  counter-- 计数器，累加的指标(推荐使用_total作为后缀Counter类型指标的名称时推荐使用_total作为后缀)
	2.  gauge--仪表盘，直接显示系统当前值,样本数据可增可减；
	3.  Histogram和Summary主用用于统计和分析样本的分布情况;Histogram通过histogram_quantile函数是在服务器端计算的分位数。 而Sumamry的分位数则是直接在客户端计算完成

12. avg （） by （）--  其中的by 可以视作group by
13. rate()--函数，可以计算在单位时间内样本数据的变化情况即增长率
14. delta()--计算样本在一段时间内的变化情况
15. predict_linear()--预测变化）

16. Transform data： 数据转换，数据处理
	 1.  Add field from calculation： 基于两种方式新增字段，通过原有的值进行计算（需要继续实践）
	 2.  Concatenate fields： 将所有字段连接起来组成新字段
	 3.  Config from query results：从查询结果中提取字段，并应用于另外一个查询中
	 4.  Convert field type： 做数据的格式转换
	 5.  Filter data by name： 选择需要展示的字段
	 6. create heatmap： 根据源数据计算热图
	 7.  Extract fields： 分析内容中的字段 (JSON, labels等)
	 8.  Filter by name： 通过正则或指定字段过滤数据
	 9.  Filter data by query： 当一个图表面版有多个query结果展示时，用来选择需要展示的字段
	 10.  Filter data by value： 根据值做筛选，参考Excel筛选器
	 11.  Group by： 此转换按指定的字段(列)值对数据进行分组；之后对每组数据做运算
	 12.  Join by field： 根据相同字段做聚合（表连接）
	 13.  Join by labels： 将带标签的结果展示到由标签连接的表中
	 14.  Grouping to matrix： 将三个字段组合成一个矩阵
	 15.  Merge： 合并多个查询到一起展示
	 16. Histogram： 根据输入数据计算直方图
	 17.  Sort by： 排序
	 18.  Organize fields： 允许用户重新排序、隐藏或重命名字段/列，只能用于单个查询的面板
	 19.  Partition by values：通过一个或多个字段中的唯一/枚举值进行分区
	 20.  Rename by regex： 使用正则表达式和替换模式重命名部分查询结果
	 21.  Reduce： 压缩字段
	 22. ser
  
16.  系统资源监控用得较多的方法是"USE"方法，分别表示为：Utilization（使用率）、Saturation（饱和度）、Errors（错误数 ）
-------------------------------------------------------------------------------
## 使用技巧
1 . 链式变量
```
设置dashboard变量
原则： 变量任意设置，但是都必须通过查询得到，所以需要有相应标签值
      定义和使用时都可以结合正则，利于实现筛选和查询
      需要建立层级和下钻关系，需要书写时建立

label_values(up{origin_prometheus=~"$origin_prometheus"},job)
label_values(up{origin_prometheus=~"$origin_prometheus",job=~".*$department.*"},job)
label_values(up{origin_prometheus=~"$origin_prometheus",job=~"$job"},instance)
```
2 . data link 
```
在某列值或者某个pannel，添加link
复制目标dashboard的url,并编辑URL；主要内容(&var-variables=${__data.fields.instance})----(作用：将点击的instance的值传递给该dashboard的变量)
输入$ 可以提示变量填写
```
------------------------------------------------
###  **系统**
1
![](attachments/Pasted%20image%2020230512172201.png)
2
![](attachments/Pasted%20image%2020230512174628.png)
-----------------------------

## 自定义项目

**NOTE:**
install node_exporter--> systemd 管理 (.service文件)-->ExecStart=   --collector.textfile.directory=/   --> systemctl daemon-reload  && restart node_exporter --> 编写脚本收集指标数据 --> install "sponge" -->crontab -e  -->  * * * * *  | sponge    /.\*/textfile/name.prom --> 使用
sponge 命令来自moreutilst 工具集，需要apt-get  install  *（作用：从标准输入读取数据并追加重定向输出，和管道符配合使用，此命令会避免因为输出过大导致内存和磁盘不足）   

#/bin/bash

---------------------------------------
### original
1 . 文件夹大小--GitHub
```
#!/bin/sh
# Usage: add this to crontab:
#
# */5 * * * *  directory_size.sh /var/lib/prometheus | sponge /var/lib/node_exporter/textfile/directory_size.prom

echo "# HELP node_directory_size_bytes Disk space used by some directories"
echo "# TYPE node_directory_size_bytes gauge"

dir="$1"
for directory in $(tree -dfi $dir | head -n -2 | sed 's/\.//g' ); do
  du --block-size=1 --summarize "$directory" \
    | sed -ne 's/\\/\\\\/;s/"/\\"/g;s/^\([0-9]\+\)\t\(.*\)$/node_directory_size_bytes{directory="\2"} \1/p'
done
```

2 . 监控僵尸进程--process_exporter 
```
#!/bin/bash
echo "# HELP Zombie_nums using for counting the numbers of Zombie proccess"
echo "# TYPE Zombie_nums gauge"
Zombie_nums=`ps -A -o stat,ppid,pid,cmd | grep -e '^[Zz]' |wc -l`
echo "Zombie_nums $Zombie_nums"

```

4 . 文件打开数--process_exporter [运用](https://blog.51cto.com/root/3055821)
```

/proc/sys/fs/file-nr                                              # 文件中的第一个数字表示已分配文件句柄的数量
sudo find /proc -print | grep -P '/proc/\d+/fd/'| wc -l`          # 命令用于计算系统中所有进程打开文件的数量
                                                                  process_exporter 和 find 查找到的数量基本一致 

文件句柄通常是操作系统内核中用于表示打开文件或其他 I/O 设备的结构体或数据结构。它包含了一些有关该文件或设备的信息，例如文件的大小、访问权限、位置指针等，是用于标识打开文件或设备的唯一标识符

文件描述符通常用于表示进程与打开文件之间的关系。文件描述符是一个整数，用于标识进程中打开文件或其他 I/O 设备的抽象概念。在 Linux 和类 Unix 系统中，文件描述符从 0 开始计数（在一些特殊情况下，如网络套接字，文件描述符也可以是负数）。文件描述符还可以用于表示其他类型的 I/O 设备，例如管道、套接字、字符设备、块设备等。

在进程打开文件时，操作系统会为该文件分配一个文件句柄，并返回一个文件描述符供进程使用
```

5 . 端口和进程监控 
```
#!/bin/bash

echo "# HELP node_open_ports list the port listened"
echo "# TYPE node_open_ports gauge"

sudo ss -nltp | awk '/.*:[0-9]+/ {split($4,a,":"); split($(NF),b,":"); split(b[2],c,",");split(c[2],d,/=/); gsub("\\(\\(","",c[1]); print "listened_port=\""a[2]"\", pid=\""d[2]"\", process="c[1]""}' | sort | uniq | awk '{gsub("\\(\\(","\"", $3); print "node_open_ports{"$0"} 1.0"}' | paste -sd "\n" -


```

6 .  目录内文件监控（指定文件名）
```
#!/bin/bash

# */5 * * * *  directory_filecheck.sh /path | sponge /var/lib/node_exporter/textfile/directory_file.prom

echo "# HELP node_directory_filecheck used to check the file download by crontab"
echo "# TYPE node_directory_filecheck gauge"
dir="$1"
weekday=$(date +%u)
date=$(date +%Y-%m-%d)
count=$(find $dir -name  "$date*.zip" | wc -l)
file=$(find $dir -name  "$date*.zip")
if [ -z "$file" ]; then
  file="$date.zip"
fi
if [$weekday  -eq 1 ] && [$weekday -eq 7]; then
  echo "node_directory_filecheck{file=\"$file\",} 3"
else
  echo "node_directory_filecheck{file=\"$file\",} $count" 
fi

  
```

7 .  目录内所有文件
```
#/bin/bash 

# */5 * * * *  directory_filelist.sh /path | sponge /var/lib/node_exporter/textfile/directory_size.prom

echo "# HELP node_directory_filelist used to show the file download by crontab"
echo "# TYPE node_directory_filelist gauge"

dir="$1"
tree "$dir"  --timefmt %Y%m%d  -Dsfi | grep -E '*.zip' | awk '{print $NF, $2, $3 }' | sed 's/]//g' | awk '{printf "node_directory_xiangyucheck{fname=\"%s\", fsize=\"%s\", fdat=\"%s\"} 1\n", $1,$2,$3}'

```

8 . 目录下的.zip 文件数量统计
```
#!/bin/bash
# */5 * * * *  /path/directory.sh /path | sponge /path/textfile/directory_size.prom
echo "# HELP node_directory_filecount used to count file num under monitoring directory"
echo "# TYPE node_directory_filecount gauge"

dir="$1"
for directory in $(tree -dfi $dir | head -n -2 | sed 's/\.//g' ); do
  zipcount=$(find "$directory" -type f  -name "*.zip" 2>/dev/null | wc -l)
  echo "node_directory_filecount{directory=\"$directory\", fcounts=\"$zipcount\"} 1"
done
```
-------------------
### FINAL
```
#!/bin/bash
# 指定目录下的文件计数
# */1 * * * *  /bin/bash  /path/directory_xycount.sh  /path  | sponge /path/textfile/directory_xycount.prom

echo "# HELP node_directory_xycount used to count file num under monitoring directory"
echo "# TYPE node_directory_xycount gauge"

weekday=$(date +%u)
date=$(date '+%Y%m%d' -d '-1day')
if [ -d "$1/$date" ]; then
  for directory in $(tree -dfi "$1/$date" | head -n -2 | sed 's/\.//g' ); do
    zipcount=$(find "$directory" -type f  -name "*.zip" 2>/dev/null | wc -l)
    echo "node_directory_xycount{directory=\"$directory\", weekday=\"$weekday\", fcounts=\"$zipcount\"} 1"
  done
else
  echo "node_directory_xycount{directory=\"$1/$date\", weekday=\"$weekday\",fcounts=\"\"} 0"
fi

```

```
#!/bin/bash
# 指定目录下的文件计数
# ivol: */1 * * * * /bin/bash  /path/directory_ivolcount.sh  /path  | sponge /path/textfile/directory_ivolcount.prom

echo "# HELP node_directory_ivolcount used to count file num under monitoring directory"
echo "# TYPE node_directory_ivolcount gauge"

weekday=$(date +%u)
if [ -d "$1" ]; then
  for directory in $(tree -dfi $1 | head -n -2 | sed 's/\.//g' ); do
    zipcount=$(find "$directory" -type f  -name "*.zip" 2>/dev/null | wc -l)
    echo "node_directory_ivolcount{directory=\"$directory\", weekday=\"$weekday\", fcounts=\"$zipcount\"} 1"
  done
else
  echo "node_directory_ivolcount{directory=\"$1\", weekday=\"$weekday\",fcounts=\"\"} 0"
fi
```

```
#!/bin/sh
# 指定目录的目录大小
# xy: */1 * * * * /bin/bash  /path/directory_xysize.sh  /path  | sponge /path/textfile/directory_xysize.prom

echo "# HELP node_directory_xysize_bytes Disk space used by some directories"
echo "# TYPE node_directory_xysize_bytes gauge"

weekday=$(date +%u)
date=$(date '+%Y%m%d' -d '-1day')
if [ -d "$1/$date" ]; then
  for directory in $(tree -dfi "$1/$date" | head -n -2 | sed 's/\.//g' ); do
    du --block-size=1 --summarize "$directory" \
          | sed -ne 's/\\/\\\\/;s/"/\\"/g;s/^\([0-9]\+\)\t\(.*\)$/node_directory_xysize_bytes{directory="\2",weekday="'$weekday'"} \1/p'
  done
else
  echo "node_directory_xysize_bytes{directory=\"$1/$date\",weekday=\"$weekday\"} 0"
fi

```

```
#!/bin/bash
# 指定目录的目录大小
# ivol: */1 * * * * /bin/bash /path/directory_ivolsize.sh  /path  | sponge /path/textfile/directory_ivolsize.prom

echo "# HELP node_directory_ivolsize_bytes Disk space used by some directories"
echo "# TYPE node_directory_ivolsize_bytes gauge"

weekday=$(date +%u)
if [ -d "$1" ]; then
  for directory in $(tree -dfi $1 | head -n -2 | sed 's/\.//g' ); do
    du --block-size=1 --summarize "$directory" \
          | sed -ne 's/\\/\\\\/;s/"/\\"/g;s/^\([0-9]\+\)\t\(.*\)$/node_directory_ivolsize_bytes{directory="\2",weekday="'$weekday'"} \1/p'
  done
else
  echo "node_directory_ivolsize_bytes{directory=\"$1\",weekday=\"$weekday\"} 0"
fi
```

```
#/bin/bash 
# 指定目录的文件信息列表
# */1 * * * *  /bin/bash /path/pathdirectory_xylist.sh /path | sponge /path/textfile/directory_xylist.prom

echo "# HELP node_directory_xylist used to show the file download by crontab"
echo "# TYPE node_directory_xylist gauge"

weekday=$(date +%u)
date=$(date '+%Y%m%d' -d '-1day')

if [ -d "$1/$date" ]; then
  tree "$1/$date" --timefmt %Y%m%d -Dsfi | grep -E '*.zip'  \
    | awk -v weekday="$weekday" '{date=substr($NF, length($NF)-11, 8); printf "node_directory_xylist{weekday=\"%s\",fname=\"%s\",fsize=\"%s\",fdat=\"%s\"} 1\n", weekday, $NF, $2, date}'
else
  echo "node_directory_xylist{weekday=\"$weekday\",fname=\"$1/$date\",fsize=\"\",fdat=\"\"} 0"
fi
```

```
#/bin/bash 
# 指定目录的文件信息列表
# */1 * * * *  /bin/bash  /path/pathdirectory_ivollist.sh /path | sponge /path/textfile/directory_ivollist.prom

echo "# HELP node_directory_ivollist used to show the file download by crontab"
echo "# TYPE node_directory_ivollist gauge"

weekday=$(date +%u)

if [ -d "$1" ]; then
  tree "$1" --timefmt %Y%m%d -Dsfi | grep -E '*.zip' | awk '{date=substr($NF, length($NF)-11, 8); print $NF, $2, date}' | sort -k3 -r \
    | awk -v weekday="$weekday" '{printf "node_directory_ivollist{weekday=\"%s\",fname=\"%s\",fsize=\"%s\",fdat=\"%s\"} 1\n", weekday, $1, $2, $3}'
else
  echo "node_directory_ivollist{weekday=\"$weekday\",fname=\"$1\",fsize=\"\",fdat=\"\"} 0"
fi
```

```
不使用
#/bin/bash 
# 指定目录下的文件大小
# */1 * * * * /bin/bash /path/directory_ivolzip.sh  /path | sponge /path/textfile/directory_ivolzip.prom

echo "# HELP node_directory_ivolzip used to show the file download by crontab"
echo "# TYPE node_directory_ivolzip gauge"

date=arraystream_$(date '+%Y%m%d' -d '-1day').zip
if [ -e "$1/$date" ]; then
  ls -l "$1/$date" | awk '{printf "node_directory_ivolzip{fname=\"%s\"} %s\n", $9, $5}'
else
  echo "node_directory_ivolzip{fname=\"$1/$date\"} 0"
fi
```

--------------------------------------------------------
## Process_exporter  

1 . [版本地址](https://github.com/ncabatoff/process-exporter/releases)

2 . 修改配置文件，监控全部进程
```
process_names:
  - name: "{{.Comm}}"
    cmdline:
    - '.+'
```

3 . systemd 管理
```
vim  /etc/systemd/system/process_exporter.service 
[Unit]
Description=process_exporter
Documentation=https://prometheus.io/
After=network.target
[Service]
Type=simple
ExecStart=/home/inboc/demo/process_exporter/process-exporter -config.path=/home/inboc/demo/process_exporter/conf.yml
Restart=on-failure
[Install]
WantedBy=multi-user.target

systemctl daemon-reload
systemctl start process_exporter.service   &&   systemctl enabled process_exporter.service
```

4 . prometheus 新增job
```
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["10.13.3.109:9090"]
  - job_name: "node_exporter"
    static_configs:
      - targets: ["10.13.3.110:9100"]
  - job_name: 'process_exporter'
    static_configs:
    - targets: ['10.13.3.110:9256']
```

5 . prometheus重载
```
curl -X POST http://ADDRESS:9090/-/reload
```



-----------------------
##  导出dashboard
1 . dadasource变量，创建为DataSource  类型  ，使用时，pannel中的数据源选择为$datasource，避免数据源不能识别
2 . share-->export-->json file
--------------------
