# Juniper-vLabs-Lab-2
Juniper vLabs Lab 2 (vSRX Firewall Security Zones - Trust Zone)

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


9)  As we can see, traffic is making it into the interface, but we see no outbound traffic returning to the source.

10)  We can run the packet-drop records command on the vSRX to see if the firewall is dropping the packets

jcluser@vSRX1> show security packet-drop records 

19:49:13.052137:LSYS-ID-00 10.100.11.2/50206-->10.100.12.2/9;icmp,ipid-27717,ge-0/0/0.0,Dropped by FLOW:First path Invalid zone
19:49:12.537753:LSYS-ID-00 10.100.11.2/50206-->10.100.12.2/8;icmp,ipid-24133,ge-0/0/0.0,Dropped by FLOW:First path Invalid zone
19:49:12.027785:LSYS-ID-00 10.100.11.2/50206-->10.100.12.2/7;icmp,ipid-20293,ge-0/0/0.0,Dropped by FLOW:First path Invalid zone
19:49:11.524291:LSYS-ID-00 10.100.11.2/50206-->10.100.12.2/6;icmp,ipid-15941,ge-0/0/0.0,Dropped by FLOW:First path Invalid zone
19:49:11.035512:LSYS-ID-00 10.100.11.2/50206-->10.100.12.2/5;icmp,ipid-12101,ge-0/0/0.0,Dropped by FLOW:First path Invalid zone
19:49:10.517509:LSYS-ID-00 10.100.11.2/50206-->10.100.12.2/4;icmp,ipid-8517,ge-0/0/0.0,Dropped by FLOW:First path Invalid zone
19:49:10.008931:LSYS-ID-00 10.100.11.2/50206-->10.100.12.2/3;icmp,ipid-4165,ge-0/0/0.0,Dropped by FLOW:First path Invalid zone
19:49:09.507837:LSYS-ID-00 10.100.11.2/50206-->10.100.12.2/2;icmp,ipid-65092,ge-0/0/0.0,Dropped by FLOW:First path Invalid zone
19:49:09.006869:LSYS-ID-00 10.100.11.2/50206-->10.100.12.2/1;icmp,ipid-60996,ge-0/0/0.0,Dropped by FLOW:First path Invalid zone

11)  As we can see, we are seeing traffic from host1 destined for host2 being blocked due to "Invalid Zone"

12)  Lets add the 'Trust' security zone and put both ge-0/0/0 and ge-0/0/1 inside that zone and allow ping/icmp traffic:

set security zones security-zone trust host-inbound-traffic system-services ping
set security zones security-zone trust interfaces ge-0/0/0.0
set security zones security-zone trust interfaces ge-0/0/1.0

13)  Now lets try the ping again from host1 to host2 and see what happens:

jcluser@Host1> ping 10.100.12.2 rapid count 10    
PING 10.100.12.2 (10.100.12.2): 56 data bytes
..........
--- 10.100.12.2 ping statistics ---
10 packets transmitted, 0 packets received, 100% packet loss

jcluser@vSRX1> clear security packet-drop records                 
60 packet-drop records cleared

jcluser@vSRX1> 

jcluser@vSRX1> 

jcluser@vSRX1> show security packet-drop records | refresh        
---(refreshed at 2024-04-03 20:32:52 UTC)---
Total packet-drop records: 0
---(refreshed at 2024-04-03 20:32:57 UTC)---
Total packet-drop records: 0
---(refreshed at 2024-04-03 20:33:02 UTC)---
20:33:02.535083:LSYS-ID-00 10.100.11.2/40991-->10.100.12.2/8;icmp,ipid-28479,ge-0/0/0.0,Dropped by POLICY:Denied by Policy  default-policy-logical-system-00
20:33:02.027942:LSYS-ID-00 10.100.11.2/40991-->10.100.12.2/7;icmp,ipid-24127,ge-0/0/0.0,Dropped by POLICY:Denied by Policy  default-policy-logical-system-00
20:33:01.524527:LSYS-ID-00 10.100.11.2/40991-->10.100.12.2/6;icmp,ipid-20287,ge-0/0/0.0,Dropped by POLICY:Denied by Policy  default-policy-logical-system-00
20:33:01.015159:LSYS-ID-00 10.100.11.2/40991-->10.100.12.2/5;icmp,ipid-15167,ge-0/0/0.0,Dropped by POLICY:Denied by Policy  default-policy-logical-system-00
20:33:00.512633:LSYS-ID-00 10.100.11.2/40991-->10.100.12.2/4;icmp,ipid-11583,ge-0/0/0.0,Dropped by POLICY:Denied by Policy  default-policy-logical-system-00
20:33:00.005229:LSYS-ID-00 10.100.11.2/40991-->10.100.12.2/3;icmp,ipid-7231,ge-0/0/0.0,Dropped by POLICY:Denied by Policy  default-policy-logical-system-00
20:32:59.495118:LSYS-ID-00 10.100.11.2/40991-->10.100.12.2/2;icmp,ipid-3647,ge-0/0/0.0,Dropped by POLICY:Denied by Policy  default-policy-logical-system-00

14)  As we can now see from the packet-drop records, the traffic is now hitting a Policy Deny, lets create a policy to allow the icmp traffic inside the trusted security zone:

set security policies from-zone trust to-zone trust policy ALLOW_ICMP match source-address any 
set security policies from-zone trust to-zone trust policy ALLOW_ICMP match destination-address any 
set security policies from-zone trust to-zone trust policy ALLOW_ICMP match application  
set security policies from-zone trust to-zone trust policy ALLOW_ICMP then permit


