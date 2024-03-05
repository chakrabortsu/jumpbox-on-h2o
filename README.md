# Jumpbox
Automation to create a jumpbox or bastion host in an H2O environment.

The jumpbox is a prerequisite for other H2O platform installation
helpers. If you already have the H2O CLI installed and working for your H2O
slot you can skip directly to the `Create Jumpbox` section below otherwise
[install and configure the H2O CLI](../docs/h2o-cli-install.md) before
continuing.

## Create Jumpbox
This script assumes you're running it from a Mac or Linux workstation connected
to the VMware VPN. Before running the jumpbox configure or install script ensure
you have [govc], [ytt], and [h2o-CLI] installed and available on your PATH.

> IMPORTANT - Ensure you have the [H2O CLI installed and properly configured](../docs/h2o-cli-install.md) before continuing. 

To create a jumpbox run the following commands, using your actual H2O slot id:

```sh
./configure.sh <h2o-slot-id> && ./install.sh
```

For example:
```sh
./configure.sh h2o-2-18562 && ./install.sh
```

This will download the latest Ubuntu Focal OVA and spin up a jumpbox VM in
vSphere at the first IP in the user-workload network.

## SSH to Jumpbox

To SSH into the jumpbox use the key that the install script generated in the
`jumpbox/.ssh` directory. The jumpbox IP can be found in the `jumpbox.config`
file.

```bash
$ ssh -i .ssh/id_rsa ubuntu@yourjumpboxip
```

## Advanced Configuration
The `configure.sh` script generates a complete valid jumpbox.config by reading
the environment details from the H2O API, however you may want to override
those values. To customize any of the settings, execute the configure and
the install scripts separately.

```sh
./configure.sh <h2o-slot-id>
```

Edit the values as needed in the generated jumpbox.config.

```bash
# Required
slot_password='supersecret'
h2o_domain='h2o-11-255.h2o.vmware.com'
jumpbox_ip='10.220.41.199'
jumpbox_gateway='10.220.41.222'

# Optional - overrides defaults
jumpbox_netmask='255.255.255.224'
jumpbox_dns='10.220.136.2,10.220.136.3'
vm_name='jumpbox'
vm_network='user-workload'
root_disk_size='50G'
datastore='vsanDatastore'
ram='2048'
```

- `slot_password` is the global password for the H2O slot.
- `h2o_domain` is the domain suffix for the slot.
- `jumpbox_ip` is the IP address of the jumpbox.
- `jumpbox_netmask` is the network mask used by the jumpbox.
- `jumpbox_gateway` is the network gateway used by the jumpbox.
- `jumpbox_dns` is the comma delimited list of DNS servers used by the jumpbox.
- `vcenter_host` is the vCenter host name that is used by govc to spin up the jumpbox.
- `vm_name` is the VM name, by default jumpbox.
- `vm_network` is the network name the jumpbox is attached to, by default user-workload.
- `root_disk_size` is the size of the jumpbox HDD, by default 50G.
- `datastore` is the vCenter datastore name, by default vsanDatastore.
- `ram` is the amount of RAM to give the jumpbox, by default this is 8192 (8G).

After completing your edits, run the install script:
```sh
./install.sh
```

## IP Range Notes
In a `vsphere_avi_wcp` slot you will need to skip the first 4 IPs. The first one
is used by a network appliance, second by WCP, and third (unpingable) by an AVI
SE. For example given a `user-workload` network of `10.214.178.96/27`:
- 10.214.178.97 - First IP, but something is pingable probably a network appliance.
- 10.214.178.98 - Used by the supervisor cluster (WCP).
- 10.214.178.99 - Bound to an Avi SE, but not pingable.
- 10.214.178.100 - First usable IP.

[govc]: https://github.com/vmware/govmomi/releases
[ytt]: https://github.com/vmware-tanzu/carvel-ytt/releases
[h2o-CLI]: https://build-artifactory.eng.vmware.com/ui/native/h2o-local/cli/2.2.0/
