# Description of the problem we're trying to solve:
- You have a PCF/PAS foundation running multiple PCC Clusters in different Orgs and Spaces
- You want to extract metrics from the loggregator for one particular instance of PCC
- How do you do that?

## Reference: The PAS environment I used 

- PAS 2.4.3 
- PCC 1.7.0 
- Ops Manager 2.4
- vSphere 6.5 

## Step 1. Where are you?

Make sure you are pointing at the correct `API`,`ORG` and `SPACE`:
```
$ cf t
```
```
api endpoint:   https://api.system.pcf4u.com
api version:    2.125.0
user:           admin
org:            demo
space:          demo
```

## Step 2. List the PCC Service Instances in your ORG and SPACE

```
$ cf s | (head -n 3; grep cloud)
```
```
Getting services in org demo / space demo as admin...

name               service          plan          bound apps            last operation
dev-cluster        p-cloudcache     dev-plan      pcc-lookaside-cache   create succeeded
ExtraSmallPCC      p-cloudcache     extra-small                         create succeeded
SmallPCC           p-cloudcache     small                               create succeeded
```
As you can see, there are 3 Service Instances of PCC in this ORG and SPACE, and only one of them is bound to an App.

## Step 3. The GUIDs for all Service Instances across all ORGs and SPACEs

For this step to work, you will need to be logged in as `admin`. Please note that Service Instances of all types (PCC, MySQL, NFS, Spring Cloud, etc...) will be displayed, including any additional PCC Services Instances that were not in the ORG and SPACE used in Steps 1 & 2.

```
$ cf curl /v2/service_instances | jq '.resources[] .entity .name, .resources[] .metadata .guid'
```
```
"myVolume"
"autoscale-demo"
"mysql-dev"
"dev-cluster"
"uaa-sso-instance"
"RALPH"
"SQLSvr4Chess"
"Birch"
"spring-cloud-broker-db"
"spring-cloud-broker-rmq"
"push-notifications-mysql"
"push-notifications-rabbitmq"
"SmallPCC"
"ExtraSmallPCC"
"PCC_Dev_Test"
"2bf6ada8-5360-4b65-ad58-962def6dea64"
"bf195dd6-5a15-4854-9827-618317f8e2d4"
"4cf228ed-9ea1-418c-9aa1-f4a199351fd2"
"f0dcd3e4-4645-466e-ac14-89686cb2c905"
"2a358ea8-6478-4e84-af19-d58caa5e88fa"
"8f26a4fe-f353-4eff-9ab3-39d8d92b0072"
"7c0f00a6-12d9-405d-a4c7-47012690d499"
"47e99301-0f2e-4d65-8842-93f9a475cecc"
"18d0aa30-fe38-4a20-80fb-16f52b947cf4"
"ba210046-52ee-4a35-9937-459e2559e053"
"ad595a5b-5151-4a08-802f-5b10543154ec"
"75a29f5f-b0e7-434e-b299-16c4705b75d7"
"a83cf6e1-7592-4b69-b8aa-2e6494686aca"
"f216dc9a-a9c1-4a7c-a7b9-de16814ebf43"
"18c6a4f2-44d9-4e2a-978a-1b96b97c90dc"
```

Note that we have 15 Service Instance Names followed in the same order by their respective Service Instance GUIDs.

The Service Instance named `"PCC_Dev_Test"` for example, is a PCC Service Instance that was not created in the Demo/Demo ORG and SPACE.

The Service Instance named `"dev-cluster"` (fourth on the list) is the one bound to the `pcc-lookaside-cache` App, so let's use it as the example. It's corresponding GUID is `"f0dcd3e4-4645-466e-ac14-89686cb2c905"`

## Step 4. Let's collect loggregator data for the dev_cluster

