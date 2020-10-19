# wireguard-nextdns

## Installing Wireguard on Ubuntu 20.04

```
sudo apt install wireguard
```

navigate to wireguard directory

```
cd /etc/wireguard
```

create public and private key

```
wg genkey | tee privatekey | wg pubkey > publickey
```

generate PSK for first peer

```
wg genpsk > peer-one-psk
```

generate PSK for second peer

```
wg genpsk > peer-two-psk
```

create wireguard configuration

```
vim /etc/wireguard/wg0.conf
```

and paste the following (this config is for two clients)

```
[Interface]
Address = 192.168.11.1/24
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
ListenPort = [LISTEN_PORT]
PrivateKey = [INSERT_HERE_THE_PRIVATE_KEY_YOU_GENERATED_ABOVE]

[Peer]
PublicKey = [PEER_ONE_PUBLIC_KEY]
PresharedKey = [PEER_ONE_PSK]
AllowedIPs = 192.168.11.2/32


[Peer]
PublicKey = [PEER_TWO_PUBLIC_KEY]
PresharedKey = [PEER_TWO_PSK]
AllowedIPs = 192.168.11.3/32
```
