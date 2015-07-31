# VirtualBox provider for Terraform

Inspired by [terraform-provider-vix](https://github.com/hooklift/terraform-provider-vix)

# How to build and install

1. Make sure you've got `goop` installed, we are using goop to lock the version of dependencies. `go get -v github.com/nitrous-io/goop`
2. `git clone https://github.com/ccll/terraform-provider-virtualbox.git`
3. Run `goop install` in the cloned repo to install all dependencies.
4. `goop go build` to build this plugin.
5. `goop go install` to install the plugin binary at `.vendor/bin/terraform-provider-virtualbox`.
6. Copy `terraform-provider-virtualbox` binary to the same directory as your `terraform` binary.

# Resources

## "virtualbox_vm"

### Attributes

- `name`, string, required: The name of the virtual machine.
- `image`, string, required: The url of the image file.
- `cpus`, int, optional, default=2: The number of CPUs.
- `memory`, string, optional, default="512mib": The size of memory, allow human friendly units like 'MB', 'MiB'.
- `status`, string, optional, default="running": The status of the VM, allowed values: 'poweroff', 'running'. This value will be updated at runtime to reflect the real status of the VM, and you can also specify it explicitly in config to manually control the status of the VM. This value defaults to 'running', so `terraform apply` will always try to keep the VM running if not specified otherwise.
- `network_adapter` - list: The network adapters in the VM, you can have up to 4 adapters.
  - `.#.type`, string, requried: The type of the network, allowed values: 'nat', 'bridged', 'hostonly', 'internal', 'generic'.
  - `.#.device`, string, optional, default="IntelPro1000MTServer": The type of the virtual hardware device, allowed values: 'PCIII', 'FASTIII', 'IntelPro1000MTDesktop', 'IntelPro1000TServer', 'IntelPro1000MTServer'.
  - `.#.host_interface`, string, optional: Some network type (hostonly, bridged, etc) must bind to a host interface to work properly, use this field to specify the name of the host interface.
  - `.#.status`, string, computed: The status of the network adapter, possible values: 'up', 'down'.
  - `.#.mac_address`, string, computed: The MAC address of the adapter, this is generated by VirtualBox.
  - `.#.ipv4_address`, string, computed: The IPv4 address assigned to the adapter.
  - `.#.ipv4_address_available`, string, computed: Wheather or not an IPv4 address is actaully assigned to the adapter, possible values: "yes", "no".

### Network adapter types
- [x] NAT
- [x] bridged
- [ ] Hostonly  (not tested, probably can work)
- [ ] Internal  (not tested)
- [ ] Generic  (not tested)

# Example

```
resource "virtualbox_vm" "node" {
    count = 2
    name = "${format("node-%02d", count.index+1)}"

    image = "~/ubuntu-15.04.tar.xz"
    cpus = 2
    memory = "512mib"

    network_adapter {
        type = "nat"
    }

    network_adapter {
        type = "bridged"
        host_interface = "en0"
    }
}

output "IPAddr" {
    # Get the IPv4 address of the bridged adapter (the 2nd one) on 'node-02'
    value = "${element(virtualbox_vm.node.*.network_adapter.1.ipv4_address, 1)}"
}

```

# Limitations

- Only local images is allowed, for now.

# Example images

- [ubuntu-15.04](https://github.com/ccll/terraform-provider-virtualbox-images/releases/tag/ubuntu-15.04)

# How to make an image

- An image file is a tarball file in format '.tar.gz', '.tar.bz2' or '.tar.xz'.
- An image tarball should contain atleast one virtual disk files, for now only ".vdi' and '.vmdk' is supported. You can run 'VBoxManage clonehd' to convert formats.
- '.vbox' files is ignored as we are creating a brand new VM instead of cloning from existing one, so you can avoid packing them into the image. There might be a small chance your image will not work in the newly created VM if some spec flags varies greatly, but as long as you make your image in a VM created with default flags, things should work smoothly.
- All virtual disk files in the image will be attached to the same SATA controller in **alphabet** order, so name them properly before making the tarball.
- VirtualBox Guest Addition **must** be installed and running in the guest OS, as the IP address is retrieved via it. If you have a better approach which does not require Guest Addition, please write to me, or better, send a PR.

# TODO

- [ ] Auto download image from remote url.
- [ ] Validate downloaded image against checksum.
- [ ] Download the same image only once (based on checksum).
- [ ] Re-download corrupted image (based on checksum).
 
