## TP 5 - Une "vraie" topologie ?
### I. Toplogie 1 - intro VLAN
####  Setup clients
##### les différents pings:
##### les guests:
```
guest1> ping 10.5.20.12
84 bytes from 10.5.20.12 icmp_seq=1 ttl=64 time=0.293 ms
84 bytes from 10.5.20.12 icmp_seq=2

guest2> ping 10.5.20.11
84 bytes from 10.5.20.11 icmp_seq=1 ttl=64 time=0.277 ms
84 bytes from 10.5.20.11 icmp_seq=2 ttl=64 time=0.474 ms
```
##### les admins:
```
admin1> ping 10.5.10.12
84 bytes from 10.5.10.12 icmp_seq=1 ttl=64 time=0.338 ms
84 bytes from 10.5.10.12 icmp_seq=2 ttl=64 time=0.495 ms

admin2> ping 10.5.10.11
84 bytes from 10.5.10.11 icmp_seq=1 ttl=64 time=0.467 ms
84 bytes from 10.5.10.11 icmp_seq=2 ttl=64 time=0.508 ms

```
#### Setup VLANs

##### le switch1:
```
10   admins                           active    Et0/0
20   guests                           active    Et1/0

```
##### l'interface trunk
```
IOU1#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Et2/0       on               802.1q         trunking      1

```
##### le switch2:
```

10   admins                           active    Et0/0
20   guests                           active    Et1/0

```
##### l'interface trunk
```
IOU2#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Et2/0       on               802.1q         trunking      1
```
##### ping de guest
```
guest1> ping 10.5.20.12
84 bytes from 10.5.20.12 icmp_seq=1 ttl=64 time=0.308 ms
84 bytes from 10.5.20.12 icmp_seq=2 ttl=64 time=0.398 ms
```
##### ping de admin 
```
admin1> ping 10.5.10.12
84 bytes from 10.5.20.12 icmp_seq=1 ttl=64 time=0.326 ms
84 bytes from 10.5.20.12 icmp_seq=2 ttl=64 time=0.374 ms

```

##### ping de guest transformé en ip admin:
```
ip 10.5.10.13

guest1> ping 10.5.10.12
host (10.5.10.12) not reachable

```
### II. Topologie 2 - VLAN, sous-interface, NAT
##### ping admin:
```
admin1> ping 10.5.10.12
84 bytes from 10.5.10.12 icmp_seq=1 ttl=64 time=0.348 ms
84 bytes from 10.5.10.12 icmp_seq=2 ttl=64 time=0.535 ms

admin1> ping 10.5.10.13
84 bytes from 10.5.10.13 icmp_seq=1 ttl=64 time=0.442 ms
84 bytes from 10.5.10.13 icmp_seq=2 ttl=64 time=0.659 ms

admin2> ping 10.5.10.11
84 bytes from 10.5.10.11 icmp_seq=1 ttl=64 time=0.382 ms
84 bytes from 10.5.10.11 icmp_seq=2 ttl=64 time=1.610 ms


admin2> ping 10.5.10.13
84 bytes from 10.5.10.13 icmp_seq=1 ttl=64 time=0.400 ms
84 bytes from 10.5.10.13 icmp_seq=2 ttl=64 time=0.493 ms

```

##### ping guest:
```
guest1> ping 10.5.20.12
84 bytes from 10.5.20.12 icmp_seq=1 ttl=64 time=0.630 ms
84 bytes from 10.5.20.12 icmp_seq=2 ttl=64 time=0.472 ms

guest1> ping 10.5.20.13
84 bytes from 10.5.20.13 icmp_seq=1 ttl=64 time=0.411 ms
84 bytes from 10.5.20.13 icmp_seq=2 ttl=64 time=0.532 ms


guest2> ping 10.5.20.11
84 bytes from 10.5.20.11 icmp_seq=1 ttl=64 time=0.519 ms
84 bytes from 10.5.20.11 icmp_seq=2 ttl=64 time=0.393 ms

guest2> ping 10.5.20.13
84 bytes from 10.5.20.13 icmp_seq=1 ttl=64 time=0.364 ms
84 bytes from 10.5.20.13 icmp_seq=2 ttl=64 time=0.562 ms

```

##### on configure les vlans afin de récupérer ce résultat sur les trois switchs:
```
10   admins                           active    Et0/0
20   guests                           active    Et0/1
```
##### on configure les trunk pour le switch 1 et 3:
```
Port        Vlans allowed on trunk
Et0/2       1-4094

Port        Vlans allowed and active in management domain
Et0/2       1,10,20

```
##### Puis pour le switch 2:
```
Port        Vlans allowed on trunk
Et0/2      1-4094
Et0/3     1-4094

Port        Vlans allowed and active in management domain
Et0/2       1,10,20
Et0/3       1,10,20
```
##### Je vais changer l'ip d'un guest:
```
guest1> ip 10.5.10.15/24
Checking for duplicate address...
guest1 : 10.5.10.15 255.255.255.0

guest1> ping 10.5.10.12
host (10.5.10.12) not reachable
```
```
R1(config)#interface fastEthernet 0/0.10
R1(config-subif)#encapsulation dot1Q 10
R1(config-subif)#ip address 10.5.10.254 255.255.255.0
R1(config-subif)#exit
R1(config)#interface fastEthernet 0/0
R1(config)#no shut
R1(config-subif)#exit
R1(config)#interface fastEthernet 0/0.20
R1(config-subif)#encapsulation dot1Q 20
R1(config-subif)#ip address 10.5.20.254 255.255.255.0
```
##### On vérifie que tout est bon:
```
R1#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            unassigned      YES unset  up                    up  
FastEthernet0/0.10         10.5.10.254     YES manual up                    up  
FastEthernet0/0.20         10.5.20.254     YES manual up                    up  
FastEthernet1/0            unassigned      YES unset  administratively down down
```
##### On modifie le trunk du switch2:
```
IOU2(config)#interface Ethernet 1/0
IOU2(config-if)#switchport trunk encapsulation dot1q
IOU2(config-if)#switchport mode trunk
IOU2(config-if)#switchport trunk allowed vlan 10,20
```
##### une fois la modification effectuer on vérifie:
```
Port        Vlans allowed on trunk
Et0/2      1-4094
Et0/3       1-4094
Et1/0       10,20

Port        Vlans allowed and active in management domain
Et0/2      1,10,20
Et0/3       1,10,20
Et1/0       10,20
```

