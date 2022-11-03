# open5gs

### Install Open5GS
We are deploying Open5gs as a native Linux daemon service application.
```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository ppa:open5gs/latest
sudo apt update
sudo apt install open5gs
```
**Note :** We are deploying Open5gs and Ueransim on seperate VM's say server1 and server2 respectively.

### Setup Open5GS

Upadte the Amf config file loacted at /etc/open5gs/amf.yaml by replacing given ip of ngap address to local eth0 ip.
And then restart amf service.
```bash
sudo systemctl restart open5gs-amfd
```
Check logs of amf
```bash
sudo tail -f /var/log/open5gs/amf.log
```
shows like this
```
11/07 04:17:26.737: [amf] INFO: [Removed] Number of AMF-UEs is now 0 (../src/amf/context.c:1268)
11/07 04:17:26.738: [sctp] INFO: AMF terminate...done (../src/amf/app.c:42)
Open5GS daemon v2.3.6
11/07 04:17:26.762: [app] INFO: Configuration: '/etc/open5gs/amf.yaml' (../lib/app/ogs-init.c:129)
11/07 04:17:26.762: [app] INFO: File Logging: '/var/log/open5gs/amf.log' (../lib/app/ogs-init.c:132)
11/07 04:17:26.764: [sbi] INFO: nghttp2_server() [127.0.0.5]:7777 (../lib/sbi/nghttp2-server.c:144)
11/07 04:17:26.764: [amf] INFO: ngap_server() [172.31.10.16]:38412 (../src/amf/ngap-sctp.c:53)
11/07 04:17:26.764: [sctp] INFO: AMF initialize...done (../src/amf/app.c:33)
```

Upadte the Upf config file loacted at /etc/open5gs/upf.yaml by replacing given ip of gtpu address to local eth0 ip.
And then restart upf service.
```bash
sudo systemctl restart open5gs-upfd
```
Check logs of smf
```bash
sudo tail -f /var/log/open5gs/upf.log
```

shows like this
```
11/07 04:18:19.224: [app] INFO: SIGTERM received (../src/main.c:53)
11/07 04:18:19.224: [app] INFO: Open5GS daemon terminating... (../src/main.c:212)
11/07 04:18:19.225: [upf] INFO: PFCP de-associated (../src/upf/pfcp-sm.c:178)
11/07 04:18:19.225: [upf] INFO: [Removed] Number of UPF-sessions is now 0 (../src/upf/context.c:190)
11/07 04:18:19.226: [app] INFO: UPF terminate...done (../src/upf/app.c:39)
Open5GS daemon v2.3.6
11/07 04:18:19.243: [app] INFO: Configuration: '/etc/open5gs/upf.yaml' (../lib/app/ogs-init.c:129)
11/07 04:18:19.243: [app] INFO: File Logging: '/var/log/open5gs/upf.log' (../lib/app/ogs-init.c:132)
11/07 04:18:19.256: [pfcp] INFO: pfcp_server() [127.0.0.7]:8805 (../lib/pfcp/path.c:30)
11/07 04:18:19.256: [gtp] INFO: gtp_server() [172.31.10.16]:2152 (../lib/gtp/path.c:30)
11/07 04:18:19.257: [app] INFO: UPF initialize...done (../src/upf/app.c:31)
```

### NAT Port Forwarding
In order to bridge between the 5G Core UPF and Internet, we need enable IP forwarding and add a NAT rule to the IP Tables. Following are the NAT port forwarding we have to do. Without this port forwarding the connectivity from 5G Core to internet would not work.
```bash
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo systemctl stop ufw
sudo iptables -I FORWARD 1 -j ACCEPT
```
### Access Open5gs Dashboard
Now we need to access our open5gs dashboard.
```bash
sudo apt update
sudo apt install curl
curl -fsSL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt install nodejs
git clone https://github.com/open5gs/open5gs.git

# run webui with npm
cd webui
npm ci --no-optional && npm run build
npm run dev --host 0.0.0.0

# the web interface will start on
http://localhost:3000

# run this command if you are on remote serverand want to access dashboard locally
ssh -L localhost:3000:localhost:3000 ubuntu@ip
```
**Login credentials :**
**username** - admin
**password** - 1423

**Add new subscriber from dashboard :**
**IMSI**: 901700000000001,
**Subscriber Key**: 465B5CE8B199B49FAA5F0A2EE238A6BC,
**USIM Type**: OPc,
**Operator Key**: E8ED289DEBA952E4283B54E88E6183CA

### Install UERANSIM
On server2.

```bash
# install cmake and other packages
sudo apt update
sudo apt upgrade
sudo apt install iproute2
sudo snap install cmake --classic
sudo apt install gcc
sudo apt install g++
sudo apt install libsctp-dev
sudo apt install make
```
```bash
# clone ueransim
git clone https://github.com/aligungr/UERANSIM
cd UERANSIM
make
```
### Setup gNB
We have to do some changes to the gNB config files located in UERANSIM/config/open5gs-gnb.yaml. Update the "linkIp", "ngapIp", "gtpIp" field with local ip (server 2 ip) and "amfConfigs: address" field with amf ip ( server1 ip).

```bash
# start gnb with open5gc-gnb.yaml config file
sudo ./build/nr-gnb -c config/open5gs-gnb.yaml
```

### Setup UE
We have to do some changes to the UE config files located in UERANSIM/config/open5gs-ue.yaml. Update the "gnbSearchList" with the IP address of the server2.

```bash
# start gnb with open5gc-ue.yaml config file
sudo ./build/nr-ue -c config/open5gs-ue.yaml
```
