# PersonalCertificateAuthority

- Download and Setup 3 Microsoft Server 2022 vm instances with default settings and one normal Windows 10 Desktop. The Desktop doesnt need any setup.

- Designate one of those machines as the DC (Domain Controller), one as the RootCA and the other as the Subordinate CA.

- First, on the DC machine create a folder under the C drive called “temp”. The path should be this “C:\temp”

- Next, on the DC machine run this script FIRST in an Administrator Powershell ISE.



- Once the Script has successfully finished, open up Server Manager, press “Manage” then “Add Roles and Features”. In the prompt click next 3 times until you get to the “Server Roles” part.

- Install DNS, AD Domain Services, and AD Lightweight Directory Services.

- Click ok and next to everything and then install.

- AD CONFIGURATION

- “Add a new forest”, press next

- Provide a domain name “example.domain.com”. Press next

- Provide a password. Example “Password123”. Press next

- Skip DNS Options and press next.

- Additional Options press next.

- Keep defaults for Paths and press next

In Review Options verify things are correct and press next.

If you “All prerequisite checks passed”, press install

Open AD Users and Computers, create a user, make it a member of these groups

![image](https://github.com/Chris-Patrick/PersonalCertificateAuthority/assets/88513662/52abe0f0-ba28-484d-a03d-b49ed08ba311)

Next open a Admin PowerShell and enter the following command (Note: This is for test not production machines).

> Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False

Once completed Restart the DC Machine and switch over to our second Subordinate machine.

To setup the ROOT CA follow the instructions here
[a link](https://vmlabblog.com/2019/09/setup-server-2019-enterprise-ca-2-5-offline-root-ca/)

Next on the Subordinate machine we need to set it up to add it to the Domain we created on the DC machine.

Run the same PowerShell command listed above to disable the firewall. 

If you have these machines in VMWare, we need to make sure that both machines are on the same VNet and that they can talk to each other. To do that in VMWare right click the name of the machine > Settings > Network Adapter > Custom > Select and VMnet you want jsut make sure to do this on both machines and they are both on the same VMnet. 

To test this we will ping both machines using CMD

> ping <IPv4>

Replace IPv4 with the IP of the DC. You should do this on both machines so get the IP of both and test on both. If you get a “Reply” after running the command that means they can talk to each other. The Ping command needs to work on both machines because the machines need to talk in both directions, not only one.

DC Machine:

![image](https://github.com/Chris-Patrick/PersonalCertificateAuthority/assets/88513662/9180cddd-b575-4825-bd2c-b69a6a981228)


Subordinate Machine:

![image](https://github.com/Chris-Patrick/PersonalCertificateAuthority/assets/88513662/738c8c60-2c92-491e-9a8f-b60ece9fa984)


In the windows search bar type “View Network Connections” > Right click Ethernet > Properties Select “Internet Protocol Version 4 (TCP/IPv4)” (DONT UNCHECK THE BOX) > Properties > Select radio button “Use the following DNS server addresses” > Enter the DNS IP (10.10.10.12) > Press OK

On the subordinate machine go to Settings > System > About > Rename this PC (advanced) > Change > select the Domain radio button. Enter the name of the domain (example.domain.com). Enter your credentials to the DC Machine. You should see a Welcome to the domain message. Press Restart Now.

To setup the subordinate follow the instructions here
[a link](https://vmlabblog.com/2019/09/setup-server-2019-enterprise-ca-3-5-subordinate-ca/)

Go to the DC machine. Open AD users and computers. Go to Computers right click the SubCA machine > Manage > Local Users and Groups > Groups > Right Click IIS_IUSRS > Properties > Add. Add the user that you created with the permissions.

![image](https://github.com/Chris-Patrick/PersonalCertificateAuthority/assets/88513662/573f8355-1be6-4c9a-8c80-519d0c6c3ad9)


Go back to SubCA Machine. Setup ADCS again this time check every option. If you cant enable them install them with Add Roles and Features, then try again. If it asks for a service account use the account you created with the permissions. This should allow you to have a certsrv IIS server. 

Go to a Windows 10 desktop machine and go to the server http://<subordinate-machine-name>/certsrv. Make sure the machine is on the same network as everything else. 

If all is done correctly you should be able to enter credentials and access certsrv.
