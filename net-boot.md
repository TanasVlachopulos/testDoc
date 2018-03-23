# Netboot

Pro nastavení netbootu je potřeba minimální setup dvou VM, jedno bude Master, ze kterého se bude OS bootovat, druhý bude Slave, který bude bootovat po síti.

![](/assets/SUS - Page 1 %281%29.png)

Dále bude VM1 bude master, VM2 bude slave/client.

## Postup

#### Síťové karty

Vytvoříme ve VirtualBoxu novou síť pouze s hostem se sítí 172.16.0.0/24 a **vypneme v ní DHCP**.

Na VM1 nastavíme na jednom rozhraní síť s natem a na druhém staticky adresu ze sítě 172.16.0.0/24.

```
allow-hotplug enp0s8
auto enp0s8
iface enp0s8 inet static
   address 172.16.0.2
   netmask 255.255.255.0
   broadcast 172.16.0.255
```

Restartujeme síť `service networking restart` a znovu nahodíme adresu na NAT interface \(zmizela při restartu\) `dhclient enp0s3`

#### DHCP

Na VM1 nainstalujeme a nastavíme DHCP server, ten bude přidělovat adresu a další informace pro VM2. Nainstalujeme balík `isc-dhcp-server`

DHCP server je defaultně vypnutý a musí se tedy zapnout v _/etc/defaul/isc-dhcp-server_, upraví se pouze řádek s konfigurací pro IPv4, tak aby distribuoval DHCP do sítě s VM2 `INTERFACESv4="enp0s8"`

V \_/etc/dhcp/dhcp.conf \_nastavíme **domain-name**, **domain-name-servers **\(adresa DNS serveru na kt. se budou forwardovat dotazy - ve školní síti musí být nastaven školní DNS\) a DHCP zónu, nakonec restartujeme server `service isc-dhcp-server restart`:

```
option domain-name "vsb.cz";
option domain-name-servers 8.8.8.8; 

subnet 172.16.0.0 netmask 255.255.255.0 {
  range 172.16.0.10 172.16.0.20;  # rozsah pridelovanch adres
  option broadcast-address 172.16.0.255; 
  option routers 172.16.0.2;  # adresa GW - rozhrani aktualniho VM1
}
```

Na VM2 nastavíme na enp0s8 \(nebo enp0s3, podle toho kt. karta je zapojená\) získávání adresu z DHCP:

```
allow-hotplug enp0s8
auto enp0s8
iface enp0s8 inet dhcp
```

#### NAT

Aby VM2 mohlo do internetu je nutné na VM1 nastavit NAT mezi síťovkami. Nejdříve je nutné na VM1 povolit směrování mezi rozhraními, následně se zapne NAT \(parametr -o je výstupní rozhraní s do internetu, v tomto případě je to síťovka ze sítě NAT\):

```
 echo 1 > /proc/sys/net/ipv4/ip_forward
 iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
 iptables -t NAT -L  # list all nat IP rules
```

NAT není perzistentní, aby se zachoval i po restartu můžeme udělat následující:

```
apt install iptables-persistent
iptables-save >/etc/iptables/rules.v4  # save actual iptables settings
```

#### NFS

Network file system - mapuje vzdálený síťový disk na počítač jako lokální disk.

na VM1 nainstalujeme `nfs-kernel-server` a do **/etc/exports** vložíme nastavení pro export lokálních složek jako vzdálené úložiště a nakonec restartujeme servisu:

```
/home   172.16.0.*(rw,sync,no_subtree_check)
```

Seznam exportů můžeme vylistovat pomocí příkazu `exportfs`.

Na VM2 nainstalujeme `nfs-common` a otestujeme připojení adresáře home z VM1.

#### TFTP

TFTP se používá pro zavedení kernelu na druhém VM při startu. Na VM1 nainstalujeme `tftpd-hpad`v **/etc/default/tftpd-hpa **je config, ale není jej nutné nijak upravovat. 

Do **/srv/tftp** jsou mapovány soubory dostupné skrze TFTP. Pro testovací účely tam vložíme nějaký soubor.

Na VM2 nainstalujeme TFTP klient `tftp-hpa` a zkusíme stáhnout testovací soubor pomocí následujících příkazů:

```
root@sus:~$ tftp
(to) 172.16.0.2
tftp> binary
tftp> get testfile.txt
tftp> q 
```

Do složky **/srv/tftp** na VM1 rozbalíme soubory potřebné pro [netboot \[link\]](http://ftp.cz.debian.org/debian/dists/Debian9.4/main/installer-amd64/current/images/netboot/netboot.tar.gz).

```
wget http://ftp.cz.debian.org/debian/dists/Debian9.4/main/installer-amd64/current/images/netboot/netboot.tar.gz
tar -xf netboot.tar.gz
rm netboot.tar.gz
```





