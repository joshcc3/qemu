I get no multicast data from any of them. Do you know why I'm not getting multicast data from any of the interfaces but I do get when I set the interface to INADDR_ANY?

When you specify INADDR_ANY in the ip_mreq structure for multicast membership, you're telling the operating system to choose the default interface for multicast traffic. This is often what you want for a simple setup where the machine may have several interfaces but you want to listen for multicast packets on all of them or on the "default" one that the routing table indicates should be used for multicast.

The behavior you're observing where setting imr_interface.s_addr to an interface's IP address doesn't work while INADDR_ANY does might be due to a few reasons:

Routing Table Misconfiguration:
Multicast packets are routed based on the system's routing table. The routing table must have proper routes set up for multicast traffic. When using INADDR_ANY, the system's routing logic automatically picks the right interface based on the routing table and the source of the packets. When you specify an interface directly by setting an IP address, you bypass this logic, and if the routing table isn't set up correctly for this scenario, packets won't be received.

Interface Support:
Not all interfaces support multicast traffic equally. For example, the lo (loopback) interface can be used for multicast, but only for traffic originating from the same machine. In contrast, your wireless interface wlp0s20f3 might be the one that actually connects to the network where multicast packets are being distributed.

Firewall Rules:
Some systems have firewall rules that affect how multicast traffic is received. These rules might be set up to allow such traffic only on certain interfaces.

Scope of Multicast Address:
The multicast address used also has a scope. For example, the 239.255.0.1 is in the Organization-Local Scope (addresses from 239.255.0.0 to 239.255.255.255), which means it's meant to be used within the same organization or site, and routers generally do not forward these packets beyond their local subnets.

Application Binding to Specific IP:
If an application binds to a specific IP address (like one from your wlp0s20f3 interface), it will only receive multicast packets on that interface. In contrast, binding to INADDR_ANY will listen on all interfaces, and the kernel joins the multicast group on the default interface unless you tell it otherwise with additional socket options or routing table configurations.

To ensure that you receive multicast traffic on a specific interface when not using INADDR_ANY, you should:

Make sure your routing table is configured to route multicast traffic through that interface.
Confirm that there's no firewall blocking the traffic.
Use setsockopt with IP_MULTICAST_IF to set the outbound interface for multicast packets if you're sending data.
If your machine has multiple interfaces and you're joining a multicast group on an interface that isn't the default one for multicast, you need to ensure that you've set up your multicast routes accordingly.
Here's how to specify an interface with IP_MULTICAST_IF:


The routing table uses the default one, what's the default interface and why wouldn't it be one of the ones in ifconfig.
How can I get the routing table.
What are the different scopes for multicast addresses?
What is the default interface?

set IP_MULTICAST_IF

When you're dealing with multicast on a network socket in a Unix-like operating system, there are several aspects to consider to understand which interfaces are being used for sending and receiving multicast packets:

Default Interface
The default interface for multicast transmission is typically the system's "default" network interface, which is usually determined by the routing metric of the OS. This is not a multicast-specific setting but rather the interface that the operating system uses for outgoing traffic when no specific route is defined for a given destination.

Viewing the Routing Table
To see the routing table, you can use the netstat command with the -nr options (or route -n on some Unix-like systems):

sh
Copy code
netstat -nr
Or

sh
Copy code
route -n
This shows you the kernel routing tables. For multicast specific routes, you might not see anything specific because multicast routing is usually managed differently than unicast routing.

Multicast Routing
Linux systems use IGMP (Internet Group Management Protocol) to manage multicast membership, and you can check which multicast groups your system is a part of using the following command:

sh
Copy code
netstat -ng
Publishing on Interfaces
When you create a multicast sender with the provided code, you are sending data out to the multicast group specified by MCAST_ADDR with a TTL (Time To Live) of 10. The TTL value in a multicast packet determines the scope or range that the packet will be forwarded by routers. With a TTL of 10, the packet is allowed to be forwarded up to 10 router hops away.

The socket options you've set in the code (e.g., IP_MULTICAST_TTL and IP_MULTICAST_LOOP) affect how the multicast packets are sent:

