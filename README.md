# Juniper-vLabs-Lab-2
Juniper vLabs Lab 2 (vSRX Firewall Security Zones)

- Provide the initial configuration files for the devices on github
    - https://github.com/bowlercbtlabs/Juniper-vLabs-Lab-2/blob/main/Initial%20Device%20Configurations
- Explain the basics about security zones
- configure a trust and untrust security zone
- configure a security policy to allow communication between the 2 zones


Lab Steps:

1) Login to vLabs and Reserve/Launch the Lab (should take a few minutes to start up)
    - https://jlabs.juniper.net/vlabs/portal/zones-policies/
2) When setup is done, login to the vSRX and do the following:

jcluser@vSRX1# delete security policies from-zone trust to-zone trust policy default-permit 

[edit]
jcluser@vSRX1# delete security policies from-zone trust to-zone untrust policy default-permit 

[edit]
jcluser@vSRX1# delete security zones security-zone trust 

3) At this point we should only have the interfaces configured with IP addresses and no security policy
4) Try to ping from host1 to host2 (vMX to vMX), both hosts are inside the trust zone
jcluser@Host1> ping 10.100.12.2 rapid count 10           
PING 10.100.12.2 (10.100.12.2): 56 data bytes
..........
--- 10.100.12.2 ping statistics ---
10 packets transmitted, 0 packets received, 100% packet loss

5) You can see that the ping fails, this is because of the inherent nature of the firewall, if an interface hasn't been configured in a security zone, then packets will not be allowed through the interface
6) Check the vSRX for traffic on the interface

jcluser@vSRX1> show interfaces descriptions 

jcluser@vSRX1> show interfaces terse 
Interface               Admin Link Proto    Local                 Remote
ge-0/0/0                up    up
ge-0/0/0.0              up    up   inet     10.100.11.1/24  
gr-0/0/0                up    up
ip-0/0/0                up    up
lsq-0/0/0               up    up
lt-0/0/0                up    up
mt-0/0/0                up    up
sp-0/0/0                up    up
sp-0/0/0.0              up    up   inet    
                                   inet6   
sp-0/0/0.16383          up    up   inet    
ge-0/0/1                up    up
ge-0/0/1.0              up    up   inet     10.100.12.1/24  
ge-0/0/2                up    up
ge-0/0/2.0              up    up   inet     10.100.13.1/24  

7) Lets set a description on the interfaces:

jcluser@vSRX1# set interfaces ge-0/0/0 description host1 

[edit]
jcluser@vSRX1# set interfaces ge-0/0/1 description host2    

[edit]
jcluser@vSRX1# set interfaces ge-0/0/2 description host3    

8) Monitor the ge-0/0/0 interface and start ping

jcluser@Host1> ping 10.100.12.2 rapid count 10        
PING 10.100.12.2 (10.100.12.2): 56 data bytes
..........
--- 10.100.12.2 ping statistics ---
10 packets transmitted, 0 packets received, 100% packet loss


jcluser@vSRX1> monitor interface ge-0/0/0 
Interface: ge-0/0/0, Enabled, Link is Up
Encapsulation: Ethernet, Speed: 10000mbps
Traffic statistics:Current delta
Input bytes:840 (336 bps) [84]
Output bytes:0 (0 bps)[0]
Input packets:10 (0 pps)[1]
Output packets:0 (0 pps)[0]


9)  As we can see, traffic is making it into the interface, but we see no outbound traffic.

10)  

