# Speed of Migration – What affects it and what to check? 

## The Three Biggest Factors That affects the Migration Speed: 
 

### 1. Network Bandwidth/MTU: 
Network bandwidth limits throughput for standard NBD copy operations. The migration process creates NBD servers for each disk and streams data over the network. So, the actual transfer speed depends on the available bandwidth between VMware and OpenStack environments. 
 
### MTU (Maximum Transmission Unit): The largest packet size allowed on a network path. 
- Larger MTU reduces packet overhead and can speed up migrations.  
- The MTU must be the same end-to-end. (the vjailbreak VMs NIC, the OpenStack networks and ports, the physical network path to vCenter ESXi and the storage should also use the same MTU) 
- Mismatched MTUs can cause failures. 
- Use the command: ip link show | grep mtu to verify the MTU values for network interfaces. 
- Command to verify the path MTU to the ESXi host:  
- Ping –M do –s 1473 <ESXi IP> 
- Ping –M do –s 8972 <ESXi IP> (Jumbo frames)
  
Even with the higher bandwidth network, the VDDK becomes the bottleneck: 

- VDDK can only read 2 MB chunks at a time through vCenter 
- Each chunk is compressed by fastlz protocol. 
- This creates a per-disk throughput limit regardless of available bandwidth. 
- Direct ESXi connections (bypassing vCenter) can achieve higher throughput with 23 MB read chunks. 
- While it's limited to 2 MB via vCenter. 
- So, we can choose direct ESXi connection for faster migration. 
 
 
### 2. Storage IOPS: 
Storage performance impacts both read operations from VMware datastores and write operations to OpenStack Cinder volumes.  
Speed is dependent on source storage reads as everything starts on VMware side vJailbreak pulls data from the datastores using VDDK/NBD. For incremental copies, it takes snapshots before each copy. If the datastore is busy or has lots of snapshots or busy with other VMs, the whole migration slows down. 
 
What could be done: 
- Using high IOPS Cinder volumes. 
- Scheduling migrations during off-peak hours. 
 

### 3. v2v-Helper Pod Resources (CPU/Memory): 
CPU and memory limits directly affect migration performance: 
The v2v-helper pods are where the actual migration work happens. They need enough CPU and memory to handle data compression, encryption, and managing the transfer process. 
V2v-helper pod runs NBD servers to stream disk data and uses virt-v2v for conversion. Undersized CPU/Memory can increase disk copy and conversion time. 
 
- We can increase the v2v-helper CPU and Memory request in the vjailbreak-settings configmap. 
- We should ensure that worker node or the master node has enough resource capacity for higher limits. 
 

 

## Data Copy Phase (Core factors affecting the speed of migration during data copy): 
 

### Number of Disks: 

Each additional disk on a VM introduces attach/detach overhead during migration. More disks mean: 

- Increased volume attachment time in OpenStack 
- Additional NBD server instances per disk 
- More disk conversion operations 
 

### Total Disk Size: 

Larger aggerated disk sizes extend the total copy time. 

 

 

### Migration Strategy: 

  Hot Migration: 

- Performs full initial copy plus repeated incremental syncs. 
- Uses CBT and continues until CBT threshold is reached. 
- VM remains running through the process. 
- More CBT deltas = More periodic sync data = Longer cutover window.
  
  Cold Migration: 

- Single full copy. 
- No incremental syncs needed. 
- Typically faster due to no ongoing changes. 
 

### Cutover: 

The migration controller supports multiple cutover modes: 

1. Admin-initiated: Waits for the manual trigger by the admin. 
2. Scheduled: Uses provided start and end timestamps. 
3. Immediate: Proceeds the cutover immediately after data copy. 
 

### vJailbreak Configuration Parameters: 

1. ChangedBlocksCopyIterationThreshold: 
  - Default is 20. 
  - Controls maximum incremental syncs. 
  - Prevents infinite incremental copies. 
  - Lower value = Early cutover. 

2. PeriodicSyncInterval: 
  - Default is 1hour. 
  - Frequency of incremental sync attempts. 

3.Retry Limits: 
  - This controls failure recovery behavior. 
  - Includes backoff intervals 
 

Other components from the vjailbreak-settings configmap also impacts the speed of migration directly or indirectly. 

 

### Other factors: 
OpenStack API Issues Affecting Migration Speed: 

  1. Cinder API: 

- Volume creation delays: Adds 1GB extra to the volume and waits for volume availability. 
- Volume availability: Cause delays if Cinder is slow to mark volumes as available. 
- Volume attachment verification: Dual-checks volume status from both Cinder and Nova. perspectives, adding extra API calls. 
  
  2. Nova API: 

- VM activation delays. 
- Flavor selection overhead. 
- Port creation delays. 

  3. Authentication: 

- Authentication retries: vCenter login retry limit affects connection establishment. 
 

4.Snapshots: 

- Snapshots play a critical role in hot migrations and directly influence disk read performance during the migration process. 

- When a snapshot is created, VMware converts the base disk into a read only state and starts writing all new changes to a delta file.  

- If multiple snapshots exist (snapshot chain), read operations become slower because VMware may need to traverse several delta files before reaching the base disk. Deep snapshot chains can significantly reduce effective disk throughput. 

- We can clean up unnecessary VMware snapshots so vjailbreak reads from simple, fast disk layout. 

 
5. Concurrency (Parallel Migrations): 

- Running multiple migrations at the same time directly impacts per-VM migration speed. 


6. Encryption Overhead: 

- If TLS/IPsec is used between environments: CPU cost can reduce throughput 10–40% 

 
7. Stable DNS and name resolution: 

- vJailbreak needs to resolve ESXi host names during copy. Failure or delays in that may delay NBD startup. 

 
8. Time sync: 

- Ensure vCenter, ESXi and vJailbreak clocks are in sync to avoid snapshot/CBT quirks. Although time sync doesn’t directly throttle data copy speed but it can delay scheduled steps in an ongoing migration, indirectly extending total time.