IP_MULTICAST_LOOP set to 1 means that the multicast packets you send will also be received by the local machine's multicast listeners if any are active. This is useful for loopback testing.
IP_MULTICAST_TTL defines how many "hops" the multicast packet can take in the network. This does not bind the sending to any specific interface; rather, it's a limit on how far the packet can travel.
If you want to ensure that multicast packets are sent over a specific interface, you need to use the IP_MULTICAST_IF socket option. This allows you to specify an outgoing interface for multicast packets. Without setting IP_MULTICAST_IF, the system's routing table will determine the default outgoing interface.

Here's an example of setting IP_MULTICAST_IF:

cpp
Copy code
struct in_addr localInterface;
localInterface.s_addr = inet_addr("your_specific_interface_ip");
if (setsockopt(serverFD, IPPROTO_IP, IP_MULTICAST_IF, &localInterface,
sizeof(localInterface)) < 0) {
perror("setsockopt - IP_MULTICAST_IF");
close(serverFD);
exit(EXIT_FAILURE);
}
You would replace "your_specific_interface_ip" with the IP address of the interface you wish to use for transmitting multicast packets. This is typically the IP address of the interface that is connected to the network where you want the multicast packets to be distributed.



To send multicast data you publish to a multicast address.
To kernel sees that message and if you've considered loopback passes it back to the system.
You can create an explicit route in your routing tables for packets to that mcast group which should allow you to forward packets to a specific interface.
you add membership for a multicast group which creates a route in the kernel table for that.
then you bind to an interface to listen for mcast packets.
I think the kernel doesn't forward the packets on a loopback to any physical interfaces so only virtual devices see these packets or the broadcast address.



MD Pcaps: https://databento.com/pcaps#samples


sudo ip tuntap add dev tap0 mode tap user `whoami`
sudo ip link set tap0 up
sudo ip addr add 192.168.100.1/24 dev tap0

on the guest vm:
sudo ip addr add 192.168.100.2/24 dev ens3

nc -lk 1234 on guest
echo abcd | nc 192.168.100.2 1234 on host to test network connectivity.


yum install bridge-utils

qemu-img resize /home/joshuacoutinho/Downloads/Fedora-Server-KVM-39-1.5.x86_64.qcow2 30G

./qemu-system-x86_64 -m 1024 -enable-kvm -drive if=virtio,file=test.qcow2,cache=none -cdrom ~/Downloads/Fedora-Workstation-Live-x86_64-38-1.6.iso \
-netdev tap,id=net0,ifname=tap0,script=no,downscript=no \
-device e1000e,netdev=net0,id=nic0

./qemu-system-x86_64 -m 2048 -enable-kvm -drive if=virtio,file=igb-patch.qcow2,cache=none  -cdrom ~/Downloads/Fedora-Workstation-Live-x86_64-38-1.6.iso -netdev tap,id=net0,ifname=tap0,script=no,downscript=no -device igb,netdev=net0,id=nic0

joshuacoutinho@fedora:~/CLionProjects/qemu-8.1.3/build$ 
./qemu-system-x86_64 -m 2048 -enable-kvm -hda /home/joshuacoutinho/Downloads/Fedora-Server-KVM-39-1.5.x86_64.qcow2 -netdev tap,id=net0,ifname=tap0,script=no,downscript=no -device igb,netdev=net0,id=nic0
https://linuxconfig.org/how-to-resize-a-qcow2-disk-image-on-linux


output/host/bin/qemu-system-x86_64 -m 2048 -kernel output/images/bzImage -drive file=output/images/rootfs.ext2,if=virtio,format=raw -append "rootwait root=/dev/vda" -netdev tap,id=net0,ifname=tap0,script=no,downscript=no -device igb,netdev=net0,id=nic0

joshuacoutinho@fedora:~$ #sudo qemu-system-x86_64 -kernel ../../linux/arch/x86/boot/bzImage -hda ../isos/Fedora-Server-KVM-39-1.5.x86_64.qcow2 -nographic -append "root=/dev/sysvg/root console=ttyS0" -initrd /boot/initramfs-6.7.0-rc3-00284-g815fb87b7530-dirty.img -m 2G



