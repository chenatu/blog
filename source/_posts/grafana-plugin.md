title: 安利一个运维监控平台grafana的插件grafana-influx-dashboard
date: 2016/11/14
categories:
- coding
tags:
- 运维
---
在这安利一个grafana的监控插件[grafana-influx-dashboard](https://github.com/anryko/grafana-influx-dashboard) 使用方法直接在github主页就能看到

利用grafana+collectd+influxdb快速搭建监控系统是很多创业公司采用的一个方案，网上也有很多相关教程。但是对于管理几十台上百台机器的话，一台一台配置graph确实好麻烦。grafana提供了相应的scripted template编程方式。但是编写起来还是要花一些时间，尤其对于前端经验匮乏的运维来说还是有一定难度。这个grafana的插件利用js模板直接获得所有主机，通过输入不同的get参数能进行多样的监控选择，废话不说，直接上效果。

主机列表
![getdash.jpg][1]

某台机器的所有指标

![web062.jpg][2]

所有主机的load
![load.jpg][3]

有时间了我也会对这个插件做二次开发

  [1]: http://7xpw11.com1.z0.glb.clouddn.com/getdash.jpg
  [2]: http://7xpw11.com1.z0.glb.clouddn.com/web062.jpg
  [3]: http://7xpw11.com1.z0.glb.clouddn.com/load.jpg