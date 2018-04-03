# DNS

#### Prerekvizity

Připravíme si 2 servery master a slave, oba s 2 síťovkami. První síťiovka bude mít síť s NATem a internetem, druhá bude síť pouze s hostem. Na rozhranní pouze s hostem bude lepší nakonfigurovat statickou IP adresu, v tomto případě budou použity adresy 172.16.0.2 a 172.16.0.3.

#### Postup

Na mastru i slavu nainstalujeme populární DNS server `bind9` a několik utilit, které nám pomohou při konfiguraci `dnsutils` a `udns-utils`.

Na mastru v konfiguraci **/etc/bind/named.conf.options **odkomentujeme sekci **forwarders** a nastavíme nějaký vhodný DNS server na kt. se budou přeposílat DNS dotazy. Ve školní síti to musí být **158.196.0.53**, mimo školu to může být třeba 8.8.8.8, nebo 1.1.1.1.

Na mastru v **/etc/bind** vytvoříme nový zónový soubor začínající na db.\* \(je zvykem ho mít pojmenovaný podle domény\) například **db.tanas.local**. Šablonou pro obsah souborů může být například snippet z [webu SUS](http://seidl.cs.vsb.cz/wiki/index.php/SUS#.C5.A0est.C3.A1_p.C5.99edn.C3.A1.C5.A1ka).

```
$TTL 1h                    ;doba expirace všech záznamů
@       IN      SOA     ns1.tanas.local. abuse.tanas.local. (  ; autoritativní DNS server + email správce bez @
                         2018040301 ; seriové číslo, často ve formě data
                         4h         ; jak často si stahuje data sekundární server
                         2h         ; za jak dlouho se má sek.server pokusit stáhnout data při neúspěchu
                         2w         ; kdy platnost dat v sek.serveru vyprší
                         1h )       ; jak dlouho si mají data pamatovat cache servery
;
@       IN      NS      ns1.tanas.local. ; autoritativní servery pro doménu, musí mít i A záznamy
@       IN      NS      ns2.tanas.local. ; autoritativní servery pro doménu, musí mít i A záznamy


tanas.local.   IN      MX      10      smtp.tanas.local.  ; primary MX record for SMTP server on master
tanas.local.   IN      MX      20      smtp2.tanas.local. ; secundary MX record for SMTP server on slave
tanas.local.   IN      A               172.16.0.2         ; primary A record for domain tanas.local
ns1            IN      A               172.16.0.2
ns2            IN      A               172.16.0.3         ; record for secondary name server on slave VM
smtp           IN      A               172.16.0.2
smtp2          IN      A               172.16.0.3
www            IN      CNAME           tanas.local.       ; aliases for A record
wiki           IN      CNAME           tanas.lcoal.
```

Správnost kofigurace můžeme ověřit pomocí `named-checkzone db.tanas.local /etc/bind/db.tanas.local` výstupem by mělo být OK.

V souboru **/etc/bind/named.conf.local** ještě musíme přidat distribuci zóny:

```
zone "tanas.local" {
       type master;                      // this server is master for this domain
       file "/etc/bind/db.tanas.local";  // path to zone configuration
};
```

restartujeme `service bind9 restart` a zkontrolujeme stav `service bind9 status`

Funkčnost můžeme ověřit vyhledáním v DNS pomocí `nslookup tanas.local localhost` výsledek by měla být adresa 172.16.0.2, kterou jsme nastavili.

