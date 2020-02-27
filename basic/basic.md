```
cat > /etc/libvirt/qemu/template.xml   << EOF
<domain type='kvm'>

  <name>template</name>
  <uuid>f4ea90d5-cf25-411c-bdf3-33c2fef8f3a5</uuid>
  <memory unit='MiB'>mem-size</memory>
  <currentMemory unit='MiB'>mem-size</currentMemory>
  <vcpu placement='static'>core-num</vcpu>




  <os>
    <type arch='x86_64' machine='pc-i440fx-rhel7.0.0'>hvm</type>
    <boot dev='hd'/>
  </os>
  <features>
    <acpi/>
    <apic/>
  </features>
  <cpu mode='host-model' check='partial'>
    <model fallback='allow'/>
  </cpu>
  <clock offset='utc'>
    <timer name='rtc' tickpolicy='catchup'/>
    <timer name='pit' tickpolicy='delay'/>
    <timer name='hpet' present='no'/>
  </clock>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>destroy</on_crash>
  <pm>
    <suspend-to-mem enabled='no'/>
    <suspend-to-disk enabled='no'/>
  </pm>
  <devices>
    <emulator>/usr/libexec/qemu-kvm</emulator>



    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2'/>
      <source file='/kvm/image/template.qcow2'/>


      <target dev='vda' bus='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x07' function='0x0'/>
    </disk>


    <disk type='file' device='cdrom'>
      <driver name='qemu' type='raw'/>
      <target dev='hda' bus='ide'/>
      <readonly/>
      <address type='drive' controller='0' bus='0' target='0' unit='0'/>
    </disk>
    <controller type='usb' index='0' model='ich9-ehci1'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x7'/>
    </controller>
    <controller type='usb' index='0' model='ich9-uhci1'>
      <master startport='0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x0' multifunction='on'/>
    </controller>
    <controller type='usb' index='0' model='ich9-uhci2'>
      <master startport='2'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x1'/>
    </controller>
    <controller type='usb' index='0' model='ich9-uhci3'>
      <master startport='4'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x2'/>
    </controller>
    <controller type='pci' index='0' model='pci-root'/>
    <controller type='ide' index='0'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x1'/>
    </controller>
    <controller type='virtio-serial' index='0'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/>
    </controller>
    <interface type='network'>
      <mac address='52:54:a4:a9:5c:43'/>
      <source network='br0'/>
      <model type='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
    </interface>
    <serial type='pty'>
      <target type='isa-serial' port='0'>
        <model name='isa-serial'/>
      </target>
    </serial>
    <console type='pty'>
      <target type='serial' port='0'/>
    </console>
    <channel type='spicevmc'>
      <target type='virtio' name='com.redhat.spice.0'/>
      <address type='virtio-serial' controller='0' bus='0' port='2'/>
    </channel>
    <channel type='unix'>
      <target type='virtio' name='org.qemu.guest_agent.0'/>
      <address type='virtio-serial' controller='0' bus='0' port='3'/>
    </channel>
    <input type='tablet' bus='usb'>
      <address type='usb' bus='0' port='1'/>
    </input>
    <input type='mouse' bus='ps2'/>
    <input type='keyboard' bus='ps2'/>
    <graphics type='vnc' port='-1' autoport='yes' keymap='en-us'>
      <listen type='address'/>
    </graphics>
    <sound model='ich6'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
    </sound>
    <video>
      <model type='qxl' ram='65536' vram='65536' vgamem='16384' heads='1' primary='yes'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
    </video>
    <redirdev bus='usb' type='spicevmc'>
      <address type='usb' bus='0' port='2'/>
    </redirdev>
    <redirdev bus='usb' type='spicevmc'>
      <address type='usb' bus='0' port='3'/>
    </redirdev>
    <memballoon model='virtio'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x08' function='0x0'/>
    </memballoon>
  </devices>
</domain>
EOF
```


