-------------------------------------------------------------
-------------------------------------------------------------
CONFIGURACIÓN DEL SERVIDOR OPENVPN
-------------------------------------------------------------
-------------------------------------------------------------

sudo su -
apt install easy-rsa ssh openvpn
mkdir ~/easy-rsa
ln -s /usr/share/easy-rsa/* ~/easy-rsa/
cd ~/easy-rsa

-----------
-----------

nano vars

set_var EASYRSA_REQ_COUNTRY    "GT"
set_var EASYRSA_REQ_PROVINCE   "GT"
set_var EASYRSA_REQ_CITY       "GT"
set_var EASYRSA_REQ_ORG        "OpenVPN"
set_var EASYRSA_REQ_EMAIL      "rpereza4@miumg.edu.gt"
set_var EASYRSA_REQ_OU         "OpenVPN"
set_var EASYRSA_ALGO           "ec"
set_var EASYRSA_DIGEST         "sha512"

./easyrsa init-pki

-----------
-----------

./easyrsa build-ca nopass
cp ~/easy-rsa/pki/ca.crt /usr/local/share/ca-certificates/
update-ca-certificates
./easyrsa gen-req server nopass
./easyrsa sign-req server server
cp ~/easy-rsa/pki/private/server.key /etc/openvpn/server/
cp ~/easy-rsa/pki/issued/server.crt /etc/openvpn/server/
cp ~/easy-rsa/pki/ca.crt /etc/openvpn/server/
openvpn --genkey --secret ta.key
cp ta.key /etc/openvpn/server
mkdir -p ~/client-configs/keys
./easyrsa gen-req client1 nopass
cp pki/private/client1.key ~/client-configs/keys/
./easyrsa sign-req client client1
cp pki/issued/client1.crt ~/client-configs/keys/
cp ~/easy-rsa/ta.key ~/client-configs/keys/
cp /etc/openvpn/server/ca.crt ~/client-configs/keys/
cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz /etc/openvpn/server/
gunzip /etc/openvpn/server/server.conf.gz

-----------
-----------

nano /etc/openvpn/server/server.conf

;dh dh2048.pem
dh none
push "route 192.168.1.1 255.255.255.0"
;tls-auth ta.key 0 # This file is secret
tls-crypt ta.key
;cipher AES-256-CBC
cipher AES-256-GCM
auth SHA256
user nobody
group nogroup

-----------
-----------

nano /etc/sysctl.conf

net.ipv4.ip_forward = 1
sysctl -p

-----------
-----------

nano /etc/ufw/before.rules

# START OPENVPN RULES
# NAT table rules
*nat
:POSTROUTING ACCEPT [0:0]
# Allow traffic from OpenVPN client to eth0 (change to the interface you discovered!)
-A POSTROUTING -s 10.8.0.0/8 -o eth0 -j MASQUERADE
COMMIT
# END OPENVPN RULES

-----------
-----------

nano /etc/default/ufw
DEFAULT_FORWARD_POLICY="ACCEPT"

ufw allow 1194/udp
ufw disable
ufw enable

systemctl -f enable openvpn-server@server.service
systemctl start openvpn-server@server.service
systemctl status openvpn-server@server.service

mkdir -p ~/client-configs/files
cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf ~/client-configs/base.conf

-----------
-----------

nano ~/client-configs/base.conf

remote 81.xxx.xxx.xxx 1194
user nobody
group nogroup
;ca ca.crt
;cert client.crt
;key client.key
;tls-auth ta.key 1
cipher AES-256-GCM
auth SHA256
key-direction 1

; script-security 2
; up /etc/openvpn/update-resolv-conf
; down /etc/openvpn/update-resolv-conf

; script-security 2
; up /etc/openvpn/update-systemd-resolved
; down /etc/openvpn/update-systemd-resolved
; down-pre
; dhcp-option DOMAIN-ROUTE .

-----------
-----------

nano ~/client-configs/make_config.sh

#!/bin/bash

# First argument: Client identifier

KEY_DIR=~/client-configs/keys
OUTPUT_DIR=~/client-configs/files
BASE_CONFIG=~/client-configs/base.conf

cat ${BASE_CONFIG} \
    <(echo -e '<ca>') \
    ${KEY_DIR}/ca.crt \
    <(echo -e '</ca>\n<cert>') \
    ${KEY_DIR}/${1}.crt \
    <(echo -e '</cert>\n<key>') \
    ${KEY_DIR}/${1}.key \
    <(echo -e '</key>\n<tls-crypt>') \
    ${KEY_DIR}/ta.key \
    <(echo -e '</tls-crypt>') \
    > ${OUTPUT_DIR}/${1}.ovpn

-----------
-----------

chmod 700 ~/client-configs/make_config.sh
cd ~/client-configs
./make_config.sh client1

-------------------------------------------------------------
-------------------------------------------------------------
PARA CREAR NUEVOS CLIENTES EJECUTAR LOS SIGUIENTES COMANDOS EN EL SERVIDOR:
-------------------------------------------------------------
-------------------------------------------------------------

cd ~/easy-rsa
./easyrsa gen-req client2 nopass
cp pki/private/client2.key ~/client-configs/keys/
./easyrsa sign-req client client2
cp pki/issued/client2.crt ~/client-configs/keys/
cd ~/client-configs
./make_config.sh client2

-------------------------------------------------------------
-------------------------------------------------------------
PARA INSTALAR EL CLIENTE OPENVPN EN UBUNTU:
-------------------------------------------------------------
-------------------------------------------------------------

sudo su -
apt update
apt install openvpn
sudo openvpn --config client2.ovpn


------------------------------------------------------------
------------------------------------------------------------

systemctl stop openvpn-server@server.service
systemctl start openvpn-server@server.service
systemctl status openvpn-server@server.service


------------------------------------------------------------
------------------------------------------------------------
