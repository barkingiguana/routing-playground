# Routing

A playground for BGP and other routing fun.

```
$ bundle
$ # Set up the routers:
$ bundle exec rake compound:routing:000-setup
$ # Set up the hosts:
$ bundle exec rake compound:routing:001-hosts
```

You now have a network of 4 AS (using private AS allocation)

3 AS have 1 router and 1 host.
1 AS announces that it is carrying the internet via router-d.

The routers talk BGP to each other, to share routes.

```
             [ host-a ]   .-----[ router-d ]-----{{ internet }}
                 |       /
                 |      /
            [ router-a ]
           /            \
          /              \
         /                \
  [ router-b ]----------[ router-c ]
      /                       \
     /                         \
[ host-b ]                  [ host-c ]
```

If you'd like to play with the hosts:

```
$ ssh -l vagrant 10.8.42.12 # hosts are .10 to .12, routers are .13 to .16.
vagrant@host-c:~$ route -vn
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.8.99.1       0.0.0.0         UG    0      0        0 enp0s9
10.8.42.0       0.0.0.0         255.255.255.0   U     0      0        0 enp0s8
10.8.99.0       0.0.0.0         255.255.255.0   U     0      0        0 enp0s9
vagrant@host-a:~$ tracepath -n 10.8.98.100
 1?: [LOCALHOST]                                         pmtu 1500
 1:  10.8.99.1                                             0.282ms
 1:  10.8.99.1                                             0.249ms
 2:  192.168.3.1                                           0.466ms
 3:  10.8.98.100                                           3.551ms reached
     Resume: pmtu 1500 hops 3 back 3
```

# Problems...

TODO: the hosts and router-{a,b,c} can't access the internet. Packets leave the network via router-d, but the replies never come back. I _think_ that's because while we can route to the internet, the internet can't route back to us because we're using a private IP address range. I think I need to use NAT on a loopback in router-d to mangle the source IP - I expected the VirtualBox gateway to do that, but maybe it only does it for 10.0.2.15 ie the IP that VirtualBox expects the VM to have in the VM<->gateway network. Need to learn more about how the VirtualBox NAT gateway works.

```
vagrant@host-c:~$ tracepath -n 8.8.8.8
 1?: [LOCALHOST]                                         pmtu 1500
 1:  10.8.99.1                                             0.322ms
 1:  10.8.99.1                                             0.231ms
 2:  192.168.2.1                                           0.395ms
 3:  192.168.4.2                                           1.430ms
 4:  no reply
 5:  no reply
```

Observe ICMP packets transiting the link to the internet on router-d, while pinging from host-c:
```
vagrant@router-d:~$ sudo tcpdump -AxXX -vv -i enp0s3 -t icmp -n
tcpdump: listening on enp0s3, link-type EN10MB (Ethernet), capture size 262144 bytes
IP (tos 0x0, ttl 61, id 35, offset 0, flags [DF], proto ICMP (1), length 84)
    10.8.99.100 > 8.8.8.8: ICMP echo request, id 2362, seq 1, length 64
        0x0000:  5254 0012 3502 0800 270c 4486 0800 4500  RT..5...'.D...E.
        0x0010:  0054 0023 4000 3d01 c00a 0a08 6364 0808  .T.#@.=.....cd..
        0x0020:  0808 0800 2c3b 093a 0001 e184 6558 0000  ....,;.:....eX..
        0x0030:  0000 b2d9 0a00 0000 0000 1011 1213 1415  ................
        0x0040:  1617 1819 1a1b 1c1d 1e1f 2021 2223 2425  ...........!"#$%
        0x0050:  2627 2829 2a2b 2c2d 2e2f 3031 3233 3435  &'()*+,-./012345
        0x0060:  3637                                     67
```
