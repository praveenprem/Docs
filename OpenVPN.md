# **_OpenVPN installation and Configuration_**

This documentation is for OpenVPN server installation and configuration for _**username**_ and _**password**_ authentication with _**Certificate**_ Authority and _**HMAC**_

---

## Table of content

* [Download and installation](#installation)
* [RSA template](#template)
* [CA variables configuration](#varsConfig)
* [Build Certificate authority](#buildCA)
* [Create the Server Certificate, Key, and Encryption Files](#serverKeyFiles)
* [Configure the OpenVPN service](#serviceConfig)
* [Allow IP forwarding](#ipForwarding)
* [Firewall rules and connection masquerade](#firewallRules)
* [Enable default firewall forwarded packets](#packetsForwading)
* [Open server port and enable changes](#enableServerChanges)
* [Start and enable the VPN server](#startServer)
* [Creating client config file](#clientConfig)

---
<a name="installation"/>

### Download and installation

```
apt -y update
apt -y install openvpn easy-rsa
```
<a name="template"/>

### Copy `easy rsa` template

```
make-cadir ~/openvpn-ca
```

<a name="varsConfig"/>

### Configure CA variables
Edit `vars` file and update following
```
export KEY_COUNTRY=
export KEY_PROVINCE=
export KEY_CITY=
export KEY_ORG=
export KEY_EMAIL=
export KEY_OU=

export KEY_NAME=
```

<a name="buildCA"/> 

### Build Certificate authority
Source the variables
```
source vars
```
Clean the previous environment(CA, TA and KEYs)
```
./clean-all
```
Build the CA
```
./build-ca
```

<a name="serverKeyFiles"/>

### Create the Server Certificate, Key, and Encryption Files
Generate server certificate and key pair.
###### Leave password fields empty, unless otherwise required
```
./build-key-server server
```
Generate Diffie-Hellman key
```
./build-dh
```
Generate HMAC signature
```
openvpn --genkey --secret keys/ta.key
```

<a name="serviceConfig"/>

### Configure the OpenVPN service
Copy all the keys to `/etc/openvpn` folder
```
cd ~/openvpn-ca/keys
sudo cp ca.crt server.crt server.key ta.key dh2048.pem /etc/openvpn
```
Copy the default server template to the service folder
```
gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz | sudo tee /etc/openvpn/server.conf
```
Delete the existing content and add the following.

###### _Server port_
```
port 1194
```
###### _Connection protocol_
```
proto tcp
```
###### _Device type_
```
dev tun
```
###### _Certificate authority_
```
ca /etc/openvpn/ca.crt
```
###### _Server certificate_
```
cert /etc/openvpn/server.crt
```
###### _Server key_
```
key /etc/openvpn/server.key  # This file should be kept secret
```
###### _Diffie-Hellman key_
```
dh /etc/openvpn/dh2048.pem
```
###### _VPN IP and Subnet mask_
```
server 10.8.0.0 255.255.255.0
```
```
ifconfig-pool-persist ipp.txt
```
```
push route 192.168.1.0 255.255.255.0
```
```
keepalive 10 120
```
###### _HMAC key_
```
tls-auth ta.key 0 # This file is secret
```
###### _Key direction_
```
key-direction 0
```
###### _Cryptographic ciphers_
```
cipher AES-128-CBC   # AES
```
###### _HMAC message digest algorithm_
```
auth SHA256
```
###### _Compression algorithm_
```
comp-lzo
```
###### _User and Group settings_
```
user nobody
```
```
group nogroup
```
###### _Prevent key files re-read_
```
persist-key
```
###### _Preserve most recent IP address/port_
```
persist-tun
```
###### _Client certificate authentication removal_
```
client-cert-not-required
```
###### _Linux system account authentication plugin_
```
plugin /usr/lib/openvpn/openvpn-plugin-auth-pam.so "login"
```
```
status openvpn-status.log
```
```
verb 3
```

<a name="ipForwarding"/>

### Allow IP forwarding
Edit the `/etc/sysctl.conf` and uncomment following line
```
net.ipv4.ip_forward=1
```
Confirm the above change with following command.
```
sudo sysctl -p
```

<a name="firewallRules"/>

### Firewall rules and connection masquerade
Add new firewall default policy for `postrouting` on `nat` table with `masquerade`
```
*nat
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING -s 10.8.0.0/8 -o *interface* -j MASQUERADE
COMMIT
```
Replace the `*interface*` with interface name. Use following command to find the default interface.
```
ip route | grep default
```

<a name="packetsForwading"/>

### Enable default firewall forwarded packets
Edit `/etc/default/ufw` and update the existing rule with this:
```
DEFAULT_FORWARD_POLICY="ACCEPT"
```

<a name="enableServerChanges"/>

### Open server port and enable changes
Allow server port on the firewall
```
sudo ufw allow *port*/*protocol*
```
Allow SSH connections
```
sudo ufw allow OpenSSH
```
Disable and enable the UFW firewall
```
sudo ufw disable
sudo ufw enable
```

<a name="startServer"/>

### Start and enable the VPN server
Start the OpenVPN service
```
sudo systemctl start openvpn@server
```
###### Where `@server` is the name of the config file
Check the service status
```
sudo systemctl status openvpn@server
```
Check the new network interface config
```
ip addr show tun0
```
Enable service on system boot
```
sudo systemctl enable openvpn@server
```

<a name="clientConfig"/>

### Creating client config file
###### _Define config file type_
```
client
```
###### _Device type_
```
dev tun
```
###### _Server protocol_
```
proto tcp
```
###### _Server address with port_
```
remote 51.x.x.x 1194
```
###### _Connection retry limit_
```
resolv-retry infinite
```
###### _Client side port selection_
```
nobind
```
###### _User and Group settings_
```
user nobody
```
```
group nogroup
```
###### _Prevent key files re-read_
```
persist-key
```
###### _Preserver most recent IP address/port_
```
persist-tun
```
###### _Username and Password authentication_
```
auth-user-pass
```
###### _Key direction_
```
key-direction 1
```
###### _Cryptographic ciphers_
```
cipher AES-128-CBC
```
###### _HMAC message digest algorithm_
```
auth SHA256
```
###### _Compression algorithm_
```
comp-lzo
```
###### _Certificate authority block_
```
<ca>
-----BEGIN CERTIFICATE-----
MIIE2TCCA8GgAwIBAgIJAKBW5A69CWnY..........
..........................................
..........................................
..........................................
..........................................
..........................................
..........................................
..........................................
..........................................
..........................................
..........................................
..........................................
..........................................
..........................................
..........................................
..........................................
..........................................
..........................................
..........................................
.............OjmpGakx5+OWSQ3vzSTk2B3f8tz
-----END CERTIFICATE-----
</ca>
```
###### _HMAC key block_
```
<tls-auth>
-----BEGIN OpenVPN Static key V1-----
ac34d............................
.................................
.................................
.................................
.................................
.................................
.................................
.................................
.................................
.................................
.................................
.................................
.................................
.................................
.................................
............................a3d97
-----END OpenVPN Static key V1-----
</tls-auth>
```