##### les pings vers le routeur:
```
admin1> ping 10.5.10.254
84 bytes from 10.5.10.254 icmp_seq=1 ttl=64 time=9.672 ms
84 bytes from 10.5.10.254 icmp_seq=2 ttl=64 time=6.387 ms

admin2> ping 10.5.10.254
84 bytes from 10.5.10.254 icmp_seq=1 ttl=64 time=9.620 ms
84 bytes from 10.5.10.254 icmp_seq=2 ttl=64 time=3.189 ms

admin3> ping 10.5.10.254
84 bytes from 10.5.10.254 icmp_seq=1 ttl=64 time=9.341 ms
84 bytes from 10.5.10.254 icmp_seq=2 ttl=64 time=7.284 ms

guest1> ping 10.5.10.254
84 bytes from 10.5.10.254 icmp_seq=1 ttl=64 time=9.847 ms
84 bytes from 10.5.10.254 icmp_seq=2 ttl=64 time=5.682 ms

guest2> ping 10.5.10.254
84 bytes from 10.5.10.254 icmp_seq=1 ttl=64 time=9.254 ms
84 bytes from 10.5.10.254 icmp_seq=2 ttl=64 time=3.351 ms

guest3> ping 10.5.10.254
84 bytes from 10.5.10.254 icmp_seq=1 ttl=64 time=9.168 ms
84 bytes from 10.5.10.254 icmp_seq=2 ttl=64 time=5.294 ms

```
### 5. NAT
##### on configure la nat 
```
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#interface fastEthernet 0/0.10
R1(config-subif)#ip nat inside
R1(config-subif)#exit
R1(config)#interface fastEthernet 0/0.20
R1(config-subif)#ip nat inside
R1(config-subif)#exit
R1(config)#interface fastEthernet 1/0
R1(config-if)#ip nat outside
R1(config-if)#exit
R1(config)#access-list 1 permit any
R1(config)#ip nat inside source list 1 interface fastE
R1(config)#ip nat inside source list 1 interface fastEthernet 1/0 overload
R1(config)#exit
```
##### une fois la configuration effectuer on vérifie:

```
R1#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            unassigned      YES unset  up                    up  
FastEthernet0/0.10         10.5.10.254     YES manual up                    up  
FastEthernet0/0.20         10.5.20.254     YES manual up                    up  
FastEthernet1/0            unassigned      YES unset  administratively down down
NVI0                       unassigned      NO  unset  up                    up  

```
```
guest1> ping 8.8.8.8
*10.5.20.254 icmp_seq=1 ttl=255 time=9.416 ms (ICMP type:3, code:1, Destination host unreachable)
*10.5.20.254 icmp_seq=2 ttl=255 time=7.912 ms (ICMP type:3, code:1, Destination host unreachable)
*10.5.20.254 icmp_seq=3 ttl=255 time=11.052 ms (ICMP type:3, code:1, Destination host unreachable)
*10.5.20.254 icmp_seq=4 ttl=255 time=10.345 ms (ICMP type:3, code:1, Destination host unreachable)
*10.5.20.254 icmp_seq=5 ttl=255 time=8.206 ms (ICMP type:3, code:1, Destination host unreachable)
```

### 3. Serveur DHCP

![](https://i.imgur.com/sNCuJPj.png)

ensuite on start le dhcp puis on test :
```
guest1> ip dhcp
DDORA IP 10.5.20.100/24 GW 10.5.20.254

```
### 4. Serveur Web
sudo firewall-cmd --add-port=80/tcp --permanent

![](https://i.imgur.com/6KXpeZ1.png)
 
### 5. Serveur DNS
```
sudo firewall-cmd --add-port=68/tcp --permanent
```
on modifie les fichiers tp5.b1.db et 20.5.10. puis on:
```
service named start
```
puis on test:
```
guest1> ping client2.tp5.b1.
client2.tp5.b1 resolved to 10.5.20.12
84 bytes from 10.5.20.12 icmp_seq=1 ttl=64 time=0.314 ms
84 bytes from 10.5.20.12 icmp_seq=2 ttl=64 time=0.379 ms
```

On modifie une nouvelle fois les fichier en rajoutant toute les ip a tp5.b1.db.
```
guest2> ping web.tp5.b1.
web.tp5.b1 resolved to 10.5.30.12
84 bytes from 10.5.30.12 icmp_seq=1 ttl=63 time=21.284 ms
84 bytes from 10.5.30.12 icmp_seq=2 ttl=63 time=13.781 ms

```
```
guest1> ping dhcp.tp5.b1.
dhcp.tp5.b1 resolved to 10.5.20.253
84 bytes from 10.5.10.12 icmp_seq=1 ttl=64 time=1.187 ms
84 bytes from 10.5.10.12 icmp_seq=2 ttl=64 time=0.760 ms
```
on modifie les lignes : 

option domain-name-servers 10.5.30.11;
option routers 10.5.20.254;
