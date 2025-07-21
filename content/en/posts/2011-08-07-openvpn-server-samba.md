---
translationKey: "2011-08-07-openvpn-server-samba"
categories:
- linux
- openvpn
- server
date: "2011-08-07T11:00:00Z"
ref: 2014-08-17-openvpn-server-samba
title: "Set Up an OpenVPN Server with Samba Share"
---

> *This article was originally written in 2011. It has been revised for clarity and updated with more modern configurations. Technologies like OpenVPN and Samba are constantly evolving, so always consult the official documentation for the latest recommendations.*

This guide aims to set up a simple OpenVPN server to secure access to a local network and route all web traffic through it.

The local network uses the `192.168.0.0/24` address range. A server on this network, with the IP address `192.168.0.10`, will host OpenVPN and a DNS server (dnsmasq) to manage local DNS queries. We will call it "Server."

## OpenVPN Installation

First, install the OpenVPN package suitable for your Linux distribution. A detailed HOWTO is available on the [official OpenVPN website](http.openvpn.net/index.php/open-source/documentation/howto.html#quick).

For generating certificates and keys, we will use `easy-rsa`. On modern systems, it is recommended to install it as a separate package:
```bash
sudo apt-get update
sudo apt-get install easy-rsa
```

Next, copy the `easy-rsa` directory to a working directory to avoid altering the original files.
```bash
cp -r /usr/share/easy-rsa/ ~/openvpn-ca
cd ~/openvpn-ca
```
> **Note:** The following commands are based on `easy-rsa` version 3. If you are using an older version, the commands may differ (`build-ca`, `build-key-server`, etc.).

## Generating Certificates and Keys

### Initializing the Certificate Authority (CA)

Create a `vars` file in the `~/openvpn-ca` root to define the variables for your certificates.
```
set_var EASYRSA_REQ_COUNTRY    "US"
set_var EASYRSA_REQ_PROVINCE   "California"
set_var EASYRSA_REQ_CITY       "YourCity"
set_var EASYRSA_REQ_ORG        "YourOrganization"
set_var EASYRSA_REQ_EMAIL      "contact@example.com"
set_var EASYRSA_REQ_OU         "IT"
```

Initialize the new Public Key Infrastructure (PKI):
```bash
./easyrsa init-pki
```

Build the Certificate Authority (CA) certificate. Choose a Common Name, for example, your domain name.
```bash
./easyrsa build-ca
```

### Server Certificate and Key

Generate the certificate and private key for the OpenVPN server.
```bash
./easyrsa build-server-full server nopass
```
The Common Name will be "server." The `nopass` option creates a private key that is not password-protected.

### Client Certificate and Key

Generate a certificate and key for each client that will connect to the VPN.
```bash
./easyrsa build-client-full client_name nopass
```
Replace `client_name` with a unique name for each client. If you want to protect the key with a password, omit the `nopass` option.

### Diffie-Hellman Parameters

Generate the Diffie-Hellman parameters. Use a length of 2048 bits for adequate security.
```bash
./easyrsa gen-dh
```

### `tls-auth` Key for Added Security

To enhance security, generate a `tls-auth` key, which will help protect the server from DoS attacks.
```bash
openvpn --genkey --secret pki/ta.key
```

## OpenVPN Server Configuration

Copy the generated files to the OpenVPN configuration directory.
```bash
sudo cp pki/ca.crt pki/issued/server.crt pki/private/server.key pki/dh.pem pki/ta.key /etc/openvpn/server/
```

Create the configuration file `/etc/openvpn/server/server.conf`. You can use the examples provided with OpenVPN as a starting point.

Here is a basic configuration:
```ini
port 1194
proto udp
dev tun

ca ca.crt
cert server.crt
key server.key
dh dh.pem

server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt

# Push routes to allow clients to access the local network
push "route 192.168.0.0 255.255.255.0"

# Redirect all client traffic through the VPN
push "redirect-gateway def1 bypass-dhcp"

# Push DNS servers to clients
push "dhcp-option DNS 10.8.0.1" # Internal DNS
push "dhcp-option DNS 8.8.8.8"   # Public DNS as a fallback

client-to-client
keepalive 10 120

# Security
tls-auth ta.key 0 # Server-side
cipher AES-256-GCM
auth SHA256

user nobody
group nogroup
persist-key
persist-tun
status /var/log/openvpn/openvpn-status.log
verb 3
```

### Enabling IP Forwarding

For the server to route packets from VPN clients to the Internet, enable IP forwarding.
For temporary activation:
```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```
To make the change permanent, edit `/etc/sysctl.conf` and uncomment or add the line:
```
net.ipv4.ip_forward=1
```
Apply the change with `sudo sysctl -p`.

### Firewall Configuration (NAT)

Add an `iptables` rule so that outgoing traffic from the VPN is correctly routed.
```bash
sudo iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
```
Replace `eth0` with your primary network interface.

To make this rule persistent, install `iptables-persistent`:
```bash
sudo apt-get install iptables-persistent
sudo netfilter-persistent save
```

Then, start the OpenVPN server:
```bash
sudo systemctl start openvpn-server@server
```

## Client Configuration

On the client machine, install an OpenVPN client (like OpenVPN Connect). Create a `client.ovpn` configuration file with the following content:
```ini
client
dev tun
proto udp

remote your_server_ip 1194 # Replace with your server's IP or domain

resolv-retry infinite
nobind
persist-key
persist-tun

# Security
remote-cert-tls server
cipher AES-256-GCM
auth SHA256
key-direction 1 # Must be present with tls-auth

# Key files (place in the same directory as the .ovpn)
# or use the <ca>...</ca> syntax to include them in the file
ca ca.crt
cert client_name.crt
key client_name.key
tls-auth ta.key 1
```
You will need to copy `ca.crt`, `client_name.crt`, `client_name.key`, and `ta.key` to the client machine.

## File Sharing with Samba (via WINS)

> **Note:** WINS is an older name resolution technology, mainly for older versions of Windows. In modern networks, it is preferable to rely on DNS. This section is kept for informational purposes.

To allow VPN clients to discover Samba shares on the local network, you can use a WINS server.

On your main Samba server (which will also be the WINS server), modify `/etc/samba/smb.conf`:
```ini
[global]
    # ... other settings
    wins support = yes
    name resolve order = wins lmhosts hosts bcast
```

On other Samba servers on the network, declare the WINS server:
```ini
[global]
    # ... other settings
    wins server = 192.168.0.10 # IP of the WINS server
```

Finally, push the WINS server configuration to OpenVPN clients by adding this line to your `server.conf`:
```ini
push "dhcp-option WINS 10.8.0.1"
```
Restart the OpenVPN and Samba services to apply the changes.
