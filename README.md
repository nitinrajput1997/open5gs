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
**Note** We are deploying Open5gs and Ueransim on seperate VM's.

### Setup Open5GS

Upadte the Amf config file loacted at /etc/open5gs/amf.yaml by replacing given ip of ngap address to local eth0 ip.
And then restart amf service.
```bash
sudo systemctl restart open5gs-amfd
```
Upadte the Upf config file loacted at /etc/open5gs/upf.yaml by replacing given ip of ngap address to local eth0 ip.
And then restart upf service.
```bash
sudo systemctl restart open5gs-upfd
```
### NAT Port Forwarding
```bash
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo systemctl stop ufw
sudo iptables -I FORWARD 1 -j ACCEPT
```
### Access Open5gs Dashboard
```bash
sudo apt update
sudo apt install curl
curl -fsSL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt install nodejs
git clone https://github.com/open5gs/open5gs.git

# run webui with npm
cd webui
npm run dev --host 0.0.0.0

# the web interface will start on
http://localhost:3000
```
