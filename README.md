# Nutanix external inventory script for Ansible
## Overview
This script generates inventory that Ansible can understand by making API requests to one or more Nutanix clusters.

It assumes there is a nutanix.yml file alongside it. To specify a different path to nutanix.yml, define the NUTANIX_YML_PATH environment variable:

```bash
export NUTANIX_YML_PATH=/path/to/nutanix.yml
```

This script requires that all VM names and IP addresses be unique across clusters, and will fail otherwise. This is to prevent VMs with identical names or IP addresses from overwriting each other in the inventory.

Caching is implemented in order to reduce traffic and speed up inventory results. The length of time cache files are kept is determined by `max_cache_age` in nutanix.yml.

You can also force an update of the cache with the `--refresh-cache` switch.

For each host, the following variables are registered:
 - acropolisVm
 - clusterUuid
 - consistencyGroupName
 - containerIds - list
 - containerUuids - list
 - controllerVm
 - cpuReservedInHz
 - diskCapacityInBytes
 - displayable
 - fingerPrintOnWrite
 - guestOperatingSystem
 - hostId
 - hostName
 - hostUuid
 - hypervisorType
 - ipAddresses - list
 - memoryCapacityInBytes
 - memoryReservedCapacityInBytes
 - numNetworkAdapters
 - numVCpus
 - nutanixGuestTools - object
 - nutanixVirtualDiskIds - list
 - nutanixVirtualDiskUuids - list
 - nutanixVirtualDisks - list
 - onDiskDedup
 - powerState
 - protectionDomainName
 - runningOnNdfs
 - stats - object
 - usageStats - object
 - uuid
 - vdiskFilePaths - list
 - vdiskNames - list
 - virtualNicIds - list
 - virtualNicUuids - list
 - vmId
 - vmName

You can run against a specific host by using `--host` and specifying either the VM name or IP address.

When run in `--list` mode, which is the default mode when Ansible calls this script, VMs are enumerated by their IP (note that VMs without assigned IPs will not be output) and grouped according to the following:

- Name of their respective cluster.
- Any groups specified in the VM description field.
- The _meta group, with the above variables registered plus any hostvars specified in the VM description field.

When run in `--names` mode, or when environment variable `NUTANIX_MODE=names`, VMs are enumerated by their names and grouped according to the following:

- Name of their respective cluster.
- Power state.
- Any groups specified in the VM description field.
- The _meta group, with the above variables registered plus any hostvars specified in the VM description field.

Ansible can use this mode on the command line as in the following:

```bash
NUTANIX_MODE=names ansible-playbook -i nutanix.py myplaybook.yml
```

A VM description should look something like one of the following:

```json
{"groups":["tomcat"]}
{"groups":["texas","pd","tomcat"]}
{"groups":["texas","pd","nfs"],"hostvars":{"foo":"bar","boo":"far"}}
```

VM descriptions can also be enclosed in quotes, which can be useful if using a script to pass descriptions as strings when creating or updating VMs.

VMs whose descriptions are not in JSON will not be grouped according to any description field information, though they will still be grouped according to cluster name and, if listed by name, power state.

## Configuration
Configuration is pretty straightforward. An example nutanix.yml file will look something like this:

```yaml
# Cluster settings
clusters:
    prod-cluster:
        address: prod-cluster.example.com
        port: 9440
        username: admin
        password: pass123
        verify_ssl: False
    qa-cluster:
        address: qa-cluster.example.com
        port: 9440
        username: admin
        password: pass123
        verify_ssl: False

# Global caching settings
caching:
    cache_max_age: 300
	cache_path: '/tmp/'
	cache_base_name: 'ansible-nutanix.cache'
```

The address, username, and password fields are all required. The port will default to 9440 if not specified, and verify_ssl will default to True if not specified. Caching settings are also required, though cache_max_age can be set to 0 if you don't want to use cache files.
