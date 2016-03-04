DRA用户手册
===================

###前言

本文为DRA用户手册,为使用该工具的用户提供帮助


----------


###环境配置

 - 配置多节点openstack（至少两台compute节点）
 - 必须配置的服务包括nova，ceilometer，glance，keystone，neutron
 - 实现compute节点间虚拟机热迁移配置
 - 安装python的oslo_config, oslo.messaging, matplotlib包


----------


###系统配置

 - 源码包中的Dynamic Resource Allocation/Openstack/Conf/OpenstackConf.py文件中包含对系统的配置信息，其中：

  + `CONTROLLER_HOST` 主节点ip
  + `COMPUTE1_HOST` 计算节点1 ip
  + `COMPUTE2_HOST` 计算节点2 ip
  + `AUTH_URL` keystone认证url
  + `NOVA_URL` nova认证url
  + `CEILOMETER_URL` ceilometer认证url
  + `HOST_USERNAME` 主节点用户名
  + `HOST_PASSWORD` 主节点密码
  + `COMPUTE1_HOST_USERNAME` 计算节点1用户名
  + `COMPUTE1_HOST_PASSWORD` 计算节点1密码
  + `COMPUTE2_HOST_USERNAME` 计算节点2用户名
  + `COMPUTE2_HOST_PASSWORD` 计算节点2密码
  + `PARAMS` 命令执行环境变量
 

----------

###使用

####openstack
 - 启动服务（在Dynamic Resource Allocation/Hades/Cmd目录下）
   + arbiterPMA
   + monitorPMA
   + arbiter
   + policyService
   + eventService
 
 - 载入策略文件，执行Dynamic Resource Allocation/PolicyService/RpcApi.py，其中策略文件在Dynamic Resource Allocation/Resource/testPolicy.xml。其中规则migrate_0中的‘compute1_compute1’, 'compute2_compute2'修改为实际使用的计算节点的id

    ```
    < rule name = "migrate_0">
       (defrule migrate_0
            (host_violation ?resourceId ?meter_name)
            (bind ?vm (python-call Vm_Random_Selector ?vms))
            (bind ?hostIds (python-call Host_Filter "['compute1_compute1', 'compute2_compute2']" "{'compute.node.cpu.percent' : {'min' : 0, 'max' : 90}}"))
            (bind ?destHost (python-call Host_Generic_Selector ?hostIds "['Host_CpuUtil_Cost']" "[1]"))
            (python-call Migrate ?vm ?destHost))
    </rule>
    ```
    

 - 向规则引擎发送事件，执行Dynamic Resource Allocation/EventService/RpcApi.py。其中发送事件中的'compute2_compute2'改为实际计算节点的id
	```
	api.sendEvent({}, "pike", "monitorPMA", "(host_collect_data_statistics compute2_compute2 compute.node.cpu.percent %s None None None avg)" % query)
	```

####simulator
模拟器模拟了不同应用场景下的虚拟机和物理机的资源使用情况，存放在Dynamic Resource Allocation/Simulator下。

main.py文件为模拟器主文件，通过设置‘exp‘变量实施不同的模拟，其中：

- exp=1 为MATLAB类型应用模拟
- exp=2 为GAME类型应用模拟
- exp=3 为HADOOP类型应用模拟
- exp=4 为MATLAB, GAME, HADOOP类型应用混合场景模拟
- exp=5 产生后台数据与前端页面交互

###查看结果
####openstack
无具体输出，运行时刻log信息会在每个服务进程打印，例如arbiterPMA：
```
config init

serve service

service wait

Starting hades_arbiterPMA_topic node (version 1.0)
Creating RPC server for service hades_arbiterPMA_topic
loadPolicy
handleEvent: (host_violation compute2_compute2 compute.node.cpu.percent)
Get_Vms_On_Host: compute2_compute2
[u'9635390f-06f8-4388-875b-94947ae649f6', u'9c0ad800-bbb8-47c2-9f03-8c9583fae56e', u'7855a7ef-6723-406c-84ae-c429bfdb02c5', u'2bdb61c4-8a60-4aa9-9085-f4300efbd0ab', u'47a89463-ed3e-42c3-8c61-7e451d57d4ad']
Vm_Random_Selector
9c0ad800-bbb8-47c2-9f03-8c9583fae56e
Host_Filter : ['compute1_compute1', 'compute2_compute2']
Collect_Data_Statistics
compute1_compute1 : 1
Collect_Data_Statistics
compute2_compute2 : 1
filteredHosts: ['compute1_compute1', 'compute2_compute2']
Host_Generic_Selector: ['compute1_compute1', 'compute2_compute2']
Host_CpuUtil_Cost
Collect_Data_Statistics
compute1_compute1 : 0.99
Host_CpuUtil_Cost
Collect_Data_Statistics
compute2_compute2 : 0.99
compute2_compute2
migrate: 9c0ad800-bbb8-47c2-9f03-8c9583fae56e to compute2
```


----------


####simulator
四种类型应用的模拟输出会通过matplotlib打印图表，其中：

 - exp=1 显示物理机CPU使用率信息和MATLAB集群通信开销
 - exp=2 显示物理机Bandwidth使用率信息和GAME虚拟机状态
 - exp=3 显示HADOOP集群通信开销
 - exp=4 显示物理机Bandwidth使用率信息和MATLAB, HADOOP集群通信开销
