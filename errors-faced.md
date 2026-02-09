Common Migration Issues I Faced in the Lab

Network Configurations:
  The first thing that hit me was network configuration. After migrating VMs from VMware to OpenStack, they'd lose network connectivity completely. I discovered this happens because network interfaces get new names in OpenStack.
  I found that vJailbreak tries different approaches based on the Linux distribution. For Ubuntu 17.10+, it uses netplan configurations. For older systems, it falls back to udev rules. 
  
  What I learned: Always check if network persistence is enabled. The system logs warnings when it can't get network interface names migrate.go:1184-1186 .

Storage Mapping Headaches:
  Another frequent issue was storage mapping failures. The system would throw errors like "VMware Datastore(s) not found in StorageMapping" when I tried to migrate VMs.
  The controller checks if each VMware datastore has a corresponding target volume type, and fails if any are missing.

fstab Entries Not Populating:
  I ran into situations where disks wouldn't mount after migration. The fstab file still had old device names instead of UUIDs.
  When this fails, you'll see warnings in the logs about failed mount persistence script execution.

Network Mapping Validation Errors:
  I also saw "VMware network not found in NetworkMapping" errors. This occurs when the system tries to map each VM's network to a target network but can't find a match.

Volume Creation Timeouts:
  I encountered situations where volume creation would timeout. The system waits for volumes to become available within configurable intervals and retry limits.
  When Cinder is slow to provision volumes, these timeouts can trigger migration failures.

Notes:
These issues taught me to always validate network and storage mappings before starting migrations, and monitor the migration logs closely for warnings about network configuration and fstab updates. 
The system provides detailed logging that helps identify where things go wrong. Also these are just few errors/issues, there could be multiple issues in production migraions.

