# Container daemons for OpenWrt

This feed provides various container-related tools and daemons for OpenWrt.
Due to complexity or other factors these are currently not suitable for upstream submission.

## Prerequisites:
Various kernel options (such as cgroups, fhandle) need to be enabled to run programs such as Docker - these
are not enabled in OpenWrt by default.

A sample diffconfig for x86-64 is in openwrt_configs/

## Docker on OpenWrt
This uses the pre-compiled static binaries provided by Docker, this avoids trying to build inside
the OpenWrt buildroot.

(A 'natively' built package is not impossible, it already exists in other musl-based distributions such as
Buildroot and Alpine, merely someone hasn't done it yet...).

Currently supported in this feed is x86_64 and aarch64 - these are in the ```docker-binary-x86-64```
and ```docker-binary-aarch64``` packages respectively.

### Storage media requirements
You must have some form of conventional storage to store the docker data dir ("/var/lib/docker"),
if your OpenWrt system is on ext4 this will work (or you have an external drive mounted).

Be aware that the Docker binaries and the data Docker creates are massive in OpenWrt terms - around
130MB for the Docker binaries and the data dir can easy balloon past 1GiB.

If you build your own OpenWrt images, you can adjust ```CONFIG_TARGET_ROOTFS_PARTSIZE``` to provide
enough space out of the box.

### Usage
An initscript and config file is provided.
One caveat is that the default Docker network setup (--iptables=true) is not working on
OpenWrt, but this is a not a bad thing - as the Docker setup would interfere with OpenWrt's firewall.

I suggest creating a bridge network for your containers (this takes the role of 'docker0' on
a "normal" host system):
```
cat /etc/config/network
...
config interface 'container'
        option type 'bridge'
        option proto 'static'
        option force_link '1'
        option bridge_empty '1'
        option ipaddr '10.21.0.1'
        option netmask '255.255.255.0'
```
Hints: ```bridge_empty``` is important - netifd will not create an 'empty' bridge (with no real interfaces attached) by default.

Similarly, ```force_link``` tells netifd to ignore the link status of any other interfaces in the bridge.

You can then add the 'container' network into a firewall zone to allow containers to access the
wider internet.

To configure the Docker daemon, see /etc/config/docker:
```
cat /etc/config/docker
config daemon
        option enable 1 # On a fresh install, this will be set to 0, set to 1 when ready
        option datadir '/opt/docker' # This must be on a 'real' block device (set up /etc/config/fstab first)
        option bridge 'br-container'
```

Note that the bridge option refers to the 'default' bridge and needs to be specified as the Linux network name.
If you set up a bridge in OpenWrt (as we do above), prefix the interface name with 'br-'.

When you start a container in Docker you will see the host OpenWrt as the containers router:
```
root@OpenWrt:~# ifconfig br-container
br-container Link encap:Ethernet  HWaddr DA:56:5C:AE:5C:31
          inet addr:10.21.0.1  Bcast:10.21.0.255  Mask:255.255.255.0
          inet6 addr: fe80::806a:48ff:fe09:734a/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1493 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2345 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:64022 (62.5 KiB)  TX bytes:8776413 (8.3 MiB)

root@OpenWrt:~# docker run -i -t alpine:latest
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:0A:15:00:03
          inet addr:10.21.0.3  Bcast:10.21.0.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:5 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:526 (526.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

### Running routers inside a container
The provided init script for Docker contains support for connecting containers
to OpenWrt bridges ```(br-X)```, this can be used to create containerized routers.

Normally, this would not be possible without using a specific network driver in Docker (e.g macvlan),
to avoid this, we do the required network setup outside the container.

This is very rudimentary and may be removed in favour of other solutions in the future.

```
# cat /etc/config/docker
config container 'openwrt'
        option image 'registry.gitlab.com/mcbridematt/openwrt-docker/arm64'
        option network 'lan:eth1 containerlan:eth0'
```

With the above configuration, ```br-lan``` will be connected to ```eth1``` in the container
and ```br-containerlan``` will be connected to eth0.

As this is done after the container process started, a flag file (/tmp.) is provided so
the container can wait for the setup to be completed:
```
# Put this in your container entrypoint script
if [ ! -z "${NETWORKWAIT}" ]; then
        while [ ! -f /tmp/.nsnetsetup ]; do
                sleep 1
        done
fi
```

### Known issues / Planned changes
* A kernel exception may occur when a container is stopped (on network namespace removal) - this doesn't seem to have any knock-on effects to the host system

* The init script doesn't check to see if the dockerd socket is responsive yet

* It is proposed to separate runc and the other binaries out from the docker package, in the case of runc a separate init and config is likely (runc would tie in better with procd)

### Running OpenWrt inside Docker?
See my [OpenWrt Container](https://gitlab.com/mcbridematt/openwrt-docker) repository - these have
the network wait function described above.

### See also:
* [OpenWrt container builds](https://gitlab.com/mcbridematt/openwrt-docker)
* [muvirt / KVM on OpenWrt](https://gitlab.com/traversetech/muvirt)
