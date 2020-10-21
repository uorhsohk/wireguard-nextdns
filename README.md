# wireguard-nextdns

## Installing Wireguard on Ubuntu 20.04

```
sudo apt install wireguard
```

navigate to wireguard directory

```
sudo cd /etc/wireguard
```

create public and private key

```
umask 077
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
Address = 10.200.200.0/24
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

now, sample client configuration

```
[Interface]
PrivateKey = [INSERT_PEER_PRIVATE_KEY]
Address = 192.168.11.2/24
DNS = 192.168.11.1

[Peer]
PublicKey = [INSERT_SERVER_PUBLIC_KEY]
PresharedKey = [INSERT_PEER_ONE_PSK]
AllowedIPs = 0.0.0.0/0
Endpoint = [SERVER_PUBLIC_IP]:[SERVER_LISTEN_PORT]
```

## nextDNS Configuration

first create an account (it's free for 300,000 Requests, which should be enough for normal use)

https://my.nextdns.io/

and then you will be given an ID.

to setup nextDNS on Ubuntu run the following command

```
sh -c 'sh -c "$(curl -sL https://nextdns.io/install)"'
```

now, you will be asked some questions, answer them carefully

```
USER@SERVER_NAME:~# sh -c 'sh -c "$(curl -sL https://nextdns.io/install)"'
INFO: OS: ubuntu
INFO: GOARCH: [...]
INFO: GOOS: linux
INFO: NEXTDNS_BIN: [...]
INFO: LATEST_RELEASE: [...]
i) Install NextDNS
e) Exit
Choice (default=i):
```
enter (i)

after the installation is done, the following will be shown

```
NextDNS installed and started using systemd init
NextDNS Configuration ID:
```

here you have to enter your nextDNS Config-ID, this can be found on https://my.nextdns.io/, just head there and copy and paste the ID here.

```
Report device name? [Y|n]: [ANSWER_WHAT_EVER_YOU_WANT]
Setup as a router? (y/n): y
Enable caching? [Y|n]: [ANSWER_WHAT_EVER_YOU_WANT]
Automatically setup local host DNS? [Y|n]: y
```

now nextDNS is setup and ready to be used as DNS-Forwarder.
To start nextdns run the following command

```
nextdns start
```

and check the logs using

```
nextdns log
```

### FirewallD Configuration

```
firewall-cmd --add-port=[LISTEN_PORT]/tcp
firewall-cmd --add-port=[LISTEN_PORT]/udp
firewall-cmd --add-masquerade
firewall-cmd --add-rich-rule='rule family="ipv4" source address="10.200.200.0/24" accept'
```

```
systemctl restart wg-quick.target
```
