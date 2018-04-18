# DNS & Apache

## DNS

### Prerekvizity

Připravíme si 2 servery master a slave, oba s 2 síťovkami. První síťiovka bude mít síť s NATem a internetem, druhá bude síť pouze s hostem. Na rozhranní pouze s hostem bude lepší nakonfigurovat statickou IP adresu, v tomto případě budou použity adresy 172.16.0.2 a 172.16.0.3.

```text
allow-hotplug enp0s8
auto enp0s8
iface enp0s8 inet static
   address 172.16.0.2
   netmask 255.255.255.0
   broadcast 172.16.0.255
```

### Postup

Na mastru i slavu nainstalujeme populární DNS server `bind9` a několik utilit, které nám pomohou při konfiguraci `dnsutils` a `udns-utils`.

Na mastru v konfiguraci **/etc/bind/named.conf.options **odkomentujeme sekci **forwarders** a nastavíme nějaký vhodný DNS server na kt. se budou přeposílat DNS dotazy. Ve školní síti to musí být **158.196.0.53**, mimo školu to může být třeba 8.8.8.8, nebo 1.1.1.1.

Na mastru v **/etc/bind** vytvoříme nový zónový soubor začínající na db.\* \(je zvykem ho mít pojmenovaný podle domény\) například **db.tanas.local**. Šablonou pro obsah souborů může být například snippet z [webu SUS](http://seidl.cs.vsb.cz/wiki/index.php/SUS#.C5.A0est.C3.A1_p.C5.99edn.C3.A1.C5.A1ka).

```text
$TTL 1h                    ; record expiration
@       IN      SOA     ns1.tanas.local. abuse.tanas.local. (  ; athoritative DNS server with admin email withnout  @
                         2018040301 ; serial number
                         4h         ; slave server refresh periode
                         2h         ; failure refresh interval
                         2w         ; expiration
                         1h )       ; persistency period
;
@       IN      NS      ns1.tanas.local. ; authoritative server for domain, aditional A record required
@       IN      NS      ns2.tanas.local. ; authoritative server for domain, aditional A record required


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

V souboru **/etc/bind/named.conf.local** ještě musíme přidat distribuci zóny a nastavit master server jako mastra:

```text
acl "tanas.local" {
       172.16.0.3;                        // allow acces for secondary slave server
};

zone "tanas.local" {
       type master;                       // this server is master for this domain
       file "/etc/bind/db.tanas.local";   // path to zone configuration
       allow-transfer { "tanas.local"; }; // allow transfer to slave
};
```

!!! Pozor na středníky v konfiguraci.

Restartujeme `service bind9 restart` a zkontrolujeme stav `service bind9 status`

Funkčnost můžeme ověřit vyhledáním v DNS pomocí `nslookup tanas.local localhost` výsledek by měla být adresa 172.16.0.2, kterou jsme nastavili.

Na slave serveru pouze upravíme **/etc/bind/named.conf.local** přidáme do něj záznam o tom že aktuální server je slave a kdo mu dělá mastera:

```text
masters tanas.local-master { 172.16.0.2; };

zone tanas.local {
        type slave;
        file "/var/cache/bind/db.tanas.local";
        masters { tanas.local-master; };
};
```

Restartujeme server a zkontrolujeme pomocí status, jestli se provedla replikace, ověříme funknost pomocí nslookup. Výstup `service bind9 status`:

```text
Apr 04 00:27:56 sus named[1111]: zone tanas.local/IN: Transfer started.
Apr 04 00:27:56 sus named[1111]: transfer of 'tanas.local/IN' from 172.16.0.2#53: connected using 172.16.0.3#52367
Apr 04 00:27:56 sus named[1111]: zone tanas.local/IN: transferred serial 2018040304
Apr 04 00:27:56 sus named[1111]: transfer of 'tanas.local/IN' from 172.16.0.2#53: Transfer status: success
Apr 04 00:27:56 sus named[1111]: transfer of 'tanas.local/IN' from 172.16.0.2#53: Transfer completed: 1 messages, 13
records, 314 bytes, 0.001 secs (314000 bytes/sec)
Apr 04 00:27:56 sus named[1111]: zone tanas.local/IN: sending notifies (serial 2018040304)
```

**!!!** Pokud změníme nějaká nastavené v zónových souborech na masteru, musíme inkrementovati i sekvenční číslo, jinak se změny nepropíšou na slave.

## Apache

Nainstalujeme Apache2 `apt install apache2`. Ihned po nainstalování zpřístupní Apache uvítací stránku, což lze ověřit externím prohlížečem, nebo z příkazové řádky prohlížečem **Lynx**.

Pro vytvoření nové stránky musíme do `/var/www/html`, nebo nějaké z podsložek nakopírovat naši webovou stránku \(například wikimedia stránku\). V `/etc/apache2/sites-available` zkopírujeme výchozí webovou stránku **000-default.conf **a použijeme ji jako vzor, naše definice stránky by měla vypadat asi takto:

```text
<VirtualHost wiki.tanas.local:80>
    ServerName wiki.tanas.local

    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

Restartujeme apache `service apache2 reload`.

Přesvědčíme se, že v nastavení DNS propagujeme webovou stránku **wiki.tanas.local** a že tuto stránku dokážeme resolvnout, na stroji z kterého se pokoušíme o přístup musíme mít nastavený výchozí DNS server vlastní DNS server.

