# Install-CSF-Firewall-on-Ubuntu-20.04

CSF, also known as a “Config Server Firewall,” is one of the most popular and useful firewall applications for Linux.

## Step 1 – Create A ubuntu Server

## Step 2 – Install Required Dependencies

First, you will need to install some dependencies required for CSF. You can install all the dependencies with the following command:
```
apt-get install sendmail dnsutils unzip git perl iptables libio-socket-ssl-perl libcrypt-ssleay-perl 
libnet-libidn-perl libio-socket-inet6-perl libsocket6-perl -y
```

## Step 3 – Install CSF

By default, CSF is not available in the Ubuntu standard repository, so you will need to download it from their official website.
```
wget http://download.configserver.com/csf.tgz
```
Once downloaded, extract the downloaded file with the following command:
```
tar -xvzf csf.tgz
```

Next, change the directory to the extracted directory and install CSF by running the install.sh script:
```
cd csf
bash install.sh
```


Once the installation has been completed successfully, you should get the following output:
```
Installation Completed
```
Next, verify whether all required Iptables modules are installed with the following command:
```
perl /usr/local/csf/bin/csftest.pl
```
If everything is fine, you should get the following output:
```
Testing ip_tables/iptable_filter...OK
Testing ipt_LOG...OK
Testing ipt_multiport/xt_multiport...OK
Testing ipt_REJECT...OK
Testing ipt_state/xt_state...OK
Testing ipt_limit/xt_limit...OK
Testing ipt_recent...OK
Testing xt_connlimit...OK
Testing ipt_owner/xt_owner...OK
Testing iptable_nat/ipt_REDIRECT...OK
Testing iptable_nat/ipt_DNAT...OK
```
RESULT: csf should function on this server


## Step 4 – Configure CSF

Next, you will need to configure CSF as per your security standard. You can configure it by editing the file /etc/csf/csf.conf:
```
vi /etc/csf/csf.conf
```
Services using the open ports:
```
Port 20: FTP data transfer
Port 21: FTP control
Port 22: Secure shell (SSH)
Port 25: Simple mail transfer protocol (SMTP)
Port 53: Domain name system (DNS)
Port 80: Hypertext transfer protocol (HTTP)
Port 110: Post office protocol v3 (POP3)
Port 113: Authentication service/identification protocol
Port 123: Network time protocol (NTP)
Port 143: Internet message access protocol (IMAP)
Port 443: Hypertext transfer protocol over SSL/TLS (HTTPS)
Port 465: URL Rendesvous Directory for SSM (Cisco)
Port 587: E-mail message submission (SMTP)
Port 993: Internet message access protocol over SSL (IMAPS)
Port 995: Post office protocol 3 over TLS/SSL (POP3S)
```

Change the following line per your requirements:
```
TESTING = "0"
RESTRICT_SYSLOG = "3"
TCP_IN = "20,21,22,25,53,80,110,143,443,465,587,993,995"

# Allow outgoing TCP ports
TCP_OUT = "20,21,22,25,53,80,110,113,443,587,993,995"

# Allow incoming UDP ports
UDP_IN = "20,21,53,80,443"

# Allow outgoing UDP ports
# To allow outgoing traceroute add 33434:33523 to this list
UDP_OUT = "20,21,53,113,123"

# Allow incoming PING. Disabling PING will likely break external uptime
# monitoring
ICMP_IN = "1"
```
Save and close the file, then restart the CSF with the following command:

```
csf -r
```
Next, run the following command to list all Iptables rules:
```
csf -l
```

You should get the following output:
```
iptables mangle table
=====================
Chain PREROUTING (policy ACCEPT 51 packets, 3332 bytes)
num pkts bytes target prot opt in out source destination

Chain INPUT (policy ACCEPT 46 packets, 3014 bytes)
num pkts bytes target prot opt in out source destination

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
num pkts bytes target prot opt in out source destination

Chain OUTPUT (policy ACCEPT 26 packets, 15816 bytes)
num pkts bytes target prot opt in out source destination

Chain POSTROUTING (policy ACCEPT 26 packets, 15816 bytes)
num pkts bytes target prot opt in out source destination

iptables raw table
==================
Chain PREROUTING (policy ACCEPT 51 packets, 3332 bytes)
num pkts bytes target prot opt in out source destination

Chain OUTPUT (policy ACCEPT 27 packets, 15972 bytes)
num pkts bytes target prot opt in out source destination

iptables nat table
==================
Chain PREROUTING (policy ACCEPT 19 packets, 1410 bytes)
num pkts bytes target prot opt in out source destination

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
num pkts bytes target prot opt in out source destination

Chain OUTPUT (policy ACCEPT 1 packets, 76 bytes)
num pkts bytes target prot opt in out source destination

Chain POSTROUTING (policy ACCEPT 1 packets, 76 bytes)
num pkts bytes target prot opt in out source destination
```

## Step 5 – Enable CSF Web UI

CSF provides a web-based interface to manage the firewall from a web browser. By default, it is disabled in the CSF default configuration file, so you will need to enable it first.

Edit the CSF main configuration file with the following command:

```
vi /etc/csf/csf.conf
```

Change the following lines:
```
#Enable Web UI
UI = "1"

#Listening Port
UI_PORT = "8080"

#Admin username
UI_USER = "admin"

#Admin user password

UI_PASS = "your-password"

#Listening Interface
UI_IP = ""
```
Save and close the file when you are finished. Then, you will need to edit the /etc/csf/ui/ui.allow file and add your server IP and remote machine IP from where you want to access the CSF web UI.
```
vi /etc/csf/ui/ui.allow
```
Add your server IP and remote machine IP:
```
your-server-ip
remote-machine-ip
```
Save and close the file, then restart the CSF and LFD service to apply the changes:
```
csf -r
service lfd restart
```
## Step 6 Blocking and Allowing IP Addresses in CSF Firewall

One of the key functionalities of a firewall is the ability to allow or block IP addresses from accessing the server. With CSF, you can whitelist (allow), blacklist (deny) or ignore IP addresses by modifying the following configuration files:

```
csf.allow
csf.deny
csf.ignore
```
### Block an IP Address in CSF

To block an IP address, simply access the csf.deny configuration file.
```
$ sudo nano /etc/csf/csf.deny
```
Then specify the IP addresses that you want to block. You can specify the IP addresses line by line as shown:
```
192.168.100.50
192.168.100.120
```
Or you can use the CIDR notation to block an entire subnet.

192.168.100.0/24

### Allow an IP Address in CSF
To allow an IP address through Iptables and exclude it from all filters or blocks, edit the csf.allow configuration file.
```
$ sudo nano /etc/csf/csf.allow
```

### Exclude an IP Address in CSF

Additionally, CSF provides you with the ability to exclude an IP address from IPtables or filters. Any IP address in the csf.ignore file will be exempted from the iptables filters. It can only be blocked if specified in the csf.deny file.

To exempt an IP address from the filters, access the csf.ignore file.
```
$ sudo nano /etc/csf/csf.ignore
```
