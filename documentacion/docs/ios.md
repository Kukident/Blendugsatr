R1# configure terminal
R1(config)# interface fastethernet0/0
R1(config-if)# ip nat inside

Next step is to set the serial interface S0/0 as the outside interface:
R1(config-if)# interface serial0/0
R1(config-if)# ip nat outside
R1(config-if)# exit

We now need to create an Access Control List (ACL) that will include local (private) hosts or network(s). This ACL will later on be applied to the NAT service command, effectively controlling the hosts that will be able to access the Internet. You can use standard or extended access lists depending on your requirements:

R1(config)# access-list 100 remark == [Control NAT Service]==
R1(config)# access-list 100 permit ip 192.168.0.0 0.0.0.255 any
The above command instructs the router to allow the 192.168.0.0/24 network to reach any destination. Note that Cisco router standard and extended ACLs always use wildcards (0.0.0.255).

All that's left now is to enable NAT overload and bind it to the outside interface previously selected:

R1(config)# ip nat inside source list 100 interface serial 0/0 overload

R1# show ip nat translations

http://www.firewall.cx/cisco-technical-knowledgebase/cisco-routers/260-cisco-router-nat-overload.html