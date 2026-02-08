
1. Title

**vjailbreak-vm-migration-workflow.**

2. About the Tool

vjailbreak is a open-source migration tool created by platform9.
It helps organizations migrate VMs from VMware to an OpenStack compatable cloud or PCD with less complexity and downtime.

--> vmware escape tool.

3. Objective
 	
This case study documents my learning and hands-on experience performing a virtual machine migration using the vjailbreak tool. 
The objective was to understand the tool’s architecture and observe real-world infrastructure transition workflows, including appliance bootstrapping, disk migration, network and storage mapping, and post-migration validation while moving workloads 
from VMware ESXi to an OpenStack environment. Rather than relying solely on documentation, I focused on analyzing internal workflows, identifying failure points, and troubleshooting issues encountered during the migration process.

4. High-level overview:
	- Connects to the source VMware vCenter / ESXi environment to discover virtual machines.
	- Lets us select VMS to migrate.
	- Converts VMware disk formats (VMDK) into OpenStack-compatible formats such as QCOW2 or RAW.
	- Recreates the VMS in PCD/openstack with correct CPU, RAM, Network and storage.
	- Automates the migration flow.

5. vjailbreak Migration Architecture (Code-Level Understanding)
  Modeule 01: Project orientation vjb VM
	- vjailbreak is designed as a self-contained migration appliance. There is a QCOW2 image to deploy the appliance.
	- It migrates VMs from VMware vCenter to OpenStack(KVM)
	- vjailbreak VM runs a k3s cluster which runs the migration components.
	- Core Components:
		- K3s Kubernetes Cluster: The foundation of vJailbreak is a lightweight K3s Kubernetes distribution that runs entirely within the VM . Key components include:
			- K3s Server: Master node Kubernetes server
			- containerd: Container runtime
			- etcd: Kubernetes state database
			- kubeconfig: Cluster authentication
		- Application Components
			vJailbreak UI
				A web interface for managing migrations, deployed as a Kubernetes Deployment 
			Migration Controller
				The core orchestration component that manages migration workflows:
				Manages custom resources (VjailbreakNode, MigrationPlan, etc.)Handles worker node provisioningCoordinates migration execution
			v2v-helper Pods
				Execute the actual VM conversion using virt-v2v:
				Convert VMDK to QCOW2 formatInject virtio drivers for WindowsHandle disk transfer and conversion
			Custom Resources
		
				vJailbreak defines several Kubernetes custom resources:
				OpenstackCreds: This CRD stores connection and authentication details for OpenStack/PCD environment
				VMwareCreds: This CRD stores vCenter/VMware authentication and connection details.
				VMwareMachines: Discovered representation of VMs
				NetworkMapping: Maps VMware network to OpenStack.
				StorageMapping: Maps VMware datastores to OpenStack storage options.
				VjailbreakNode: Represents migration infrastructure nodes
				MigrationTemplate: Reusable migration configurations
				MigrationPlan: Defines migration workflows
				Migration: Actual execution of migration.

	- Initialization Process
		First Boot Sequence
			- Cron Job Trigger: /etc/cron.d/vjailbreak-firstboot executes the script at /etc/pf9/install.sh
			- K3s Installation: Initializes single-node cluster
			- Image Import: Loads pre-loaded container images
			- Component Deployment: Applies Kubernetes manifests
			- Service Start: Starts all migration components

	- Critical Directories and Files
		Configuration Directories
			/etc/pf9/: Core configuration and scripts
			k3s.env: K3s cluster configuration
			k3s/token: Cluster join token
			images/: Pre-loaded container images deployment
			htpasswd: Web UI authentication
		Migration-Specific Paths
			/home/ubuntu/: User workspace
			vmware-vix-disklib-distrib/: VDDK libraries (must be uploaded on vjailbreak VM at /home/ubuntu)
			virtio-win/: Windows drivers (auto-downloaded)
			/var/log/pf9/: Migration logs and debugging output

	- Network Configuration
		
		Primary Interface: Configured via DHCP or static IP
		DNS Resolution: Critical for VMware connectivity - must resolve vCenter and all ESXi hosts README.md:161-179Required Connectivity
		The VM needs network access to:
		VMware: vCenter (443), ESXi hosts (902, 22)OpenStack: Keystone, Nova, Cinder, Neutron APIs
		Internet: For initial driver downloads.
			
				




