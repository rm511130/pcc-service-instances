# Description of the problem we're trying to solve:
- You have a PCF/PAS foundation running multiple PCC Clusters in different Orgs and Spaces
- You want to extract metrics, including Service Instance Names and GUIDs, for any given PCC cluster
- How do you do that in a way that matches Service Instance Names to their GUIDs?

The next few steps/sections are meant to familiarize you with the concepts and commands that provide the information necessary for pairing Service Instance Names to their repective GUIDs. 

## Reference: The PAS environment I used 

- PAS 2.4.3 
- PCC 1.7.0 
- Ops Manager 2.4
- vSphere 6.5 

## Step 1. It's important to know where you are in terms of PCF Foundation, ORG and SPACE:

Make sure you are pointing at the `API`,`ORG` and `SPACE` of interest:
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
Note that the `cf services` or `cf s` command will only list the services in the current targeted ORG and SPACE.

## Step 3. The GUIDs for all Service Instances across all ORGs and SPACEs

For this step to work, you will need to be logged in as `admin`. 

Note that Service Instances of all types (PCC, MySQL, NFS, Spring Cloud, etc...) will be displayed, including any additional PCC Services Instances that were not in the ORG and SPACE used in Steps 1 & 2.

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

The Service Instance named `"PCC_Dev_Test"` for example, is a PCC Service Instance that was not created in the Demo/Demo:ORG/SPACE. We know this because its name did no show up in the results of Step 2.

We know from Step 2 that the Service Instance named `"dev-cluster"` (fourth on the list) is the one bound to the `pcc-lookaside-cache` App, so let's use it as the example. It's corresponding GUID is `"f0dcd3e4-4645-466e-ac14-89686cb2c905"` (fourth on the list of GUIDs shown above).

## Step 4. Let's collect loggregator data for the dev_cluster

The output from `cf nozzle -n` displays all messages from the Loggregator. These messages include GUIDs for Service Instances, so we must use the GUID for `"dev-cluster"` if we are to find metrics that pertain to it:

```
$ cf nozzle -n | grep f0dcd3e4-4645-466e-ac14-89686cb2c905
```
```
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
Interesting results, but they are oddly simple. You can only see data for a single IP address. That's because `"dev-cluster"` was created as a single-node `dev-plan`.

## Step 5. Let's find UsedHeapSize for the SmallPCC Cluster GUID a83cf6e1-7592-4b69-b8aa-2e6494686aca

The SmallPCC was created using the SMALL PCC Plan that uses 3 Locators and 6 Cache-Servers. The results below show 9 consecutive IP addresses ranging from 10.0.40.10 to 10.0.40.18:

```
$ cf nozzle -n | grep a83cf6e1-7592-4b69-b8aa-2e6494686aca | grep UsedHeapSize
```
```
origin:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" eventType:ValueMetric timestamp:1552603065025384802 deployment:"service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" job:"server" index:"f6c1b40d-21ab-4832-98aa-1db482a383a2" ip:"10.0.40.18" tags:<key:"source_id" value:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:1488 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" eventType:ValueMetric timestamp:1552603065072952058 deployment:"service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" job:"server" index:"015d5a38-d79d-4049-bdcc-c1b614a67b4e" ip:"10.0.40.17" tags:<key:"source_id" value:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:1546 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" eventType:ValueMetric timestamp:1552603080475248473 deployment:"service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" job:"locator" index:"8fb576e1-1855-4ceb-bdb9-f82320993833" ip:"10.0.40.10" tags:<key:"source_id" value:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:1414 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" eventType:ValueMetric timestamp:1552603080699246417 deployment:"service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" job:"server" index:"ff9638e0-864e-4740-bddc-dced52195dd1" ip:"10.0.40.16" tags:<key:"source_id" value:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:1403 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" eventType:ValueMetric timestamp:1552603080841161250 deployment:"service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" job:"locator" index:"febdbe54-4d25-433b-b830-f7f5d9d04f7d" ip:"10.0.40.12" tags:<key:"source_id" value:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:1425 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" eventType:ValueMetric timestamp:1552603080969267405 deployment:"service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" job:"locator" index:"debb3643-6eb6-470c-88f5-4afd6138bb9d" ip:"10.0.40.11" tags:<key:"source_id" value:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:1425 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" eventType:ValueMetric timestamp:1552603083614587980 deployment:"service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" job:"server" index:"c8a7c4ed-c9fe-4e3e-aa59-c27692590f4f" ip:"10.0.40.15" tags:<key:"source_id" value:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:1472 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" eventType:ValueMetric timestamp:1552603096981669850 deployment:"service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" job:"server" index:"dac7efe6-714d-4768-95b0-cadda427530e" ip:"10.0.40.13" tags:<key:"source_id" value:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:1503 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" eventType:ValueMetric timestamp:1552603112663786086 deployment:"service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" job:"server" index:"ad3c7980-cb68-4e18-b701-c12741f91089" ip:"10.0.40.14" tags:<key:"source_id" value:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:1665 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" eventType:ValueMetric timestamp:1552603125968630128 deployment:"service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" job:"server" index:"015d5a38-d79d-4049-bdcc-c1b614a67b4e" ip:"10.0.40.17" tags:<key:"source_id" value:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:1676 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" eventType:ValueMetric timestamp:1552603125928727063 deployment:"service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" job:"server" index:"f6c1b40d-21ab-4832-98aa-1db482a383a2" ip:"10.0.40.18" tags:<key:"source_id" value:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:1618 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" eventType:ValueMetric timestamp:1552603141619849527 deployment:"service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" job:"locator" index:"8fb576e1-1855-4ceb-bdb9-f82320993833" ip:"10.0.40.10" tags:<key:"source_id" value:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:1072 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" eventType:ValueMetric timestamp:1552603141815612069 deployment:"service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" job:"server" index:"ff9638e0-864e-4740-bddc-dced52195dd1" ip:"10.0.40.16" tags:<key:"source_id" value:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:1060 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" eventType:ValueMetric timestamp:1552603141941405588 deployment:"service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" job:"locator" index:"febdbe54-4d25-433b-b830-f7f5d9d04f7d" ip:"10.0.40.12" tags:<key:"source_id" value:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:1078 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" eventType:ValueMetric timestamp:1552603142119222237 deployment:"service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" job:"locator" index:"debb3643-6eb6-470c-88f5-4afd6138bb9d" ip:"10.0.40.11" tags:<key:"source_id" value:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:1077 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" eventType:ValueMetric timestamp:1552603144791836263 deployment:"service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" job:"server" index:"c8a7c4ed-c9fe-4e3e-aa59-c27692590f4f" ip:"10.0.40.15" tags:<key:"source_id" value:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:1119 unit:"megabytes" >  
origin:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" eventType:ValueMetric timestamp:1552603157908057021 deployment:"service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" job:"server" index:"dac7efe6-714d-4768-95b0-cadda427530e" ip:"10.0.40.13" tags:<key:"source_id" value:"p-cloudcache.service-instance_a83cf6e1-7592-4b69-b8aa-2e6494686aca" > valueMetric:<name:"serviceinstance.UsedHeapSize" value:1010 unit:"megabytes" > 
```

The point of Step 5 is to show that when displaying metrics for PCC clusters, you will need to take into account that most PCC Clusters have multiple Locators and Cache-Servers, each one with their own sets of metrics. They will all share the same Service Instance Name.

## Step 6. Let's rename one of the PCC Service Instances without stopping it

```
$ cf rename-service dev-cluster new-name-dev-cluster
```
```
Renaming service dev-cluster to new-name-dev-cluster in org demo / space demo as admin...
OK
```

Let's check what happened:

```
$ cf s | (head -n 3; grep cloud)
```
```
Getting services in org demo / space demo as admin...

