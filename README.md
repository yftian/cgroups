# cgroups 
>简要记录cgroups对那些系统资源限制，以及基于golang和python不同语言的具体实现。  

blkio – BLOCK IO资源控制  
-- 
* 限额类
>限额类是主要有两种策略，一种是基于完全公平队列调度（CFQ：Completely Fair Queuing ）的按权重分配各个cgroup所能占用总体资源的百分比，好处是当资源空闲时可以充分利用，但只能用于最底层节点cgroup的配置；另一种则是设定资源使用上限，这种限额在各个层次的cgroup都可以配置，但这种限制较为生硬，并且容器之间依然会出现资源的竞争。  
* 按比例分配块设备IO资源  
>1.`blkio.weight`：填写100-1000的一个整数值，作为相对权重比率，作为通用的设备分配比。
>2.`blkio.weight_device`： 针对特定设备的权重比，写入格式为device_types:node_numbers weight，空格前的参数段指定设备，weight参数与blkio.weight相同并覆盖原有的通用分配比。{![查看一个设备的device_types:node_numbers可以使用：ls -l /dev/DEV，看到的用逗号分隔的两个数字就是。有的文章也称之为major_number:minor_number。
* 控制IO读写速度上限  
>1.`blkio.throttle.read_bps_device`：按每秒读取块设备的数据量设定上限，格式device_types:node_numbers bytes_per_second。  
>2.`blkio.throttle.write_bps_device`：按每秒写入块设备的数据量设定上限，格式device_types:node_numbers bytes_per_second。  
>3.`blkio.throttle.read_iops_device`：按每秒读操作次数设定上限，格式device_types:node_numbers operations_per_second。  
>4.`blkio.throttle.write_iops_device`：按每秒写操作次数设定上限，格式device_types:node_numbers operations_per_second。  
* 针对特定操作(read, write, sync, 或async)设定读写速度上限  
>1.`blkio.throttle.io_serviced`：针对特定操作按每秒操作次数设定上限，格式device_types:node_numbers operation operations_per_second  
>2.`blkio.throttle.io_service_bytes`：针对特定操作按每秒数据量设定上限，格式device_types:node_numbers operation bytes_per_second
* 统计与监控
>以下内容都是只读的状态报告，通过这些统计项更好地统计、监控进程的 io 情况。  

>1.`blkio.reset_stats`：重置统计信息，写入一个int值即可。  
>2.`blkio.time`：统计cgroup对设备的访问时间，按格式device_types:node_numbers milliseconds读取信息即可，以下类似。  
>3.`blkio.io_serviced`：统计cgroup对特定设备的IO操作（包括read、write、sync及async）次数，格式device_types:node_numbers operation number  
>4.`blkio.sectors`：统计cgroup对设备扇区访问次数，格式 device_types:node_numbers sector_count  
>5.`blkio.io_service_bytes`：统计cgroup对特定设备IO操作（包括read、write、sync及async）的数据量，格式device_types:node_numbers operation bytes  
>6.`blkio.io_queued`：统计cgroup的队列中对IO操作（包括read、write、sync及async）的请求次数，格式number operation  
>7.`blkio.io_service_time`：统计cgroup对特定设备的IO操作（包括read、write、sync及async）时间(单位为ns)，格式device_types:node_numbers operation time  
>8.`blkio.io_merged`：统计cgroup 将 BIOS 请求合并到IO操作（包括read、write、sync及async）请求的次数，格式number operation  
>9.`blkio.io_wait_time`：统计cgroup在各设​​​备​​​中各类型​​​IO操作（包括read、write、sync及async）在队列中的等待时间​(单位ns)，格式device_types:node_numbers operation time  
>10.`blkio.*_recursive`：各类型的统计都有一个递归版本，Docker中使用的都是这个版本。获取的数据与非递归版本是一样的，但是包括cgroup所有层级的监控数据。  

cpu – CPU资源控制  
--
>CPU资源的控制也有两种策略，一种是完全公平调度 （CFS：Completely Fair Scheduler）策略，提供了限额和按比例分配两种方式进行资源控制；另一种是实时调度（Real-Time Scheduler）策略，针对实时进程按周期分配固定的运行时间。配置时间都以微秒（µs）为单位，文件名中用us表示。    
* CFS调度策略下的配置  
  * 设定CPU使用周期使用时间上限  
  >1.`cpu.cfs_period_us`：设定周期时间，必须与cfs_quota_us配合使用。  
  >2.`cpu.cfs_quota_us` ：设定周期内最多可使用的时间。这里的配置指task对单个cpu的使用上限，若cfs_quota_us是cfs_period_us的两倍，就表示在两个核上完全使用。数值范围为1000 – 1000,000（微秒）。  
  >3.`cpu.stat`：统计信息，包含nr_periods（表示经历了几个cfs_period_us周期）、nr_throttled（表示task被限制的次数）及throttled_time（表示task被限制的总时长）。  
    * 按权重比例设定CPU的分配  
    >1.`cpu.shares`：设定一个整数（必须大于等于2）表示相对权重，最后除以权重总和算出相对比例，按比例分配CPU时间。（如cgroup A设置100，cgroup B设置300，那么cgroup A中的task运行25%的CPU时间。对于一个4核CPU的系统来说，cgroup A 中的task可以100%占有某一个CPU，这个比例是相对整体的一个值。）  
* RT调度策略下的配置  
>实时调度策略与公平调度策略中的按周期分配时间的方法类似，也是在周期内分配一个固定的运行时间。  

>1.`cpu.rt_period_us` ：设定周期时间。  
>2.`cpu.rt_runtime_us`：设定周期中的运行时间。  

cpuacct – CPU资源报告  
--  
>这个子系统的配置是cpu子系统的补充，提供CPU资源用量的统计，时间单位都是纳秒。  
>1.`cpuacct.usage`：统计cgroup中所有task的cpu使用时长  
>2.`cpuacct.stat`：统计cgroup中所有task的用户态和内核态分别使用cpu的时长  
>3.`cpuacct.usage_percpu`：统计cgroup中所有task使用每个cpu的时长  

cpuset – CPU绑定  
--  
>为task分配独立CPU资源的子系统，参数较多，这里只选讲两个必须配置的参数，同时Docker中目前也只用到这两个。  
>1.`cpuset.cpus`：在这个文件中填写cgroup可使用的CPU编号，如0-2,16代表 0、1、2和16这4个CPU。
>2.`cpuset.mems`：与CPU类似，表示cgroup可使用的memory node，格式同上

device – 限制task对device的使用  
--  
* **设备黑/白名单过滤 **  
>1.`devices.allow`：允许名单，语法type device_types:node_numbers access type ；type有三种类型：b（块设备）、c（字符设备）、a（全部设备）；access也有三种方式：r（读）、w（写）、m（创建）。  
>2`devices.deny`：禁止名单，语法格式同上。  
* 统计报告  
>1.`devices.list`：报告为这个cgroup 中的task设定访问控制的设备  

freezer – 暂停/恢复cgroup中的task  
--
>只有一个属性，表示进程的状态，把task放到freezer所在的cgroup，再把state改为FROZEN，就可以暂停进程。不允许在cgroup处于FROZEN状态时加入进程。  
* **freezer.state **，包括如下三种状态：  
– FROZEN 停止  
– FREEZING 正在停止，这个是只读状态，不能写入这个值。  
– THAWED 恢复  

memory – 内存资源管理  
--  
* 限额类  
>1.`memory.limit_in_bytes`：强制限制最大内存使用量，单位有k、m、g三种，填-1则代表无限制。  
>2.`memory.soft_limit_in_bytes`：软限制，只有比强制限制设置的值小时才有意义。填写格式同上。当整体内存紧张的情况下，task获取的内存就被限制在软限制额度之内，以保证不会有太多进程因内存挨饿。可以看到，加入了内存的资源限制并不代表没有资源竞争。  
>3.`memory.memsw.limit_in_bytes`：设定最大内存与swap区内存之和的用量限制。填写格式同上。  
* 报警与自动控制  
>1.`memory.oom_control`：改参数填0或1， 0表示开启，当cgroup中的进程使用资源超过界限时立即杀死进程，1表示不启用。默认情况下，包含memory子系统的cgroup都启用。当oom_control不启用时，实际使用内存超过界限时进程会被暂停直到有空闲的内存资源  
* 统计与监控类  
>1.`memory.usage_in_bytes`：报告该 cgroup中进程使用的当前总内存用量（以字节为单位）  
>2.`memory.max_usage_in_bytes`：报告该 cgroup 中进程使用的最大内存用量  
>3.`memory.failcnt`：报告内存达到在 memory.limit_in_bytes设定的限制值的次数  
>4.`memory.stat`：包含大量的内存统计数据。  
* cache：页缓存包括 tmpfs（shmem），单位为字节。  
* rss：匿名和 swap 缓存不包括 tmpfs（shmem），单位为字节。  
* mapped_file：memory-mapped 映射文件大小包括 tmpfs（shmem），单位为字节  
* pgpgin：存入内存中的数  
* pgpgout：从内存中读出的页数  
* swap：swap 用量单位为字节  