```
cat /root/create_vm.sh 
#!/bin/bash
images_dir=/kvm/image
xml_dir=/etc/libvirt/qemu

read -p "请输入新建虚拟机名称: " new_kvm

if [  -f "${xml_dir}/${new_kvm}.xml" ];then
        echo "${new_kvm} 已存在,部署程序退出..."
            exit 1
fi

if [  -f "${images_dir}/${new_kvm}.qcow2" ];then
                echo "${new_kvm}.qcow2  已存在,部署程序退出..."
                exit 1
fi


virsh vol-clone  CentOS7.6.1810.x86_64.qcow2  --pool kvmimage ${new_kvm}.qcow2


read -p "请输入新建虚拟机磁盘大小单位G: " disk_size


echo "开始创建虚拟磁盘"
virsh  vol-resize  ${new_kvm}.qcow2  --pool kvmimage  --capacity $disk_size'G'


read -p "请输入新建虚拟机内存大小单位M   : " mem_size

read -p "请输入新建虚拟机cpu核心数 : " core_num







################配置虚拟机模板#######################################################################################################################################################################################
cp ${xml_dir}/template.xml   ${xml_dir}/${new_kvm}.xml
sed  -i  "s#.*:.*:.*:.*:.*:.*#<mac address='`MACADDR="52:54:$(dd if=/dev/urandom count=1 2>/dev/null | md5sum | sed -r 's/^(..)(..)(..)(..).*$/\1:\2:\3:\4/')"; echo $MACADDR`'/>#g"   ${xml_dir}/${new_kvm}.xml
sed    -i  "s#<uuid>.*-.*-.*-.*-.*</uuid>#<uuid>`uuidgen`</uuid>#g"      ${xml_dir}/${new_kvm}.xml 
sed -i     "s#template#${new_kvm}#g"           ${xml_dir}/${new_kvm}.xml
sed    -i  "s#mem-size#$mem_size#g"      ${xml_dir}/${new_kvm}.xml
sed    -i  "s#core-num#$core_num#g"      ${xml_dir}/${new_kvm}.xml
#######################################################################################################################################################################################################################



virsh define   ${xml_dir}/${new_kvm}.xml  


virsh autostart ${new_kvm}

                echo "创建成功"



read -p "请输入为虚拟机配置的ip: " new_kvm_ip

mkdir /tmp/${new_kvm}

echo "NAME=eth0"   >               /tmp/${new_kvm}/ifcfg-eth0
echo "DEVICE=eth0" >>              /tmp/${new_kvm}/ifcfg-eth0
echo "ONBOOT=yes" >>               /tmp/${new_kvm}/ifcfg-eth0
echo "TYPE=Ethernet" >>    /tmp/${new_kvm}/ifcfg-eth0
echo "BOOTPROTO=none" >>   /tmp/${new_kvm}/ifcfg-eth0
echo "IPADDR=${new_kvm_ip}"   >>    /tmp/${new_kvm}/ifcfg-eth0
echo "NETMASK=255.255.255.0"  >>   /tmp/${new_kvm}/ifcfg-eth0
echo "GATEWAY=172.16.99.250"   >>   /tmp/${new_kvm}/ifcfg-eth0
echo "DNS1=223.5.5.5"   >>   /tmp/${new_kvm}/ifcfg-eth0

cat  /tmp/${new_kvm}/ifcfg-eth0

read -p "请确认您的网卡配置，按y键继续?[y/n]: " eth0_config
    if [ ! "${eth0_config}" = "y" ];then
        echo "输入错误!"
            exit 1
    fi


echo "开始配置$new_kvm 的网卡，请稍后......."

virt-copy-in   -d     $new_kvm        /tmp/${new_kvm}/ifcfg-eth0    /etc/sysconfig/network-scripts/

virsh start  $new_kvm   1>/dev/null

echo "正在启动虚拟机请稍后....."

sleep 15s

echo "虚拟机启动成功，登录赋值如下命令即可  ssh ${new_kvm_ip}"

rm -rf /tmp/${new_kvm}  2>/dev/null
```
