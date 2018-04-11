# Netboot \(draft\)

&gt; dpkg -L package vylistuje soubory balíku

**návod:**

vytvořit 2x virtuální počítač, na jedno nechat síťovou vou kartu s natem a síť s hostem \(bez dhcp serveru\), na druhém počítači připojit jenom síť s hostem \(stejnou jako u VM 1\)

na VM 1 vytvořit DHCP server, nainstalovat `isc-dhcp-server`

musíme dhcp server zapnout v /etc/default

v /etc/dhcp/dhcpd.conf je konfigurace dhcp, je nutné nastavit option domain name a dostupný DNS server

můžeme zapnout i autorizovaný režim

v souboru nastavíme jeden subnet, ze kterého se pro druhé VM budou přiřazovat adresy, default GW \(option routers\) necháme na dresu VM 1

restartujeme `service isc-dhcp-server restart`

> autoritativní DNS server - server je schopen přiřazovat IP adresy i v sítích ve kerých neparticipuje
>
> NFS - network file system - mapuje vzdálený disk na lokální disk

Na VM 1 nainstalujeme `nfs-kernel-server`

NFS distribuuje soubory tak jak je na síťovém disku disku najde, tedy včetně user ID a group ID, což namená, že na serveru, který síťový disk namountuje musí mít uživatele se stejným ID a group ID, pokud se používí LDAP není to třeba moc řešit, ale pokud jsou uživatelé lokální je nutné ověřit jestli tam stejný user existuje

je nutné modifikovat \`/etc/exports\` kde se nastaví co se exportuje a kam se to exportuje

```text
/home    192.168.20.*(rw,sync,no_subtree_check)
```

\`exportfs\` ukáže jaké složky jsou kam exportovány

je nutné nastavit na VM1 NAT v iptables a zapnout interní forwarding

na VM2 nainstalujeme klienta NFS \`apt install nfs-common\`

na VM2 namountujeme disk

\`\`\`

mount 192.168.1.23:/home /home

\`\`\`

Defaultně se provádí export tak že do nfs nemůže zapisovat root

pro zavedení systému přes síť je nutné zprovoznit TFTP server, který doručí kernel na bootovanou stanici

na VM1 nainstalovat \`tftpd-hpa\` v /etc/default/tftpd-hpa je konfig

do /srv/tftp jsou mapovány soubory kt. se distribuují přes tfpt, z testovacích důvodů tam plácneme nějaký soubor

na VM2 nainstalujeme TFTP klienta \`tftp-hpa\` abychom otestovali funkčnost TFPT serveru

```text
connect IP_serveru
binary
get testovacisoubor.txt
```

Do složky distribuované tftp serverem nahrajeme sobory nutné pro netboot

v konfiguraci DHCP serveru musíme ještě přidat informace o tom že má bootovat ze sítě, odkud to má stahovat, a jak\`\`

```text
subnet 192.168.57.0 netmask 255.255.255.0 {
range 192.168.57.100 192.168.57.200;
option broadcast-address 192.168.57.255;
option routers 192.168.57.2;
next-server 192.168.57.2;
filename "pxelinux.0";
}
```

v nastavení virtualboxu je ještě nutné zapnout bootování ze sítě

v složce sftp musíme mít tyto soubory pro boot, najdedeme je v připraveném balíku, ale jsou tam všelijak porozházené

```text
Debian
ldlinux.c32
libcom32.c32
libutil.c32
pxelinux.cfg
pxelinux.0
vesamenu.c32
```

v /srv/tftp/pxelinux.cfg se nastaví defaultní bootovací konfig

```text
DEFAULT vesamenu.c32
PROMPT 0

MENU TITLE  Boot Menu

LABEL Debian - NetBoot
KERNEL /Debian/vmlinuz-3.16.0-4-686-pae
APPEND initrd=/Debian/initrd.img-3.16.0-4-686-pae root=/dev/nfs nfsroot=192.168.57.2:/tftpboot/Debian/root,udp ip=dhcp rw
```

Položka kernel odkazuje na cestu na TFTP serveru \(ne lokální složka\) složku debian tam budeme muset vytvořit. V položce APPENDje cesta nfsroot, ta odkazyje na složky v adresáři tftp serveru, z kt. se stane na klientovi rootovský file system.

u nfsroot je položka `,udp` toto je nesmírně důležité protože kdyby to tam nebylo snažil by se klient při bootu navazovat spojení s NFS server přes TCP, nicméně při bootvání dojde několikrát ke změně adresy a zhození spojení, TCP spojení pak vytimeoutuje na, proto se musí použít UDP, který není vázán na session

Tento rootovský file system msuíme v sftp složce vytvořit, ty vytvoříme na základně aktuální struktury serveru, musíme kopírovat, nebo vytvořit následující složky, některé složky se musí překopírovat, některé není nutné protože jsou mapovány do paměti za běhu

```text
bin -> cp
boot -> cp
dev
etc -> cp
home
lib -> cp
media
mnt
opt -> cp
proc
root -> cp
run
sbin -> cp
srv -> cp
sys
tmp
usr -> cp
var -> cp
```

u /tmp se musí ještě změnit oprávnění `chmod 777 tmp/` a `chmod o+t tmp/`

v nfs configu musíme nastavit export rootovského FS, s tím že povolíme no root squash, díky čemuž bude moci root editovat soubory `/tftpboot/Debian/root 192.168.57.0/24(rw,async,no_root_squash)`

v distribouvaném root FS, musíme upravit některé soubory, které nechceme disribuovat na klienty, to je napíklad nastavení fstab a taky musíme smazat nastavení obou síťových karet, kt. klient nemá