mkonjibhu123
user jc
edited sudoers and turned off the firewall from the management console here: 192.168.100.2:9090.


I tried getting multicast traffic looped back to the host forwarded to the tap device. The loopback works.
Adding a listener to ensure the multicast group is subscribed to and adding the following rules don't do anything to forward the traffic.
I suspect the tap0 is not getting the traffic or if it is qemu is not forwarding the traffic to the nic. either way something to investigate and experiment on.


A simple way to route specific packets that youre interested in:
mark the packets in pre routing
sudo iptables -t mangle -A PREROUTING -d 239.255.0.1 -p udp --dport 12345 -j MARK --set-mark 1

# Create a new routing table with a name, for example, "multicast"
echo "200 multicast" | sudo tee -a /etc/iproute2/rt_tables

# Add a routing rule that uses the "multicast" table for marked packets
sudo ip rule add fwmark 1 table multicast

# Set the default route for this table to go out via tap0
sudo ip route add default dev tap0 table multicast


{ - this doesn't work
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -A FORWARD -i wlp0s20f3 -o tap0 -d 239.255.0.1 -j ACCEPT
}

Details for packet info on a machine:
/proc/net/dev - live packet info
/proc/net/snmp
ip -s link
ss -s
iptables -L -v -n
nft list ruleset
tc -s qdisc show dev eth0
ethtool -S eth0

----------------------


------------------

debugging why packets are being dropped:
/proc/net/snmp is invaluable:

Tcp: RtoAlgorithm RtoMin RtoMax MaxConn ActiveOpens PassiveOpens AttemptFails EstabResets CurrEstab InSegs OutSegs RetransSegs InEr
rs OutRsts InCsumErrors
Tcp: 1 200 120000 -1 169558 83834 4467 9373 36 7750904 7437847 80969 969 42987 0
Udp: InDatagrams NoPorts InErrors OutDatagrams RcvbufErrors SndbufErrors InCsumErrors IgnoredMulti MemErrors
Udp: 7891464 10 61057700 120022858 61057700 3037386066 0 40171429 0


recvbuf and sndbug numbers as well as inerrors or ignoredmulti

I get recvbuferrors however the packets are being delievered in order.


----

[jc@fedora data]$ sudo tcpdump -i any -w multicast_dump10ms.pcap 'udp and port 12345 and dst 239.255.0.1'


https://blog.cloudflare.com/missing-manuals-io_uring-worker-pool/

netstat -su

/proc/net/udp
111675752 packet receive errors
174849548 packets sent
111675752 receive buffer errors

    111680564 packet receive errors
    174854709 packets sent
    111680564 receive buffer errors

ChatGPT
In the netstat -su command output for UDP statistics, the terms "packet receive errors" and "receive buffer errors" refer to specific issues encountered by the UDP protocol when processing incoming packets:

Packet Receive Errors:
This statistic represents the number of all errors that occurred during packet reception. This is a cumulative count of all issues that prevented a packet from being processed successfully, which can include:

Checksum errors, indicating that the packet data was corrupted in transit.
Packets that exceed the maximum transmission unit (MTU) and cannot be reassembled.
Packets that arrived at a closed port, with no listening process.
Packets that could not be processed for other reasons, like header issues.
Receive Buffer Errors:
These errors specifically count the number of packets that have been lost due to insufficient buffer space. When the UDP protocol receives a packet, it places it into a queue (socket receive buffer) to be read by the application. If this buffer is full, the new incoming packet cannot be queued and is consequently dropped. This buffer overflow situation is commonly encountered when an application doesn't read incoming data quickly enough, causing the buffer to fill up beyond its capacity. The "receive buffer errors" would then increment each time this happens.

Here's an example snippet from the output of netstat -su, where these values are typically listed:
ss -u -a : receive send queue size view usage.


Setting up network on host:
# Bring the interface down to make changes
sudo ip link set dev ens3 down

# Assign the IP address
sudo ip addr add 192.168.100.10/24 dev ens3

# Bring the interface up to enable networking
sudo ip link set dev ens3 up

