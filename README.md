# Routing

A playground for BGP and other routing fun.

```
$ bundle
$ bundle exec rake compound:routing:000-setup
```

You now have a network of 3 AS (private allocation), each with 1 router and
1 host:

```
             [ host-a ]
                 |
                 |
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
$ cd test/compound/routing
$ bundle exec vagrant ssh router-a # or router-b or router-c
vagrant@127.0.0.1's password: vagrant
vagrant@router-a:~$ ssh host-a
vagrant@host-a:~$ route -vn
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.8.97.1       0.0.0.0         UG    0      0        0 enp0s9
10.8.42.0       0.0.0.0         255.255.255.0   U     0      0        0 enp0s8
10.8.97.0       0.0.0.0         255.255.255.0   U     0      0        0 enp0s9
vagrant@host-a:~$ tracepath -n 10.8.99.100
 1?: [LOCALHOST]                                         pmtu 1500
 1:  10.8.97.1                                             0.229ms
 1:  10.8.97.1                                             0.158ms
 2:  192.168.2.2                                           0.424ms
 3:  10.8.99.100                                           1.668ms reached
     Resume: pmtu 1500 hops 3 back 3
```
