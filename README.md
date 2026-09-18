<h1> Lateral movement / Credential abuse detection
</h1>



<h2>Summary </h2>
Detects Kerberoasting: an attacker with any valid even a low privilege domain account enumerates Service Principal Names in Active Directory and requests Kerberos service tickets for each one. Because service tickets are encrypted with the target service account's password hash, the attacker takes the tickets offline and attempts to crack them, turning a single authenticated foothold into a path toward privileged service account credentials, with no elevated access required to trigger it.
<br />
<br />

<br /> Lab Environment <br />
<img src="https://imgur.com/ys3H2oi.jpg"  height="80%" width="80%">
<br /> Domain Controller: Windows Server, AD DS role, one SPN registered service account seeded as the Kerberoastable target. <br />
<br />Windows 10: domain joined client.<br />
<br />Kali Linux: attacker host, Impacket toolset<br />
<br />Wazuh Manager: SIEM/detection platform<br />
<br />Wazuh agent installed on the DC, collecting the Security event channel via eventchannel log format<br />
<br />Advanced Audit Policy enabled on the DC: Kerberos Service Ticket Operations (Success/Failure), Kerberos Authentication Service (Success/Failure)<br />
<img src="https://imgur.com/4ZxtLDu.jpg"  height="80%" width="80%">


<br  />
<br />

<br />
<h2> Attack Simulation</h2>
<br />
From Kali, using a standard non privileged domain user account, this enumerates every SPN registered account in the domain and requests a TGS for each, printing crackable ticket hashes (RC4-HMAC format). 
<br />
<img src="https://imgur.com/e11fdfw.jpg"  height="80%" width="80%">
<img src="https://imgur.com/82L4Dv9.jpg"  height="80%" width="80%">

<br />
<br />

<h2>Telemetry Generated </h2>

<br />Each ticket request produces a Windows Event ID 4769 on the DC. The indicators used for detection. Ticket Encryption Type is 0x17 (RC4 HMAC) legitimate modern Kerberos deployments use AES (0x11/0x12); RC4 usage against a non machine service account is anomalous. <br />

<br />
<img src="https://imgur.com/1qrEGvr.jpg"  height="80%" width="80%">
<img src="https://imgur.com/F4oEPzx.jpg"  height="80%" width="80%">
<br />
<br />
<h2>Detection Logic (Wazuh custom rules) </h2>
<img src="https://imgur.com/C2xmOjf.jpg"  height="80%" width="80%">
<br />
<h2>Validation </h2>

<br />Confirmed rule 100021 fires in the Wazuh Dashboard during simulated attack runs.<br />
<br />Confirmed the base 4769 events decode correctly with Ticket Encryption Type, Service Name, and Account Name fields populated as expected.<br />
<br />Cracked at least one seeded weak service account password with hashcat to demonstrate real impact, not just detection.<br />

<br />

<p align="center">
<br/>
