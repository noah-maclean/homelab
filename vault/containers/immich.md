# Immich

## Setup

- run [Proxmox helper script](https://community-scripts.org/scripts/immich) with advanced settings
	- 2 cores
	- 4 GB RAM
	- 20 GB HDD
	- GPU passthrough automatically detected
- IP: 192.168.1.23

- during setup, compilation of one dependency failed, so retried with pihole blocking disabled
###  Result

- immich container has 20 GB from the SSD and folder on the 1 TB HDD used to store photos