name                   service          plan          bound apps            last operation
ExtraSmallPCC          p-cloudcache     extra-small                         create succeeded
new-name-dev-cluster   p-cloudcache     dev-plan      pcc-lookaside-cache   update succeeded
SmallPCC               p-cloudcache     small                               create succeeded
```

From the results shown above, we can see that the `new-name-dev-cluster` is still bound to the `pcc-lookaside-cache`.
I also tested the actual App and it's running normally without the need for any restarts.

And, let's look at the relationship between names of Service Instances and GUIDs:

```
cf curl /v2/service_instances | jq '.resources[] .entity .name, .resources[] .metadata .guid'
```
```
"myVolume"
"autoscale-demo"
"mysql-dev"
"new-name-dev-cluster"
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

The fourth item of the list has a new name but the same old GUID. This makes sense given that it is essentially still the same PCC Cluster.

## Step 7. What you will need to augment the loggregator stream with the Service Instance Names

You will need to use either the output of Step 3:

```
$ cf curl /v2/service_instances | jq '.resources[] .entity .name, .resources[] .metadata .guid'
```
or the output of the following scripts to append the instance_name to the output of the `cf nozzle` output:

```
$ cf env pcc-lookaside-cache | tail -n +5 | awk -f x.awk 
f0dcd3e4-4645-466e-ac14-89686cb2c905
new-name-dev-cluster
```

where `x.awk` contains the following script:

```
{
  if (index($0,"p-cloudcache"))  { do getline; while (index($0,"gfsh")==0);
                                   starts=index($0,"-")+1;
                                   ends=index($0,".")-starts;
                                   print substr($0,starts,ends);
                                 }
  if (index($0,"instance_name")) { starts=index($0,":")+3;
                                   ends=index($0,"\",")-starts;
                                   print substr($0,starts,ends);
                                   exit 0;
                                 }

}
```

## Step 8. We didn't really fix the problem:

Correct, we didn't really fix the problem. In order to output Service Instance Names alongside metrics from the Loggregator, one would need to:

(a) constantly check whether the Service Instance Name has changed 

(b) use a GUID-to-Service-Instance-Name mapping table to look-up Service Instance Names based on GUIDs

(c) append the appropriate Service Instance Name to the output of `cf nozzle`

@jtuchscherer added this feature request to his to-dos: _"combine flow of service instance metrics and service instance logs"_ back on 11-JAN-2019 11:35AM. I'm awaiting his update any moment now.





