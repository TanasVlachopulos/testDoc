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

DHCP server je defaultně vypnutý a musí se tedy zapnout v **/etc/defaul/isc-dhcp-server**, upraví se pouze řádek s konfigurací pro IPv4, tak aby distribuoval DHCP do sítě s VM2 `INTERFACESv4="enp0s8"`

V **/etc/dhcp/dhcp.conf** nastavíme **domain-name**, **domain-name-servers **\(adresa DNS serveru na kt. se budou forwardovat dotazy - ve školní síti musí být nastaven školní DNS\) a DHCP zónu, nakonec restartujeme server `service isc-dhcp-server restart`:

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

#### Netboot

Do složky **/srv/tftp** na VM1 rozbalíme soubory potřebné pro [netboot \[link\]](http://ftp.cz.debian.org/debian/dists/Debian9.4/main/installer-amd64/current/images/netboot/netboot.tar.gz).

```
wget http://ftp.cz.debian.org/debian/dists/Debian9.4/main/installer-amd64/current/images/netboot/netboot.tar.gz
tar -xf netboot.tar.gz
rm netboot.tar.gz
```

Z archivu se vybalí soubory/odkazy/složky: \_debina-installer, ldlinux.c32, pxelinux.0, pxelinux.cfg, version.info \_k těmto souborům musíme vytvořit ještě složku **Debian** a překopírovat několik souborů ze složky **debian-installer/amd64/boot-screens/ **do složky **/srv/tftp**:

```
root@sus:/srv/tftp$ cp debian-installer/amd64/boot-screens/libcom32.c32 .
root@sus:/srv/tftp$ cp debian-installer/amd64/boot-screens/libutil.c32 .
root@sus:/srv/tftp$ cp debian-installer/amd64/boot-screens/vesamenu.c32 .
```

Do složky **Debian** nakopírujeme aktuální kernel ze složky **/boot** z VM1:

```
cp /boot/vmlinuz-4.9.0-4-amd64 /srv/tftp/Debian/
cp /boot/initrd.img-4.9.0-4-amd64 /srv/tftp/Debian/
```

Uvnitř složky **Debian** vytvoříme další složku **Debian/root** a do ní nakopírujeme, nebo pouze vytvoříme složky z file systému VM1:

```
cp -r /bin .
cp -r /boot .
mkdir dev
cp -r /etc .
mkdir home
cp -r /lib .
cp -r /lib64 .
mkdir media
mkdir mnt
cp -r /opt .
mkdir proc
cp -r /root .
mkdir run
cp -r /sbin .
mkdir srv 
mkdir sys
mkdir tmp
cp -r /usr .
cp -r /var .
```

U **Debian/root/tmp** se musí ještě změnit oprávnění `chmod 777 tmp/`a `chmod o+t tmp/`

> Při chybě Kernel panic při bootu VM2 zkuntrolujte důkladně jestli jsou opravdu všechny složky nakopírované

Výsledná struktura vypadá cca takto:

```
.
├── Debian
│   ├── initrd.img-4.9.0-4-amd64
│   ├── root
│   │   ├── bin
│   │   ├── boot
│   │   ├── dev
│   │   ├── etc
│   │   ├── home
│   │   ├── lib
|   |   ├── lib64
│   │   ├── media
│   │   ├── mnt
│   │   ├── opt
│   │   ├── root
│   │   ├── run
│   │   ├── sbin
│   │   ├── srv
│   │   ├── sys
│   │   ├── tmp
│   │   └── usr
│   └── vmlinuz-4.9.0-4-amd64
├── debian-installer
│   └── amd64
│       ├── bootnetx64.efi
│       ├── boot-screens
│       ├── grub
│       ├── initrd.gz
│       ├── linux
│       ├── pxelinux.0
│       └── pxelinux.cfg
├── ldlinux.c32 -> debian-installer/amd64/boot-screens/ldlinux.c32
├── libcom32.c32
├── libutil.c32
├── pxelinux.0 -> debian-installer/amd64/pxelinux.0
├── pxelinux.cfg -> debian-installer/amd64/pxelinux.cfg
├── version.info
└── vesamenu.c32
```

Ve zkopírovaném FS musíme ještě provést pár změn v **Debian/root/etc/fstab** zrušíme mount všech disků, v **Debian/root/etc/network/interfaces **zrušíme nastavení všech síťovek kromě LO, v **Debian/root/etc/dhcp/dhcpd.conf **zrušíme DHCP nastavení.

V **pxelinux.cfg/default** je nutné zmodifikovat config \(stávající obsah je vhodné zakomentovat\):

```
DEFAULT vesamenu.c32
PROMPT 0

MENU TITLE  Boot Menu

# name of option in boot menu
LABEL Debian - Custom SUS NetBoot

KERNEL /Debian/vmlinuz-4.9.0-4-amd64
APPEND initrd=/Debian/initrd.img-4.9.0-4-amd64 root=/dev/nfs nfsroot=172.16.0.2:/srv/tftp/Debian/root,udp ip=dhcp rw
```

Důležité je aby seděli cesty a bootovacím obrazům vmlinuz a initrd.img a taky musí být správně vyplnéná položka nfsroot, která musí obsahovat celou cestu k NTP úložišti \(172.16.0.2:/srv/tftp/Debian/root\). Důležité je taky aby se použil protokol **udp** místo defaultního tcp, jinak obraz nenabootoval, protože při bootvání dojde několikrát ke změně adresy a ztrátě spojení, TCP spojení pak vytimeoutuje, proto se musí použít UDP, který není vázán na session.

Upravit musíme ještě exporty disků v NTP, do konfigu **/etc/exports** přibude ještě jeden řádek:

```
/srv/tftp/Debian/root   172.16.0.*(rw,sync,no_root_squash)
```

Option _no\_root\_squash_ je důležitá, kdyby tady nebyla nemohl by root do připojeného FS zapisovat.

Do konfigurace DHCP **/etc/dhcp/dhcp.conf** serveru ještě přidáme další 2 řádky, které řeknou VM2 že má bootovat ze sítě a kde najde boot menu.

```
subnet 172.16.0.0 netmask 255.255.255.0 {
  range 172.16.0.10 172.16.0.20;
  option broadcast-address 172.16.0.255;
  option routers 172.16.0.2;
  next-server 172.16.0.2;
  filename "pxelinux.0";
}
```

Restartujeme `isc-dhcp-server`restartujeme `nfs-kernel-server`

Ve VirtualBoxu u VM2 nastavíme jako primární a jedinou bootovací možnost síť. Restartujeme VM2.

