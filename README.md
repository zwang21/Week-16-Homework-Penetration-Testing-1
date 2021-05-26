## Week 16 Homework Submission File: Penetration Testing 1

#### Step 1: Google Dorking

- Using Google, can you identify who the Chief Executive Officer of Altoro Mutual is:

	On google.com, find: 
	`site:demo.testfire.net intext:executive`

	I got this info:
	[Executives & Management - Altoro Mutual](demo.testfire.net/index.jsp?content=inside_executives.htm "Executives & Management - Altoro Mutual")

	Link: [demo.testfire.net/index.jsp?content=inside_executives.htm](demo.testfire.net/index.jsp?content=inside_executives.htm "Executives & Management - Altoro Mutual")

	**Karl Fitzgerald** is the Chairman & Chief Executive Officer.

- How can this information be helpful to an attacker:

	This information is useful to an attacker who wants to send phishing email directly to the executive members.

#### Step 2: DNS and Domain Discovery

Enter the IP address for `demo.testfire.net` into Domain Dossier and answer the following questions based on the results:

  1. Where is the company located: 
```
Sunnyvale, CA 94085 US
```
  2. What is the NetRange IP address:

	`65.61.137.64 - 65.61.137.127`

  3. What is the company they use to store their infrastructure:

```
CustName:       Rackspace Backbone Engineering
Address:        9725 Datapoint Drive, Suite 100
City:           San Antonio
StateProv:      TX
PostalCode:     78229
Country:        US
```
	
  4. What is the IP address of the DNS server:

	`65.61.137.117`

## Step 3: Shodan

- What open ports and running services did Shodan find:

```
https://www.shodan.io/host/65.61.137.117
Open Ports: 80, 443, 8080
```

#### Step 4: Recon-ng

- Install the Recon module `xssed`. 

```
marketplace install xssed
keys add shodan_api tbmKjcV0ont04nJMTcEuBsmnDQtzLFhe
modules load recon/domains-vulnerablilities/xssed
```

- Set the source to `demo.testfire.net`. 

`options set SOURCE demo.testfire.net`

- Run the module.
 
`run`

Is Altoro Mutual vulnerable to XSS:
 
Yes, 1 total (1 new) vulnerability found. As an example, input the following script in the Search box will result in a pop up window.

`<script>alert("www.sec-r1z.com")</script>`

### Step 5: Zenmap

Your client has asked that you help identify any vulnerabilities with their file-sharing server. Using the Metasploitable machine to act as your client's server, complete the following:

- Command for Zenmap to run a service scan against the Metasploitable machine: 
```
  The Metasploitable machine IP address is 192.168.0.10
  type zenmap in kali vm command line
  Input Target 192.168.0.10
  Profile: Quick scan
  I can see the raw nmap Command is nmap -T4 -A 192.168.0.10
  Click Scan
```
- Bonus command to output results into a new text file named `zenmapscan.txt`:

On `Zenmap` screen, click `Scan` menu, select `Save Scan (Ctrl+S)`, `Select File Type: Nmap text format (.nmap)`. Type file Name as: `zenmapscan.txt`. 
Or in command line, run `nmap -T4 -A 192.168.0.10 -oN zenmapscan.txt`

- Zenmap vulnerability script command: 
	I tried to run two scripts:
    - Click the **Profile** tab at the top and select **Edit Selected Profile**.
    - Click on the **Scripting** tab and view all the scripts that start with `ftp`.
    - Select the `ftp-vsftpd-backdoor` script by placing a check in the box.
    - Click **Save Changes** to save the profile settings.    
    
    the raw Nmap command that Zenmap will run:
	`nmap -T4 -F --script ftp-vsftpd-backdoor,smb-enum-shares 192.168.0.10`

	- `-T4`: Adjusts the speed of the scan. Ranging from 0-5, 4 is a fast and aggressive timing option.
    - `-F`: Indicates a fast scan by only scanning the 100 most common ports.
    - `--script`: Runs a scripted scan.
    - `ftp-vsftpd-backdoor, smb-enum-shares`: Indicates which scripted scan to run.
    - `192.168.0.10`: IP address of host that will be scanned.

- Once you have identified this vulnerability, answer the following questions for your client:

1. What is the vulnerability:

	Zenmap was able to enumerate the vulnerable service as follows:

``` 
PORT    STATE SERVICE
21/tcp  open  ftp
  ftp-vsftpd-backdoor:
	VULNERABLE:
	vsFTPd version 2.3.4 backdoor 
      State: VULNERABLE(Exploitable)
      IDs:  BID:48539  CVE:CVE-2011-2523
        vsFTPd version 2.3.4 backdoor, this was reported on 2011-07-04.
      Disclosure date: 2011-07-03
```

```
Host script results:
  smb-enum-shares:
	account_used: <blank>
	......
    \\192.168.0.10\IPC$:
	Type: STYPE_IPC
	Comment: IPC Service (metasploitable server (Samba 3.0.20-Debian))
	Max Users: <unlimited>
	Path: C:\tmp
	Anonymous access: READ/WRITE
	......
	\\192.168.0.10\tmp:
	Type: STYPE_DISKTREE
	Comment: oh noes!
	Max Users: <unlimited>
	Path: C:\tmp
	Anonymous access: READ/WRITE
  	......
```

2. Why is it dangerous:

The concept of the attack on VSFTPD 2.3.4 is to trigger the malicious vsf_sysutil_extra(); function by sending a sequence of specific bytes on port 21, which, on successful execution, results in opening the backdoor on port 6200 of the system and running as root. 

SMB vulnerabilities allow their payloads to spread laterally through connected systems.

3. What mitigation strategies can you recommendations for the client to protect their server:
	
The vsFTPd 2.3.4 backdoor reported on 2011-07-04 (CVE-2011-2523). The VSFTPD 2.3.4 service vulnerability is very unlikly an issue in a live situation because this version of VSFTPD is old nowadays and the vulnerable version was only available for one day.

A patch was released by Microsoft for SMB vulnerabilities in March 2017. Apply SMB patch is the best solution for Samba.

---
© 2020 Trilogy Education Services, a 2U, Inc. brand. All Rights Reserved.
