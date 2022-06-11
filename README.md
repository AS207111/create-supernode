You could find an ansible script in this repo to install a supernode for the freifunk hood as207111 from scratch.

You just need to install a simple debian buster virtual or physical maschine and ansible.

After installation just login with your normal user and clone the whole repo in your home directory.

**You must change the parameters in the playbook.yml of course!**
The parameters should be explained or self explaining.

Then you could start the installation with a command like `ansible-playbook  playbook.yml -b --ask-become-pass`

## Copy playbook

Create a new `fastd_privatekey` key:

> Use the private key in your playbook, the public key is for the gluon-site.

```bash
fastd --generate-key
```

For `wireguard_privatekey_nodes` and `wireguard_privatekey_azirevpn` you can use the shared secrets.


## Supernodes

| DNS-Name                 | IP-Adresse     | IPv6-Adresse          |
|--------------------------|----------------|-----------------------|
| core-fsn1-1.as207111.net | 78.47.117.100  | 2a01:4f8:c17:5519::1  | 
| core-fsn1-2.as207111.net | 167.235.225.24 | 2a01:4f8:c012:e949::1 | 

## IP-Bereiche

Als IPv6-Bereich wird `fd01:67c:770::/64` verwendet. In diesem Bereich werden IPv6-Bereiche mit einer 68 Bit Maske verwendet. Diese teilen sich wie folgt auf:

| IP-Range               | Verwendungszweck                       |
|------------------------|----------------------------------------|
| fd01:67c:770::/68      | reserviert für feste Dienste/Maschinen |
| fd01:67c:770:1000::/68 | DHCP-Bereich Supernode 1               |
| fd01:67c:770:2000::/68 | DHCP-Bereich Supernode 2               |

Als Public IPv6-Bereich wird `2a0e:b107:1901::/48` verwendet. In diesem Bereich werden IPv6-Bereiche mit einer 68 Bit Maske verwendet. Diese teilen sich wie folgt auf:

| IP-Range               | Verwendungszweck                       |
|------------------------|----------------------------------------|
| fd01:67c:770::/68      | reserviert für feste Dienste/Maschinen |
| 2a0e:b107:1901:1000::/68 | DHCP-Bereich Supernode 1               |
| 2a0e:b107:1901:1000::/68 | DHCP-Bereich Supernode 2               |

Als IPv4-Bereich wird `10.207.0.0/16` verwendet. Dann verwenden wir 21er Bereiche für die einzelnen Bereiche:

| IP-Range       | Verwendungszweck             |
|----------------|------------------------------|
| 10.207.0.0/21  | Für interne Dienste          |
| 10.207.8.0/21  | DHCP-Bereich für Supernode 1 |
| 10.207.16.0/21 | DHCP-Bereich für Supernode 2 |
| 10.207.24.0/21 | Frei |


## Speedtest

```bash
wget --bind-address=10.207.16.1 https://proof.ovh.net/files/10Gb.dat -O /dev/null
```

## Test Supernode

```bash
uci export fastd.mesh_vpn_backbone_peer_core_fsn1_1

config peer 'mesh_vpn_backbone_peer_core_fsn1_1'
	option enabled '1'
	option key '4097475a3b2bc346a8f6e682c3b8df043a5a97b79fe83ae8d9761176cfefe050'
	option net 'mesh_vpn'
	option group 'mesh_vpn_backbone'
	list remote 'ipv4 "core-fsn1-1.as207111.net" port 10040'
	list remote '78.47.117.100:10040'
	option interface 'mesh-vpn'
```

### Test Supernode 1:

```
uci delete fastd.mesh_vpn_backbone_peer_core_fsn1_1.remote
uci add_list fastd.mesh_vpn_backbone_peer_core_fsn1_1.remote="78.47.117.100:10040:10040"
uci set fastd.mesh_vpn_backbone_peer_core_fsn1_1.key="4097475a3b2bc346a8f6e682c3b8df043a5a97b79fe83ae8d9761176cfefe050"
uci commit
/etc/init.d/fastd restart
logread -f
```

### Test Supernode 2:

```
uci delete fastd.mesh_vpn_backbone_peer_core_fsn1_1.remote
uci add_list fastd.mesh_vpn_backbone_peer_core_fsn1_1.remote="167.235.225.24:10040"
uci set fastd.mesh_vpn_backbone_peer_core_fsn1_1.key="7da5e35bc135e1ac6d37a7c3994fedd10f52093170a09b790fcecba34e789c32"
uci commit
/etc/init.d/fastd restart
logread -f
```