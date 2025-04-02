<p align="center" height="30%" width="30%">
<img width="325" alt="image" src="https://github.com/user-attachments/assets/5de22ea0-0d6c-47ac-b0e2-81eed3992388" alt="DNS LOGO"/>
</p>

<h1>DNS Lab</h1>
DNS (Domain naming server/system) converts computer names and website names that humans understand into binary code that is understood by computers to locate resources. DNS is often integrated with Active Directory. (If you have been following my Active Directory repositories, you'll know that we chose to automatically install DNS when we installed the Active Directory Domain Server, turning dc-1 into a domain controller. So we already have a DNS server both installed and running on our domain controller dc-1).
<p></p>
In this lab, we will gain DNS familiarity using ping, A-Records, and CNAME using dc-1 and client-1 virtual machines which were created in the active directory lab linked in the requirements below.

<br />

<h2>Video Demonstration</h2>

- ### [YouTube Demonstration DNS- coming soon](https://www.youtube.com)

<h2>Environments Used </h2>

- Windows 10 (21H2)
- Windows Server
- Microsoft Azure (Virtual Machines/Compute)
- Active Directory
- Active Directory Users and Computers
  
<h2>Languages and Utilities Used</h2>

- PowerShell
- DNS-specific tools
- Remote Desktop

<h2>Requirements</h2>

- Active Directory must be running in Azure on VM dc-1
- Client virtual machine must be running in Azure (client-1) and joined to the domain
- Both of these requirements have step-by-step details in my repositories:
  - [Active Directory Set Up](https://github.com/VictoriaDeery/ActiveDirectorySetUp/blob/main/README.md)
  - [Active Directory Functionality](https://github.com/VictoriaDeery/ActiveDirectoryLab-pt2/blob/main/README.md)
    
<h2>Objectives:</h2>

- A. Inspect DNS A-records on the server (hostname to IP address mappings)
- B. Create and test A-Records
- C. Manage DNS A-Records and Cache 
- D. Create a "CNAME" record (Mapping one human readable name to another)
- E. Explore Root Hints

<h2>Overview: :</h2>
<p> 
 Our dc-1 domain controller also serves as a DNS server with A records mapping hostnames to IPs. Existing records include dc-1.mydomain.com (10.0.0.4) and client-1.mydomain.com (10.0.0.5), created automatically. When client-1 pings a hostname (e.g., "mainframe"), it checks its cache first, followed by the host file, and finally the DNS server. Client-1 first checks its local cache (stored in memory) for prior interactions with this computer, as it's the quickest method. If there's no result in the cache, it consults the host file for local domain name to IP address mappings. If that fails, it queries the DNS server over the network for the IP of "mainframe.com."
<P>
Again, our DNS server in this lab is dc-1. So dc-1 will check its record for "mainframe" and if it doesn't have it, then the ping will fail, and mainframe host will not be located. To resolve this, we will create an A record for mainframe.mydomain.com on the DNS server and map it to any IP (even 10.0.0.4). A subsequent ping will succeed, not from the cache nor hostfile, but from the configured DNS server returning the mapped IP address to client-1. This IP is then stored in the local cache of client-1.
<P>
If you later update the mainframe A record in the DNS server to point to a different IP address (such as 8.8.8.8), client-1 still has its previous IP address in the cache (10.0.0.4). Pinging mainframe now would attempt to ping the previously mapped IP address even though the main record has changed. To resolve this, run ipconfig /flushdns on client-1 to clear the local cache. This will cause ping will go out over the network to query the DNS server again. 
<P>
 
<h2>Objectives Tutorial:</h2>

 <p>
 <br />
- <b>A. Inspect DNS A-records on the server (hostname to IP address mappings)</b>
<p>
<img src="https://github.com/user-attachments/assets/412f0e44-cf11-4ae9-87e0-3cf97abe8377" height="80%" width="80%"/>
<img src="https://github.com/user-attachments/assets/85712f86-a29f-4b3e-9767-bef7c3f8951d" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<p></p>

  A1. log into both dc-1 and client-1 as an admin like jane_admin. First use client-1. client-1 ->PowerShell -> ping mainframe. It will tell you it cannot be found. To see the local DNS cache type ipconfig /displaydns. If you want the cache to be in a text file called test, type ipconfig /displaydns > test.txt and hit enter to run. To open it, type notepad test.txt. Here, you can do control+F and confirm there is no mainframe here. Now to view the local host file, open notepad as an admin ->file -> open -> change file type to all files -> windows -> systemm32 ->drivers -> etc -> hosts to view the local hostfile which is where you can map IP addresses to hostname, for example if you want to assign unicorn to the local loopback address 127.0.0.1 then ping will find it in the hostfile. Even typing nslookup mainframe, would not find it at this point because there is no DNS record. So, let's create a DNS A record on dc-1 for "mainframe" and have it point to dc-1's private IP address.
</p>
- <b>B. Create and test A-Records on the server and observe them from the client</b>
<p>
  <p></p>
<img src="https://github.com/user-attachments/assets/8f1a01e0-9e82-4464-b172-e60499745723" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<p>
  B1. We will create an A-Record on the server and then observe them from the client. Now use dc-1. Get and copy dc-1's private IP address either from Azure or in dc-1 PowerShell type ipconfig and it will tell you. click the windows symbol -> administrative tools -> DNS to open the DNS server ->dc-1 -> forward lookup zones -> mydomain.com. click mydomain.com to see different records. Right-click in the records space -> new host (A or AAAA) -> set the name and IP Address as in the picture above (mainframe & dc-1's private IP). It is possible to have two records pointing to the same IP address.
  <p>
<img src="https://github.com/user-attachments/assets/b6c26c2a-ce07-41b3-a422-d50d2a9764c2" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://github.com/user-attachments/assets/9fc4b6f4-2d43-48de-9006-c8b5e14d8238" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<p>
    B2. Go back to being on client-1 PowerShell and ping mainframe successfully via the DNS server.
  </p>
</p>
- <b>C. Manage DNS A-Records and Cache </b>
<p>
  <p></p>
<img src="https://github.com/user-attachments/assets/c1b51ca2-ff44-493b-aeaf-be72242bb27c" height="80%" width="80%" alt="Disk Sanitization Steps"/>
  <p>
<img src="https://github.com/user-attachments/assets/228291c7-6d6b-4749-a0fd-f684713739ab" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://github.com/user-attachments/assets/f5bd3aa8-d436-4ae3-bb87-8cb345dfdbfc" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://github.com/user-attachments/assets/1d42076c-98a4-4e7c-90a9-ea967abcd9db" height="80%" width="80%" alt="Disk Sanitization Steps"/>

  C1. Now we will delete records from the server and observe the client DNS cache on the client to gain insight. In dc-1 change the A record to point to 8.8.8.8. instead of 10.0.0.4. Double-click the mainframe record you made and change the IP address. In client-1, PowerShell ping mainframe again. You'll notice it outputs the old IP address because that is what has been saved in the cache, which is first looked to. even typing ipconfig /displaydns will display the old IP in this local DNS cache. So, we will flush the cache and observe it as empty. Open PowerShell, run it as an admin, and type ipconfig /flushdns to flush it, and then use ipconfig /displaydns to observe that. So now, pinging mainframe will get the new IP address 8.8.8.8. Flushdns is helpful, especially if only one person cannot access a resource when others can.
</p>
- <b>D. Create a "CNAME" record (Mapping one human readable name to another)</b>
<p>
  <p></p>
<img src="https://github.com/user-attachments/assets/e504b551-bbdb-4f92-b208-ae1102264cfb" height="80%" width="80%" alt="Disk Sanitization Steps"/>
  <p>
<img src="https://github.com/user-attachments/assets/c0d6a829-ee35-46a5-bbee-1f622f279505" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<p>
  D1. In dc-1, map "search" to "www.google.com" as the fully qualified domain name (FQDN). In the DNS server, right-click in our domain like before (under mainframe) -> new alias (CNAME). Now in client-1 ping search. Since the name doesn't match the certificate of google.com, searching "search" won't work.
</p>
- <b>E. Explore Root Hints</b>
<p>
  <p></p>
  E1. Lastly,  root hints are a network of hundreds of servers that is automatically installed when a domain controller is installed and opts to install DNS. Since our domain controller doubles as a DNS server, it is responsible for resolving all requests from the network, such as knowing Google's IP address. However, if the local server doesn't know where to resolve a top-level domain, then it uses root hints.
</p>
<h2>Next Steps</h2>
Now that you understand DNS basics, take the next step by integrating DNS with other services like web hosting or diving deeper into Active Directory configurations. You can also explore my repositories on osTicket setup for managing support tickets or VPNs for enhancing secure connectivity. Visit my main page to discover even more projects! 
 <p>
   
  - [Active Directory Set Up](https://github.com/VictoriaDeery/ActiveDirectorySetUp/blob/main/README.md)
  - [osTicket: Post-Installation Configuration](https://github.com/victoriadeery/post-install-config)
  - [Virtual Private Networks (VPNs)](https://github.com/victoriadeery/Virtual-Private-Networks-VPNs)  

 
 <br />
