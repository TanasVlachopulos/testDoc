# DNS \(draft\)

nainstalovat bind9

nainstalovat dnsutils a udns-utils

158.196.0.53 dns server školy

nslookup utilita pro DNS dotazování, option type= pro změnu typu záznamu \(mx, a , aaa, ...\)

nastavení bindu je v /etc/bind/named.conf.options je třeba odkomentovat forwardery a jako forwrder nastavit nějaký dostupný DNS server, třeba ten školní

v etc/bind se vytvoří další db soubor s novou zónou

example zony zkopírujeme ze seidl webu, upravíme SOA záznam na domenu kt. chceme vytvořit, musíme inkrementovat sekvenční číslo, uvést email \(místo zavináče se píše tečka\)

