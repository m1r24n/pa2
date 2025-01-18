# introduction

This document provides guideline on how to install paragon automation VM on KVM hypervisor.

the Hypervisor is Ubuntu Linux 24.04 with openswitch for VM Connection.


# convert VMDK to qcow2

    tar xpvf paragon-2.3.0.ova
    qemu-img convert -O qcow2 -c paragon-2.3.0-disk1.vmdk paragon-2.3.0-disk1.qcow2
    qemu-img convert -O qcow2 -c paragon-2.3.0-disk2.vmdk paragon-2.3.0-disk2.qcow2

# create disk on KVM

    #!/bin/bash
    SRC_DISK1=/disk3/paragon/paragon-2.3.0-disk1.qcow2
    SRC_DISK2=/disk3/paragon/paragon-2.3.0-disk2.qcow2

    for i in node{0..3}
    do
        mkdir ${i}
        qemu-img create -f qcow2 -F qcow2 -b ${SRC_DISK1} ${i}/paragon-2.3.0-disk1.qcow2
        qemu-img create -f qcow2 -F qcow2 -b ${SRC_DISK1} ${i}/paragon-2.3.0-disk2.qcow2
    done

# create VM on KVM

    #!/bin/bash
    for VM in node{0..3}
    do
        DISK1=${VM}/paragon-2.3.0-disk1.qcow2
        DISK2=${VM}/paragon-2.3.0-disk2.qcow2
        VLAN=111
        LAN=ovs0
        virt-install --name ${VM} \
        --disk ./${DISK1},bus=scsi,device=disk \
        --disk ./${DISK2},bus=scsi,device=disk \
        --ram 32768  --vcpu 8  \
        --osinfo ubuntu22.04 \
        --network=bridge:${LAN},model=virtio,virtualport_type=openvswitch \
        --xml "./devices/interface/vlan/tag/@id=${VLAN}" \
        --xml "./devices/interface/target/@dev=${VM}_e0" \
        --console pty,target_type=serial \
        --noautoconsole \
        --hvm --accelerate  \
        --graphics vnc,listen=0.0.0.0 \
        --virt-type=kvm  \
        --import \
        --boot hd
    done

# configuration
    configure
    set paragon cluster nodes kubernetes 1 address 192.168.111.100
    set paragon cluster nodes kubernetes 2 address 192.168.111.101
    set paragon cluster nodes kubernetes 3 address 192.168.111.102
    set paragon cluster nodes kubernetes 4 address 192.168.111.103
    set paragon cluster ntp ntp-servers 0.pool.ntp.org
    set paragon cluster common-services ingress ingress-vip 192.168.111.200
    set paragon cluster applications active-assurance test-agent-gateway-vip 192.168.111.201
    set paragon cluster applications web-ui web-admin-user "irzan@juniper.net"
    set paragon cluster applications web-ui web-admin-password "J4k4rt4#170845"
    set paragon cluster applications pathfinder pce-server pce-server-vip 192.168.111.202
    commit
    exit

# installation
    request paragon config
    request paragon ssh-key
    request paragon deploy cluster input "-e ignore_iops_check=yes"



