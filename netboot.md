# Netboot

&gt; dpkg -L package vylistuje soubory balíku



**návod:**

vytvořit 2x virtuální počítač, na jedno nechat síťovou vou kartu s natem a síť s hostem \(bez dhcp serveru\), na druhém počítači připojit jenom síť s hostem \(stejnou jako u VM 1\)

na VM 1 vytvořit DHCP server, nainstalovat `isc-dhcp-server`

musíme dhcp server zapnout v /etc/default

v /etc/dhcp/dhcpd.conf je konfigurace dhcp, je nutné nastavit option domain name a dostupný DNS server

můžeme zapnout i autorizovaný režim

v souboru nastavíme jeden subnet, ze kterého se pro druhé VM budou přiřazovat adresy, default GW \(option routers\) necháme na dresu  VM 1

restartujeme `service isc-dhcp-server restart`

> autoritativní DNS server - server je schopen přiřazovat IP adresy i v sítích ve kerých neparticipuje
>
>  NFS - network file system - mapuje vzdálený disk na lokální disk



Na VM 1 nainstalujeme `nfs-kernel-server`



NFS distribuuje soubory tak jak je na síťovém disku disku najde, tedy včetně user ID a group ID, což namená, že na serveru, který síťový disk namountuje musí mít uživatele se stejným ID a group ID, pokud se používí LDAP není to třeba moc řešit, ale pokud jsou uživatelé lokální je nutné ověřit  jestli tam stejný user existuje



je nutné modifikovat \`/etc/exports\` kde se nastaví co se exportuje a kam se to exportuje



```
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



\`\`\`

connect IP\_serveru

binary

get testovacisoubor.txt

\`\`\`







