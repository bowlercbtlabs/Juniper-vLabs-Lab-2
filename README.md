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
4) Try to ping from host to host (vMX to vMX)
5) 
