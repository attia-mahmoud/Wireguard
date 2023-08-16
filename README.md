# Wireguard
Steps to create a Wireguard connection between two nodes

Useful to initiate a connection between two devices through their wireguard interfaces to bypass a firewall.

On both devices, create a private/public key pair named `privatekey` and `publickey`

```jsx
$ wg genkey | tee privatekey | wg pubkey > publickey
```

On the server, edit the config file at `/etc/wireguard/wg0.conf` to contain:

```jsx
/etc/wireguard/wg0.conf

[Interface]
PrivateKey=<SERVER'S PRIVATE KEY>
Address=<SERVER'S PRIVATE IP (EX: 10.0.0.1/8)>
SaveConfig=true
PostUp=iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE;
PostDown=iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE;
ListenPort=<SERVER'S PORT TO LISTEN ON (EX: 51820)>
```

On the client, edit the config at `/etc/wireguard/wg0.conf` to contain:

```jsx
/etc/wireguard/wg0.conf

[Interface]
PrivateKey=<CLIENT'S PRIVATE KEY>
Address=<CLIENT'S PRIVATE IP (EX: 10.0.0.2/8)>
SaveConfig=true
```

To add peers to each device, there are two ways:

First way is by editing the config file to contain:

```jsx
etc/wireguard/wg0.conf

[Peer]
PublicKey=<PEER'S PUBLIC KEY>
Endpoint=<PEER'S SOCKET ADDRESS (IP_ADDRESS:PORT)>
AllowedIPs=<IPS TO ACCEPT (0.0.0.0/0 TO ACCEPT ANY ADDRESS)>
#On The Client
PersistentKeepalive=30 
```

Second way is through the CLI:

```bash
$ wg set wg0 peer <PEERS PUBLIC KEY> endpoint <PEERS SOCKET ADDRESS (IP_ADDRESS:PORT)> \
allowed-ips <IPS TO ACCEPT (0.0.0.0/0 TO ACCEPT ANY ADDRESS)> persistent-keepalive 30 
```

Start up the wireguard interface on both devices by running:

```jsx
$ wg-quick up wg0
```
