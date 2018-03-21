# Netboot

Pro nastavení netbootu je potřeba minimální setup dvou VM, jedno bude Master, ze kterého se bude OS bootovat, druhý bude Slave, který bude bootovat po síti.

![](/assets/SUS - Page 1 %281%29.png)

Dále bude VM1 bude master, VM2 bude slave/client.

## Postup

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

Na VM1 nainstalujeme a nastavíme DHCP server, ten bude přidělovat adresu a další informace pro VM2. Nainstalujeme balík `isc-dhcp-server`

DHCP server je defaultně vypnutý a musí se tedy zapnout v _/etc/defaul/isc-dhcp-server_, upraví se pouze řádek s konfigurací pro IPv4, tak aby distribuoval DHCP do sítě s VM2 `INTERFACESv4="enp0s8"`

V _/etc/dhcp/dhcp.conf _nastavíme **domain-name**, **domain-name-servers **\(adresa DNS serveru na kt. se budou forwardovat dotazy - ve školní síti musí být nastaven školní DNS\) a DHCP zónu, nakonec restartujeme server `service isc-dhcp-server restart`:

```
option domain-name "vsb.cz";
option domain-name-servers 8.8.8.8; 

subnet 172.16.0.0 netmask 255.255.255.0 {
  range 172.16.0.10 172.16.0.20;  # rozsah pridelovanch adres
  option broadcast-address 172.16.0.255; 
  option routers 172.16.0.2;  # adresa GW - rozhrani aktualniho VM1
}
```

Na VM2 nastavíme na enp0s3 získávání adresu z DHCP \`

