简单promql案例
在系统资源监控用得较多的方法是"USE"方法，分别表示为：Utilization（使用率）、Saturation（饱和度）、Errors（错误数）

1、CPU使用率监控

(1- (avg(irate(node_cpu_seconds_total{nodename=~"monitor01",mode="idle"}[5m])))) * 100

或者

100 - (avg(irate(node_cpu_seconds_total{nodename=~"monitor01",mode="idle"}[5m])) * 100)



2、内存使用率监控

(1- (node_memory_MemAvailable_bytes{nodename="monitor01"})/node_memory_MemTotal_bytes{nodename="monitor01"} ) * 100

或

(node_memory_MemTotal_bytes{nodename="monitor01"} - node_memory_MemAvailable_bytes{nodename="monitor01"})/node_memory_MemTotal_bytes{nodename="monitor01"} * 100

如果将Buffers和Cached也作为可用内存，则内存使用率计算公式如下：

(node_memory_MemTotal_bytes{nodename="monitor01"} - (node_memory_MemFree_bytes{nodename="monitor01"} + node_memory_Buffers_bytes{nodename="monitor01"} + node_memory_Cached_bytes{nodename="monitor01"}))/node_memory_MemTotal_bytes{nodename="monitor01"} * 100



上述公式的标签名用nodename来替换了instance，因为在prometheus中配置时，instance和job都按缺省取默认值。



3、磁盘分区使用率监控

(1- (node_filesystem_avail_bytes{nodename="monitor01",mountpoint="/"} / node_filesystem_size_bytes{nodename="monitor01",mountpoint="/"})) * 100



磁盘使用预测

predict_linear()函数，根据前一个时间段的值来预测未来某个时间点数据的走势。

predict_linear(node_filesystem_free_bytes{device="rootfs",nodename=~"monitor01",mountpoint="/"}[1d],24*3600) /(1024*1024*1024)

上面这个表达式含义是根据近1天的磁盘空闲情况，预测在明天的这个时间磁盘还空闲多少。

测试：dd if=/dev/zero of=/disktest bs=1024 count=2097152



4、CPU饱和度监控

CPU饱和度通常是按系统的平均负载来衡量，如观察主机CPU数量(通常按逻辑CPU来算)在一段时间内平均运行的队列长度，当平均负载小于vCPU数量则认为是正常的，若负载长时间超出CPU的数量则认为CPU饱和。

我们先来看看如何查看主机的物理CPU个数、每个物理CPU的核数、每个物理CPU有多少个逻辑CPU（购买云主机时的vcpu数或叫线程数）。

通过过滤"physical id"查看物理CPU数

# cat /proc/cpuinfo |grep "physical id" | uniq | wc -l

通过过滤"cpu cores"查看每个物理CPU有多个核数

# cat /proc/cpuinfo | grep "cpu cores" | uniq

通过过滤"siblings"查看每个物理CPU下有多少个逻辑CPU数

# cat /proc/cpuinfo | grep "siblings" |uniq

或者通过过滤"processor"查看所有物理cpu下的逻辑cpu个数

# cat /proc/cpuinfo | grep "processor" | wc -l



CPU饱和度监控

avg(node_load1{nodename="monitor01"}) by (nodename)/count(node_cpu_seconds_total{nodename="monitor01",mode="system"}) by (nodename)

或者以下这种写法

avg(node_load1{nodename="monitor01"})/count(node_cpu_seconds_total{nodename="monitor01",mode="system"})



5、内存饱和度

可以用下面这两个指标来评估内存饱和度。

node_vmstat_pswpin：系统每秒从swap读到内存的字节数，读取的是/proc/vmstat下的pswpin(si)，单位是KB/s 。

node_vmstat_pswpout：系统每秒从内存写到swap字节数，读取的是/proc/vmstat下的pswpout(so)，单位是KB/s。

内存饱和度计算，个人理解是上面2个指标之和大于0时表示已使用到交换分区，内存达到饱和？但如果交换分区未启用该如何计算饱和度？



6、磁盘IO使用率

avg(irate(node_disk_io_time_seconds_total{nodename="monitor01"}[1m])) by(nodename) * 100



7、网卡接收/发送流量监控

irate(node_network_receive_bytes_total{nodename=~'monitor01',device=~"ens5"}[5m])*8

irate(node_network_transmit_bytes_total{nodename=~'monitor01',device=~"ens5"}[5m])*8



increase()函数，获取区间向量中的第一个和最后一个样本并返回其增长量。如果除以[区间]时间(秒)就可以获取该时间内的平均增长率与rate函数用途相同(注意是rate()不是irate)， 作者：itcooking https://www.bilibili.com/read/cv6494666?spm_id_from=333.999.0.0 出处：bilibili