# Add a default gateway if necessary
sudo ip route add default via 192.168.100.1

Need to bind to the tap device.

nc -l -p 1234
nc <host> <port>


echo 1 > /proc/sys/net/ipv4/ip_forward
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
add route to gateway router to reach the router



https://man7.org/linux/man-pages/man7/packet.7.html
use eth_p_ip for all ip packets.
packet_outgoing for packets looped back in.
you can't use the IP_ADD_MEMBERSHIP on a raw packet.
you need to use ioctls to add the multicast group 


ethtool -i ens3 for driver
sudo dnf groupinstall "Development Tools"

sudo yum install -y libncurses-devel
Kernel Configuration: Kernel modules are compiled based on the current kernel configuration. If certain features (like the igb driver) are not enabled in the kernel configuration, they won't be compiled. You can check the configuration using make menuconfig and ensure that the necessary drivers are set to be built as modules. For the igb driver, you would typically find it under Device Drivers -> Network device support -> Ethernet driver support -> Intel(R) PRO/1000 Gigabit Ethernet support.
make menuconfig

make modules SUBDIRS=drivers/net/ethernet/intel/igb
cp bzImage to the /boot/
backup existing module
sudo cp drivers/net/ethernet/intel/igb/igb.ko /lib/modules/$(uname -r)/kernel/drivers/net/ethernet/intel/igb/



Create a Network Bridge:

only works for wired connections

sudo brctl addbr br0
sudo brctl addif br0 eth0
sudo brctl addif br0 tap0
sudo ip addr add 192.168.1.2/24 dev br0
sudo ip link set dev br0 up

Edit /etc/sysctl.conf and add the following line:
net.ipv4.ip_forward = 1 
sudo iptables -t nat -A POSTROUTING -o wlp0s20f3 -j MASQUERADE
sudo iptables -A FORWARD -i wlp0s20f3 -o tap0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i tap0 -o wlp0s20f3 -j ACCEPT

nmcli con mod ens3 ipv4.dns "194.168.4.100"

on vm
sudo ip route add default via 192.168.100.1


Locate the bzImage File:

The bzImage file is typically located in arch/x86/boot/ within your kernel source directory. Based on your build output, the path would be ~/CLionProjects/linux/arch/x86/boot/bzImage.
Copy bzImage to /boot:

You need to copy this file to your /boot directory. It's common to rename bzImage to something more descriptive, often including the kernel version. For example, if your new kernel version is 5.10.1-custom, you might name the file vmlinuz-5.10.1-custom.
The command to copy and rename the file might look like this (replace [version] with your kernel version):
bash
Copy code
sudo cp ~/CLionProjects/linux/arch/x86/boot/bzImage /boot/vmlinuz-[version]
Create an Initramfs:

You also need to create an initial RAM filesystem (initramfs) for your new kernel. This can be done using the mkinitrd or dracut command:
bash
Copy code
sudo dracut /boot/initramfs-[version].img [version]
Replace [version] with the same version string you used for the vmlinuz file.
Update the Bootloader:

After copying the kernel and creating the initramfs, you need to update the bootloader so it knows about the new kernel. If you are using GRUB, this usually involves updating its configuration:
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
Make sure your new kernel entry is added to the GRUB menu.
Reboot and Select the New Kernel:

Once everything is in place, reboot your system.
During the boot process, you should see an option to boot into the new kernel in the GRUB menu. Select it to boot with your new custom kernel.


sudo rmmod igb
sudo modprobe igb
After loading the new module, you can check the kernel log with dmesg to see your "Hello!" message.

bash
Copy code
dmesg | grep Hello




Using buildroot:

sudo apt-get install make gcc g++ binutils patch bzip2 flex bison gettext pkg-config libncurses5-dev libssl-dev libelf-dev autoconf automake libtool gawk wget git python unzip rsync

with buildroot:
use make menuconfig - dont really need to change too many kernel settings apart from the custom version likely.

use local.mk to override the location of your .config and src dir
needed to install the following perl
sudo dnf install perl-English perl-ExtUtils-MakeMaker perl-Thread-Queue perl-FindBin perl-IPC-Cmd

then run make 