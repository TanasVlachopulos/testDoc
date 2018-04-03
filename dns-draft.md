# DNS \(draft\)

nainstalovat bind9

nainstalovat dnsutils a udns-utils

158.196.0.53 dns server školy

nslookup utilita pro DNS dotazování, option type= pro změnu typu záznamu \(mx, a , aaa, ...\)

nastavení bindu je v /etc/bind/named.conf.options je třeba odkomentovat forwardery a jako forwrder nastavit nějaký dostupný DNS server, třeba ten školní

v etc/bind se vytvoří další db soubor s novou zónou

example zony zkopírujeme ze seidl webu, upravíme SOA záznam na domenu kt. chceme vytvořit, musíme inkrementovat sekvenční číslo, uvést email \(místo zavináče se píše tečka\)

pomocí záznamu srv lze nastavit o jakou službu se kt. server stará, třeba kt. server obhospodařuje službu na portu 80

ve /etc/bind/named.conf.local se musí přidat vytvořená zona a cesta ke konfiguraci zony

DNS musí mít vždy i sekundární DNS server, pro povolení sekundární DNS se musí v primární povolit v konfiguraci allow-transfer, a nastaví se ACL

na sekundárním serveru se nastaví v konfiguraci server master server a vytvoří se nová zona typu slave a řekne se kt. server je pro tuto zonu mastrem

# Apache

Stránky v apache se vytvářejí v sites-avaliable a zapnuté jsou pak v sites-enabled, stránky se vytvoří v avaliable a apache je pak prolinkuje symbolickými odkazy do enabled v případě že je zapneme

v /etc/resolv.conf je nutné nastavit aby nameserver byl náš DNS server, kdybychom to neudělali apache by zjistil že doména kt. má publikovat existuje ale neukazuje na adresu apache

poté se pomocí a2ensite zapne stránka kt. jsme vytvořili

#### Ukol: vytvořit wiki

stahnout z mediawiki projekty balik media wiki

nakopírovat rozbalené mediawiki do /var/www/..

nainstaluje se php apt install lib apache php ???

nainstalujeme mysql/mariadb

vytvořit novou sites pro media wiki



