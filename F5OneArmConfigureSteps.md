## Steps to configure F5 One Arm LTM

1.	Opening and logging to F5 WebUI page:
Connect to the F5 WebUI using the Floating/Public IP of the WebUI like below.
https://169.148.32.21:8443 (here 169.148.32.21 is the Public IP and 8443 is the port number. If 8443 doesn’t work, then try with port 443)

2.	Creating Nodes:
Local Traffic -> Nodes -> Node List -> Create 
After selecting above, it will open a form.  Fill the “Name”, “Description” and “Server Address” of the backend server. Finally click “Finished”.
Repeat the above step, if you want to add more backend servers. 

3.	Creating Server Pool:
Local Traffic -> Pools -> Pool List -> Create
After selecting above, it will open a form. Fill the “Name” and “Description” of the pool. In “Health Monitors” select “gateway_icmp” from “Available” list and click “<<” to bring it to “Active” list. 
Select “Load Balancing Method” as a “Round Robin”. In “New Member” section select “Node List”. Then select the Nodes from the list which you added in previous step. Mention the “Service Port” for each node and click “Add”. 
Similarly, multiple nodes can be added to this Pool. All members of a single pool should have same Service Port. Finally click “Finished”.

4.	Creating Virtual Server:
Local Traffic -> Virtual Servers -> Virtual Servers List -> Create
After selecting above, it will open a form to create virtual server.  Fill “Name”, “Description” of the Virtual Server. 
Select "Type" as "Performance(Layer 4)". Fill “Source Address” as 0.0.0.0/0 for allowing all the IP address, “Destination Address” as your Virtual Server IP address (Load balancer IP) which you want to send traffic for load balance, “Service Port” for load balancing. 
For "Protocol" select "TCP" and for "Protocol Profile (Client)" select fastL4.
Select “Source Address Translation” as “Auto Map” and select “Default Pool” in the drop down as the one you created in Step 3 above.  Finally click Finished.

5.	Checking Final Configuration Summary:
Local Traffic -> Network Map to see the configuration summary.
Now, we got the virtual server configured.  This Virtual Server IP is used to forward the traffic between Clients and Servers.
