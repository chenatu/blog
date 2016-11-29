title: 利用collectd, influxdb和grafana进行简单的负载预警
date: 2016/11/18
categories:
- coding
tags:
- 运维
- 机器学习
---
本文仅仅是对负载预警的简单尝试，能够预测的场景也比较有限，但是作为预测工作的开始已经比较能够说明问题了。

利用collectd, influxdb和grafana进行监控系统搭建可以参考这篇文章[Monitoring hosts with CollectD, InfluxDB and Grafana](http://jansipke.nl/monitoring-hosts-with-collectd-influxdb-and-grafana/) grafana的操作比nagios和cacti真的友好很多，可定制的能力也强很多。

![负载监控][1]

虽然grafana有一定的报警能力（在grafana4.0版本之后,alert模块直接继承进来，所以推荐4.0的版本），但是能够提前十几分钟对于集群负载超标进行预警，一直是我们的一个小目标。所以在这里，我们就开始为这个小目标做了一点小努力。

在实际运行中，我们安装了这个[小插件](https://github.com/anryko/grafana-influx-dashboard)。它能够对应用负载进行批量的显示，节约了好多体力活。

![图片描述][2]

通过分析，我们可以看到以下几种负载

简单趋势型

![图片描述][3]

周期型

![图片描述][4]

规律不明显型

而对于周期型我们也可以看到在短期是可以看到一定趋势的。

在这里我们仅采用线性回归对简单趋势进行预测。这点对于python很好实现

````
from datetime import datetime
from influxdb import DataFrameClient
import numpy as np
from scipy import stats

if __name__ == "__main__":
    host = 'localhost'
    port = 8086
    user = 'root'
    password = 'root'
    dbname = 'collectd'

    client = DataFrameClient(host, port, user, password, dbname)

    print("Create pandas DataFrame")
    start_time = datetime(2016, 11, 17, 3,10).timestamp()
    end_time = datetime(2016, 11, 17, 8,50).timestamp()
    query = "select * from load_midterm where host='web153' and time > " + str(int(start_time)) + "s and time < " \
            + str(int(end_time)) + "s"
    df = client.query(query)

    slope, intercept, r_value, p_value, std_err = stats.linregress(range(df['load_midterm'].shape[0]), df['load_midterm']['value'].values)
    print(slope)
    print(intercept)
    print(r_value)
    print(p_value)
````

在这里我们用的是统计模块得到斜率和截距。需要注意的是，我们还要对R2，p值以及方差进行评价。一般来说r^2 >0.7, p<0.05, std_err<0.1

在这里，为什么没有采用一些更复杂的机器学习方法呢？

无论采用什么模型，关键是要提取一些关键feature。我观察过简单的应用服务器的其他指标，包括tcp连接数，cpu，磁盘io等，与load的协同性比较高，很难成为15min后load的先导预测feature。如果从服务器集群的整体角度去找feature的话，也许是可以做的，未来会关注这块。


在具体实现上，后端基于python flask编写一个预测服务器，而前端开发可以基于grafana的clock插件进行开发:[clock-panel](https://github.com/grafana/clock-panel)


  [1]: http://7xpw11.com1.z0.glb.clouddn.com/web57.jpg
  [2]: http://7xpw11.com1.z0.glb.clouddn.com/loads.jpg
  [3]: http://7xpw11.com1.z0.glb.clouddn.com/linear.jpg
  [4]: http://7xpw11.com1.z0.glb.clouddn.com/cycle.jpg