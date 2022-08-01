# otusVLAN

## Задание 

в Office1 в тестовой подсети появляется сервера с доп интерфесами и адресами
в internal сети testLAN

testClient1 - 10.10.10.254
testClient2 - 10.10.10.254
testServer1- 10.10.10.1
testServer2- 10.10.10.1
равести вланами
testClient1 <-> testServer1
testClient2 <-> testServer2
между centralRouter и inetRouter
"пробросить" 2 линка (общая inernal сеть) и объединить их в бонд
проверить работу c отключением интерфейсов
Формат сдачи ДЗ - vagrant + ansible

## Решение

После развертывания стенда командой ```vagrant up``` получаю следующую схему сети

![image](https://user-images.githubusercontent.com/98832702/182242423-e98fe05f-608c-4614-a5b4-dd4986edbfbc.png)

Проверяю работу Vlan 101

```
root@testServer2:~# ping 10.10.10.254
PING 10.10.10.254 (10.10.10.254) 56(84) bytes of data.
64 bytes from 10.10.10.254: icmp_seq=1 ttl=64 time=0.507 ms
64 bytes from 10.10.10.254: icmp_seq=2 ttl=64 time=0.488 ms
64 bytes from 10.10.10.254: icmp_seq=3 ttl=64 time=0.392 ms

root@testClient2:~# tcpdump
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on vlan, link-type EN10MB (Ethernet), capture size 262144 bytes
20:35:06.767260 IP 10.10.10.1 > testClient2: ICMP echo request, id 5, seq 1, length 64
20:35:06.767310 IP testClient2 > 10.10.10.1: ICMP echo reply, id 5, seq 1, length 64
20:35:07.773635 IP 10.10.10.1 > testClient2: ICMP echo request, id 5, seq 2, length 64
20:35:07.773739 IP testClient2 > 10.10.10.1: ICMP echo reply, id 5, seq 2, length 64
20:35:08.797650 IP 10.10.10.1 > testClient2: ICMP echo request, id 5, seq 3, length 64
20:35:08.797700 IP testClient2 > 10.10.10.1: ICMP echo reply, id 5, seq 3, length 64

root@testClient1:~# tcpdump
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on vlan, link-type EN10MB (Ethernet), capture size 262144 bytes
^C
```

Аналогично vlan 100

```
root@testServer1:~# ping 10.10.10.254
PING 10.10.10.254 (10.10.10.254) 56(84) bytes of data.
64 bytes from 10.10.10.254: icmp_seq=1 ttl=64 time=2.21 ms
64 bytes from 10.10.10.254: icmp_seq=2 ttl=64 time=0.371 ms
64 bytes from 10.10.10.254: icmp_seq=3 ttl=64 time=0.371 ms
^C

root@testClient1:~# tcpdump
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on vlan, link-type EN10MB (Ethernet), capture size 262144 bytes
20:37:55.945024 ARP, Request who-has testClient1 tell 10.10.10.1, length 46
20:37:55.945104 ARP, Reply testClient1 is-at 08:00:27:1d:c2:0f (oui Unknown), length 28
20:37:55.945908 IP 10.10.10.1 > testClient1: ICMP echo request, id 1, seq 1, length 64
20:37:55.945943 IP testClient1 > 10.10.10.1: ICMP echo reply, id 1, seq 1, length 64
20:37:56.945409 IP 10.10.10.1 > testClient1: ICMP echo request, id 1, seq 2, length 64
20:37:56.945451 IP testClient1 > 10.10.10.1: ICMP echo reply, id 1, seq 2, length 64
20:37:57.960495 IP 10.10.10.1 > testClient1: ICMP echo request, id 1, seq 3, length 64
20:37:57.960536 IP testClient1 > 10.10.10.1: ICMP echo reply, id 1, seq 3, length 64
20:38:01.044378 ARP, Request who-has 10.10.10.1 tell testClient1, length 28
20:38:01.045047 ARP, Reply 10.10.10.1 is-at 08:00:27:e9:03:b5 (oui Unknown), length 46

root@testClient2:~# tcpdump
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on vlan, link-type EN10MB (Ethernet), capture size 262144 bytes
^C
0 packets captured
0 packets received by filter
0 packets dropped by kernel
```

Проверяю работу bond 

```
[root@centralRouter ~]# ping 192.168.255.1
PING 192.168.255.1 (192.168.255.1) 56(84) bytes of data.
64 bytes from 192.168.255.1: icmp_seq=1 ttl=64 time=0.453 ms
64 bytes from 192.168.255.1: icmp_seq=2 ttl=64 time=0.386 ms
[root@centralRouter ~]# ifdown eth2
[root@centralRouter ~]# ping 192.168.255.1
PING 192.168.255.1 (192.168.255.1) 56(84) bytes of data.
64 bytes from 192.168.255.1: icmp_seq=1 ttl=64 time=0.428 ms
64 bytes from 192.168.255.1: icmp_seq=2 ttl=64 time=0.337 ms
^C
[root@centralRouter ~]# ifup eth2

[root@inetRouter ~]# ifdown eth2
[root@inetRouter ~]# ping 192.168.255.2
PING 192.168.255.2 (192.168.255.2) 56(84) bytes of data.
64 bytes from 192.168.255.2: icmp_seq=1 ttl=64 time=0.744 ms
64 bytes from 192.168.255.2: icmp_seq=2 ttl=64 time=0.411 ms
64 bytes from 192.168.255.2: icmp_seq=3 ttl=64 time=0.322 ms
64 bytes from 192.168.255.2: icmp_seq=4 ttl=64 time=0.401 ms
^C
```