jcluser@vSRX1# show | compare 
[edit security policies]
+    from-zone trust to-zone trust {
+        policy ALLOW_ICMP {
+            match {
+                source-address any;
+                destination-address any;
+                application junos-icmp-all;
+            }
+            then {
+                permit;
+            }
+        }
+    }

  
15)  Lets try to ping again and see what happens

jcluser@Host1> ping 10.100.12.2 rapid count 10    
PING 10.100.12.2 (10.100.12.2): 56 data bytes
!!!!!!!!!!
--- 10.100.12.2 ping statistics ---
10 packets transmitted, 10 packets received, 0% packet loss
round-trip min/avg/max/stddev = 1.488/1.834/3.590/0.591 ms       

Number of policy: 1

16)  We can now see the ping was successful. We can further check the firewall policy for matches with the "hit-count" command

jcluser@vSRX1> show security policies hit-count    
Logical system: root-logical-system
 Index   From zone        To zone           Name           Policy count
 1       trust            trust             ALLOW_ICMP     10    

17)  Lets see if other types of traffic from host1 is able to make it to host2, we'll try to ssh to host2

jcluser@Host1> ssh 10.100.12.2 
ssh: connect to host 10.100.12.2 port 22: Operation timed out

jcluser@vSRX1> show security packet-drop records | refresh        
---(refreshed at 2024-04-03 20:48:33 UTC)---
20:48:28.940509:LSYS-ID-00 10.100.11.2/51486-->10.100.12.2/22;tcp,ipid-22935,ge-0/0/0.0,Dropped by POLICY:Denied by Policy  default-policy-logical-system-00
20:48:25.740424:LSYS-ID-00 10.100.11.2/51486-->10.100.12.2/22;tcp,ipid-2199,ge-0/0/0.0,Dropped by POLICY:Denied by Policy  default-policy-logical-system-00
20:48:22.539390:LSYS-ID-00 10.100.11.2/51486-->10.100.12.2/22;tcp,ipid-47510,ge-0/0/0.0,Dropped by POLICY:Denied by Policy  default-policy-logical-system-00
20:48:19.339245:LSYS-ID-00 10.100.11.2/51486-->10.100.12.2/22;tcp,ipid-25494,ge-0/0/0.0,Dropped by POLICY:Denied by Policy  default-policy-logical-system-00
20:48:16.140058:LSYS-ID-00 10.100.11.2/51486-->10.100.12.2/22;tcp,ipid-5014,ge-0/0/0.0,Dropped by POLICY:Denied by Policy  default-policy-logical-system-00
20:48:13.132873:LSYS-ID-00 10.100.11.2/51486-->10.100.12.2/22;tcp,ipid-50325,ge-0/0/0.0,Dropped by POLICY:Denied by Policy  default-policy-logical-system-00

18)  We see the ssh traffic is no good and being dropped by the firewall, lets add another policy for the SSH traffic

set security policies from-zone trust to-zone trust policy ALLOW_SSH match source-address any
set security policies from-zone trust to-zone trust policy ALLOW_SSH match destination-address any
set security policies from-zone trust to-zone trust policy ALLOW_SSH match application junos-ssh
set security policies from-zone trust to-zone trust policy ALLOW_SSH then permit

19)  Try the ssh from host1 to host2 again:

jcluser@Host1> ssh 10.100.12.2    
The authenticity of host '10.100.12.2 (10.100.12.2)' can't be established.
ECDSA key fingerprint is SHA256:aGF721Rr++omV2vC4OFKqC9VYm+HzbqWXbH/WtFlmJ0.
Are you sure you want to continue connecting (yes/no)?

jcluser@vSRX1> show security policies hit-count                                                    
Logical system: root-logical-system
 Index   From zone        To zone           Name           Policy count
 1       trust            trust             ALLOW_ICMP     20           
 2       trust            trust             ALLOW_SSH      1            

 jcluser@vSRX1> show security flow session | refresh                                                
---(refreshed at 2024-04-03 20:53:20 UTC)---
Session ID: 149, Policy name: ALLOW_SSH/5, State: Stand-alone, Timeout: 1786, Valid
  In: 10.100.11.2/58479 --> 10.100.12.2/22;tcp, Conn Tag: 0x0, If: ge-0/0/0.0, Pkts: 6, Bytes: 1785, 
  Out: 10.100.12.2/22 --> 10.100.11.2/58479;tcp, Conn Tag: 0x0, If: ge-0/0/1.0, Pkts: 4, Bytes: 1581, 

20)  It appears the connection is working! go ahead and create a user account on host2 and try to login:

jcluser@Host2# set system login user admin class read-only authentication plain-text-password

jcluser@Host1> ssh admin@10.100.12.2 
Password:
--- JUNOS 18.3R1.9 Kernel 64-bit  JNPR-11.0-20180816.8630ec5_buil
admin@Host2> 

jcluser@vSRX1> show security flow session | refresh    
---(refreshed at 2024-04-03 20:56:41 UTC)---
Session ID: 150, Policy name: ALLOW_SSH/5, State: Stand-alone, Timeout: 1792, Valid
  In: 10.100.11.2/53935 --> 10.100.12.2/22;tcp, Conn Tag: 0x0, If: ge-0/0/0.0, Pkts: 20, Bytes: 3417, 
  Out: 10.100.12.2/22 --> 10.100.11.2/53935;tcp, Conn Tag: 0x0, If: ge-0/0/1.0, Pkts: 18, Bytes: 3497, 
Total sessions: 1

21)  It appears to be working! you have successfully created an ssh session from host1 to host2 using the trust security zone

22)  In the next lab we will focus on creating a more detailed security policy between the untrust and trust zones (host3 and host1)