```
$ cf nozzle -n | grep f0dcd3e4-4645-466e-ac14-89686cb2c905
```
```
origin:"bosh-system-metrics-forwarder" eventType:ValueMetric timestamp:1552600504000000000 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"" tags:<key:"id" value:"420d9295-2734-45f1-90f2-c66e8c896cd4" > tags:<key:"product" value:"Pivotal Application Service" > tags:<key:"source_id" value:"bosh-system-metrics-forwarder" > tags:<key:"system_domain" value:"system.pcf4u.com" > valueMetric:<name:"system.swap.kb" value:0 unit:"Kb" >  
origin:"bosh-system-metrics-forwarder" eventType:ValueMetric timestamp:1552600504000000000 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"" tags:<key:"id" value:"420d9295-2734-45f1-90f2-c66e8c896cd4" > tags:<key:"product" value:"Pivotal Application Service" > tags:<key:"source_id" value:"bosh-system-metrics-forwarder" > tags:<key:"system_domain" value:"system.pcf4u.com" > valueMetric:<name:"system.disk.persistent.inode_percent" value:0 unit:"Percent" >  
origin:"bosh-system-metrics-forwarder" eventType:ValueMetric timestamp:1552600504000000000 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"" tags:<key:"id" value:"420d9295-2734-45f1-90f2-c66e8c896cd4" > tags:<key:"product" value:"Pivotal Application Service" > tags:<key:"source_id" value:"bosh-system-metrics-forwarder" > tags:<key:"system_domain" value:"system.pcf4u.com" > valueMetric:<name:"system.cpu.wait" value:0.1 unit:"Load" >  
origin:"bosh-system-metrics-forwarder" eventType:ValueMetric timestamp:1552600504000000000 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"" tags:<key:"id" value:"420d9295-2734-45f1-90f2-c66e8c896cd4" > tags:<key:"product" value:"Pivotal Application Service" > tags:<key:"source_id" value:"bosh-system-metrics-forwarder" > tags:<key:"system_domain" value:"system.pcf4u.com" > valueMetric:<name:"system.cpu.sys" value:1.6 unit:"Load" >  
origin:"bosh-system-metrics-forwarder" eventType:ValueMetric timestamp:1552600504000000000 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"" tags:<key:"id" value:"420d9295-2734-45f1-90f2-c66e8c896cd4" > tags:<key:"product" value:"Pivotal Application Service" > tags:<key:"source_id" value:"bosh-system-metrics-forwarder" > tags:<key:"system_domain" value:"system.pcf4u.com" > valueMetric:<name:"system.disk.ephemeral.percent" value:14 unit:"Percent" >  
origin:"bosh-system-metrics-forwarder" eventType:ValueMetric timestamp:1552600504000000000 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"" tags:<key:"id" value:"420d9295-2734-45f1-90f2-c66e8c896cd4" > tags:<key:"product" value:"Pivotal Application Service" > tags:<key:"source_id" value:"bosh-system-metrics-forwarder" > tags:<key:"system_domain" value:"system.pcf4u.com" > valueMetric:<name:"system.swap.percent" value:0 unit:"Percent" >  
origin:"bosh-system-metrics-forwarder" eventType:ValueMetric timestamp:1552600504000000000 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"" tags:<key:"id" value:"420d9295-2734-45f1-90f2-c66e8c896cd4" > tags:<key:"product" value:"Pivotal Application Service" > tags:<key:"source_id" value:"bosh-system-metrics-forwarder" > tags:<key:"system_domain" value:"system.pcf4u.com" > valueMetric:<name:"system.mem.kb" value:2.050692e+06 unit:"Kb" >  
origin:"bosh-system-metrics-forwarder" eventType:ValueMetric timestamp:1552600504000000000 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"" tags:<key:"id" value:"420d9295-2734-45f1-90f2-c66e8c896cd4" > tags:<key:"product" value:"Pivotal Application Service" > tags:<key:"source_id" value:"bosh-system-metrics-forwarder" > tags:<key:"system_domain" value:"system.pcf4u.com" > valueMetric:<name:"system.disk.persistent.percent" value:13 unit:"Percent" >  
origin:"bosh-system-metrics-forwarder" eventType:ValueMetric timestamp:1552600504000000000 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"" tags:<key:"id" value:"420d9295-2734-45f1-90f2-c66e8c896cd4" > tags:<key:"product" value:"Pivotal Application Service" > tags:<key:"source_id" value:"bosh-system-metrics-forwarder" > tags:<key:"system_domain" value:"system.pcf4u.com" > valueMetric:<name:"system.disk.system.percent" value:46 unit:"Percent" >  
origin:"bosh-system-metrics-forwarder" eventType:ValueMetric timestamp:1552600504000000000 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"" tags:<key:"id" value:"420d9295-2734-45f1-90f2-c66e8c896cd4" > tags:<key:"product" value:"Pivotal Application Service" > tags:<key:"source_id" value:"bosh-system-metrics-forwarder" > tags:<key:"system_domain" value:"system.pcf4u.com" > valueMetric:<name:"system.mem.percent" value:25 unit:"Percent" >  
origin:"bosh-system-metrics-forwarder" eventType:ValueMetric timestamp:1552600504000000000 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"" tags:<key:"id" value:"420d9295-2734-45f1-90f2-c66e8c896cd4" > tags:<key:"product" value:"Pivotal Application Service" > tags:<key:"source_id" value:"bosh-system-metrics-forwarder" > tags:<key:"system_domain" value:"system.pcf4u.com" > valueMetric:<name:"system.load.1m" value:0.17 unit:"Load" >  
origin:"bosh-system-metrics-forwarder" eventType:ValueMetric timestamp:1552600504000000000 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"" tags:<key:"id" value:"420d9295-2734-45f1-90f2-c66e8c896cd4" > tags:<key:"product" value:"Pivotal Application Service" > tags:<key:"source_id" value:"bosh-system-metrics-forwarder" > tags:<key:"system_domain" value:"system.pcf4u.com" > valueMetric:<name:"system.healthy" value:1 unit:"b" >  
origin:"bosh-system-metrics-forwarder" eventType:ValueMetric timestamp:1552600504000000000 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"" tags:<key:"id" value:"420d9295-2734-45f1-90f2-c66e8c896cd4" > tags:<key:"product" value:"Pivotal Application Service" > tags:<key:"source_id" value:"bosh-system-metrics-forwarder" > tags:<key:"system_domain" value:"system.pcf4u.com" > valueMetric:<name:"system.cpu.user" value:3.1 unit:"Load" >  
origin:"bosh-system-metrics-forwarder" eventType:ValueMetric timestamp:1552600504000000000 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"" tags:<key:"id" value:"420d9295-2734-45f1-90f2-c66e8c896cd4" > tags:<key:"product" value:"Pivotal Application Service" > tags:<key:"source_id" value:"bosh-system-metrics-forwarder" > tags:<key:"system_domain" value:"system.pcf4u.com" > valueMetric:<name:"system.disk.ephemeral.inode_percent" value:1 unit:"Percent" >  
origin:"bosh-system-metrics-forwarder" eventType:ValueMetric timestamp:1552600504000000000 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"" tags:<key:"id" value:"420d9295-2734-45f1-90f2-c66e8c896cd4" > tags:<key:"product" value:"Pivotal Application Service" > tags:<key:"source_id" value:"bosh-system-metrics-forwarder" > tags:<key:"system_domain" value:"system.pcf4u.com" > valueMetric:<name:"system.disk.system.inode_percent" value:33 unit:"Percent" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552600506203552695 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"serviceinstance.TotalHeapSize" value:7638 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552600506203552695 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"member.JVMPauses" value:0 unit:"count" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552600506203552695 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"member.GetsAvgLatency" value:2800 unit:"ms" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552600506203552695 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"member.TotalFileDescriptorOpen" value:140 unit:"count" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552600506203552695 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"diskstore.TotalSpace" value:0 unit:"bytes" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552600506203552695 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"member.UsedMemoryPercentage" value:7.127819 unit:"percentage" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552600506203552695 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"member.HostCpuUsage" value:4 unit:"percentage" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552600506203552695 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"diskstore.UsableSpace" value:0 unit:"bytes" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552600506203552695 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"member.UsedMemory" value:474 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552600506203552695 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"member.PutsAvgLatency" value:6.45407e+06 unit:"ms" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552600506203552695 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"member.FileDescriptorRemaining" value:3956 unit:"count" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552600506203552695 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"serviceinstance.MemberCount" value:2 unit:"count" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552600506203552695 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"gatewayReceiver.EventsReceivedRate" value:0 unit:"count" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552600506203552695 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"member.FileDescriptorLimit" value:4096 unit:"count" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552600506203552695 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"member.MaxMemory" value:6650 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552600506203552695 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"member.GarbageCollectionCount" value:422 unit:"count" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552600506203552695 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"diskstore.DiskWritesAvgLatency" value:45211 unit:"count" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552600506203552695 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"serviceinstance.UnusedHeapSizePercentage" value:92 unit:"percent" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552600506203552695 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:591 unit:"megabytes" >
```
You can scroll to the right to see the long/complete logs.

Let's try again, but this time lets focus on `UsedHeapSize`:

```
$ cf nozzle -n | grep f0dcd3e4-4645-466e-ac14-89686cb2c905 | grep UsedHeapSize
```
```
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552601059828759374 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:252 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552601121320111583 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:312 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552601183000219456 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:251 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552601244440385793 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:307 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" eventType:ValueMetric timestamp:1552601306176586672 deployment:"service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" job:"locator-server" index:"420d9295-2734-45f1-90f2-c66e8c896cd4" ip:"10.0.40.3" tags:<key:"source_id" value:"p-cloudcache.service-instance_f0dcd3e4-4645-466e-ac14-89686cb2c905" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:383 unit:"megabytes" >
```



