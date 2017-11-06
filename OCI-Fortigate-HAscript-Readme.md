[Synopsis]

OCI Fortigate HA script is an automation for Fortigate Active/Passive design. 
The script will monitor the active Fortigate and will fail to Passive Fortigate when trigger condition is met.
During the failover, Public IP (Floating) is moved from Active Fortigate to Passive Fortigate. Also, 
routes in IPnetworks (trust or inside) adminDistance is changed such that they follow the Active Fortigate. 
This assures that traffic both directions inbound and outbound flows through the same Firewall.


[Pre-requisites]

-The script should be run on worker node in OCI environment. Worker node will linux instance (eg. ubuntu) and python2.7 installed. Also, python libraries listed should be installed.
-From the worker node, OCI environment needs to be reachable and also Fortigate Eth0 interfaces need to be reachable. 
-Two Fortigates deployed with interfaces in Shared Network (Eth0) and IP Networks (Eth1) Interfaces.
-Fortigates will have Floating Public IP between the instances. So while deploying Fortigates "Public IP" select as none in OCI.
-Create a PublicIP in OCI Shared Network IPReservation. Active Fortigate Eth0 should be assigned the Floating Public IP.
-Create two VnicSets for Fortigate1(Eth1) and Fortigate2(Eth1). 
-Create two Routes  for IP Network Prefix (Eg. 0.0.0.0/0) in IPNetworks with one route next-hop to Fortigate1Vnicset and other route next-hop set to Fortigate2Vnicset.
Each route will have separate adminDistance values. Lower AdminDistance Route is chosen.

-Active Fortigate at any given time is associated with FloatingPublic IP and DefaultRoute in IPNetworks with next-hop as ActiveFortigate Vnicset assigned with lower adminDistance.
-Passive Fortigate at any given time is not associated FloatingPublic IP and DefaultRoute in IPNetworks with next-hop as PassiveFortigate Vnicset assigned with higher adminDistance.

[Example]

		    Floating Public IP             <<--- IPReservation in shared Network
			        | 
	  ____________________________
     |                            |    
     |Eth0                        |Eth0
	 |                            |
----------                    ----------
Fortigate1                    Fortigate2
----------                    ----------
     |                            |
	 |Eth1                        |Eth1
	 |                            |
Fortigate1VnicSet1          Fortigate2VnicSet

RouteTable in IP Networks(Inside Subnets).

Route            IP AddressPrefix     Next-Hop Vnicset      AdminDistance
DefaultRoute1    0.0.0.0/0            Fortigate1VnicSet     		0
DefaultRoute2    0.0.0.0/0         	  Fortigate2Vnicset      		1

***The script will work for the above setup. The script will need to adjusted according to the routingtables in IPNetworks in OCI environment.

[OCI Environment Parameter Requirements]

The following variables will be required for the script. 

# Global OCI environment variables.
OCI_Identity_Domain = ""     Eg.  Compute-586773911 or Compute-acme
OCI_User = ""                Eg.  jack@acme.com
OCI_Password = ""            
OCI_Endpoint = ""            Eg.  https://api-z61.compute.us6.oraclecloud.com/authenticate/
Floating_Public_IP = ""      Eg.  Public Routable IP

# Global Fortigate variables in OCI environment.
Instance1 = ""               Name of 1st Fortigate in the Pair. Eg. Fortigate1
Instance2 = ""               Name of 2nd Fortigate in the Pair. Eg. Fortigate2
Route1 = ""                  Name of Route1 Eg. DefaultRoute1
Route2 = ""                  Name of Route2 Eg. DefaultRoute2

 

  
 