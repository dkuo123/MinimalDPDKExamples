After cloning the crypto library, run scripts/setup_dpdk.sh and enter the sudo password if needed if script asks for it while running

Add following to the ~/.bashrc: export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib64

Add following to the ~/.bashrc: PKG_CONFIG_PATH=/usr/local/lib64/pkgconfig 

Run source ~/.bashrc to activate these (after this point, our Crypto library will build, but to run DPDK programs the remaining steps are required)

To enable iommu mode for using the vfio-pci kernel module, add iommu=1 intel_iommu=on to the [bootloader] parameters in /usr/lib/tuned/realtime/tuned.conf and reboot the box. After the reboot, /sys/kernel/iommu_groups/ should not be empty if iommu has been properly enabled. 

Pick an interface to use DPDK on. Note down the MAC address of the default gateway of the interface (which can be found using arp -a). 

Bring down the interface using ifconfig (e.g., ifconfig down eth1). 

Load the vfio-pci module using sudo modprobe vfio-pci

Identify the PCI address of the interface to use DPDK on (e.g., dpdk-devbind.py --status) ; the PCI address will look like for example 00:06.0

Using the PCI address, bind the interface to vfio-pci kernel module (e.g., dpdk-devbind.py --bind=vfio-pci 00:06.0)

Verify that the setup was successful by doing dpdk-devbind.py --status and validating that the interface is now using a DPDK supported driver

Enable permissions for certain files/directories (this is needed so our DPDK programs can without sudo): sudo chmod og+w /dev/hugepages , sudo chown archy /dev/vfio/vfio , sudo chown archy /dev/vfio/*

The noted down MAC address can be used to set the gateway_mac_addr in the config of our DPDK programs

Until the receive side of DPDK is implemented, we need to use address spoofing to make sure our outgoing DPDK traffic makes it to the receivers (ask Viraj to turn off source/destination check on AWS box, shouldn’t require a reboot)

Relevant links:

DPDK documentation — Data Plane Development Kit 21.11.2 documentation 

GitHub - NEOAdvancedTechnology/MinimalDPDKExamples: Minimal examples of DPDK 
