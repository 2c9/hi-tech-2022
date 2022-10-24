### Install OVS
```bash
dnf install -y epel-release centos-release-openstack-train
dnf install -y openvswitch NetworkManager-ovs
systemctl enable --now openvswitch.service
```

## Add OVS bridge

```bash
ovs-vsctl add-br ovsbr0
```

```xml
### ovs.xml
<network>
  <name>ovs-network</name>
  <forward mode='bridge'/>
  <bridge name='ovsbr0'/>
  <virtualport type='openvswitch'/>
  <portgroup name='vlan1' default='yes'>
  </portgroup>
  <portgroup name='vlan2'>
    <vlan>
      <tag id='2'/>
    </vlan>
  </portgroup>
  <portgroup name='vlan-all'>
    <vlan trunk='yes'>
      <tag id='10'/>
      <tag id='20'/>
    </vlan>
  </portgroup>
</network>
```

```bash
virsh net-define --file ovs.xml
virsh net-start ovs-network
virsh net-autostart ovs-network
```

## Edit FW1

```bash
virsh shut FW1
virsh attach-interface --domain FW1 --type network --source ovs-network --model virtio --config
virsh edit FW1
```

```xml
<source network='ovs-network' portgroup='vlan1'/>
```

Start FW1

```bash
virsh start FW1
```

## Clone DB1 and DB2 from out Centos template

```bash
virt-clone --original tmpl-centos --name DB1 --auto-clone
virt-clone --original tmpl-centos --name DB2 --auto-clone
```

## Chnage network interfaces (other network)

```bash
virsh edit DB1
virsh edit DB1
```

## Resize disks

```bash
qemu-img info /var/lib/libvirt/dir_pool/centos-clone-1.qcow2
qemu-img resize /var/lib/libvirt/dir_pool/centos-clone-1.qcow2 +10G
qemu-img resize /var/lib/libvirt/dir_pool/centos-clone-2.qcow2 +10G
```

## Start VMs

```bash
virsh start DB1
virsh autostart DB1
virsh start DB2
virsh autostart DB2
```