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
