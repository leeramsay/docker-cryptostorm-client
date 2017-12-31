# Cryptostorm OpenVPN client (unofficial)
*This is not an official Cryptostorm client, it just supports Cryptostorm.*

## What does this do?
This alpine-based container contains the [Cryptostorm .ovpn configuration files](https://github.com/cryptostorm/cryptostorm_client_configuration_files). Give it a username and (optionally) specify which config file to use, and it will connect to Cryptostorm. You can then connect other containers to it via `--net=container:vpn_container_name` or through [docker-compose](https://docs.docker.com/compose/compose-file/#network_mode).

## Usage
`docker-compose` is recommended (for ease of use and clarity) over `docker run` but examples of both are provided.

### Sample `docker-compose.yml`
#### Cryptostorm (paid service)
```yaml
version: '3'

services:
  vpn:
    build: .
    environment:
      CRYPTOSTORM_USERNAME: your_long_sha512
      CRYPTOSTORM_CONFIG_FILE: cstorm_linux-balancer_udp.ovpn
    cap_add:
      - NET_ADMIN
    dns:
      - 5.101.137.251
      - 46.165.222.246
```

#### Cryptofree (free service)
```yaml
version: '3'

services:
  vpn:
    build: .
    environment:
      CRYPTOSTORM_CONFIG_FILE: cryptofree_linux-udp.ovpn
    cap_add:
      - NET_ADMIN
    dns:
      - 5.101.137.251
      - 46.165.222.246
```

### Sample `docker run`

## Details
### Firewall
The container has a built in `iptables` firewall based off [this gist](https://gist.github.com/superjamie/ac55b6d2c080582a3e64). It blocks all non-vpn traffic on `eth0` **except OpenVPN, DNS, ICMP and local (LAN) traffic**. This should prevent any external communication except to establish a VPN connection. It also means that you can still access attached containers' services from within the network (e.g., if you're running Deluge you can still connect to the web interface). DNS *should* be forwarded over the VPN once it's up.

### NET_ADMIN required
Running VPN clients in Docker **requires NET_ADMIN**. So that means you need to add `--cap-add NET_ADMIN` if running through `docker run` or use the [relevant docker-compose method](https://docs.docker.com/compose/compose-file/#cap_add-cap_drop). **If you don't do this, the container won't be able to connect and will exit**.

### Authentication and Cryptofree
Cryptostorm uses a SHA512-based authentication system ([more on their website](https://cryptostorm.is)). The SHA512 hash of your token is used as the username, and the password is ignored. This means that you need to either:

- Provide a valid SHA512 through `--env CRYPSTOSTORM_USERNAME=myhash` or [docker-compose](https://docs.docker.com/compose/compose-file/#environment) **and** set `CRYPTOSTORM_CONFIG_FILE` to a choice from [this list](https://github.com/cryptostorm/cryptostorm_client_configuration_files/tree/master/linux).

or:

- Use [Cryptofree](https://github.com/cryptostorm/cryptostorm_client_configuration_files/tree/master/cryptofree), Cryptostorm's free service capped at 160kb/s down, 130kb/s up. This is **the default**, but can be selected by setting `CRYPTOSTORM_CONFIG_FILE` to `cryptofree_linux-udp.ovpn` or `cryptofree_linux-tcp.ovpn`.

