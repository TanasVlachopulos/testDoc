# Postfix

Pro konfiguraci postfixu použijeme DNS servery z minula. Musíme ověřit, že oba servery mají srávně nastavený NX záznam, tak aby vracel adresu master DNS serveru, taky je nutné ověřit zda oba servery mají v /etc/resolv.conf nastavený jako primární DNS sebe sama.

#### Test postfixu

```
telnet smtp.tanas.local 25
helo tanas
mail from: tanas@tanas.local
rcp to: tanas@tanas.eu
msg body
.
quit
```

Ve /var/log/mail jde ověřit jestli byl email odeslán, je zde například i ID, kt. server vygeneroval při převzetí pošty.

### Postup

Nainstalovat postfix. Doménové jmeno můžeme zvolit libovolné.

Konfig je v **/etc/postfix**.

V main configu je nutné nastavit **relayhost, **touto položkou lze definovat na jaký server se všechny zprávy budou přeposílat a ten se o ně dále bude starat a bude řešit kam je přeposlat, kdyby tato položka nebyla nakonfigurováno můžeme rovnou do světa rozesílat emaily, ale ve školní síti je tato možnost blokována.

Položka **mynetworks** definuje sítě, z kterých bude možné emaily odesílat, musíme zde přidat adresu naší sítě. Položka **mydestination** je nutné nastavit, obsahuje domény o kt. se bude tento server starat, musíme zde tedy nastavit naši doménu _tanas.local_.

> Každý uživatel má ve výchozím stavu svoji poštovní schránku kt. se jmenuje stejně jako uživatelské jméno. Maily lze zobrazit v souboru /var/mail.

Ve výchozím stavu v systému běží poštovní klient mail box, který všechny přijaté emaily zapisuje do jednoho souboru /var/mail, což není úplně vhodné protože s narůstající velikosti souboru je velmi pomalé ho procházet.

#### Aliasy

Není vždy úplně vhodné aby uživatel měl jako emailovou adresu svůj login, vhodné je vytvořit alias, ty se konfigurují v /etc/aliases. V tomto souboru jsou vždy dvojice:

```
alias_adresa: originalni_adresa
alias_adresa: originalni_adresa1,originalni_adresa2
```

Soubor aliases může spoužit i pro definici emailových zkupit, pokud místo originální adresy zadáme více adres oddělených čárkou.

Pro potvrzení změn je nutné zadat příkaz `newaliases`, bez něj se změnu v souboru neaplikují.

Pokud vytvoříme takto alias tak maily sice chodit budou, ale  v hlavičce stále bude email s našim loginem, proto je nutné provést tzv. kanonické mapování. V souboru /etc/postfix/canonical, je nutné provést mapování a následně nad tímto souborem zavolat příkaz `postmap`

Soubor /etc/postfix/canonical:

```
tanas@tanas.local vlachopulos.tanas@tanas.local
```

aktivace:

```
postmap /etc/postfix/canonical
```

Příkaz postmap vygeneruje z konfiguračního souboru databázy, postfuxu se ještě musí říct aby tuto databázi používal.

```
cat /etc/postfix/main.cf | grep canonical_maps
sender_canonical_maps = hash:/etc/postfix/canonical
recipient_canonical_maps = hash:/etc/postfix/canonical
```

Postfix může použít všelijaké typy databází pomocí postconf -m můžeme vypsat dostupné databáze.

#### Způsob doručování

Nakonfigurujeme pro doručování emalů místo defaultního mailbox nástroj maildir.

V etc/postfix/main nakonfigurujeme option homedir, která říká že emaily budou doručeny do specifické složky v home adresáři uživatele, kterému email přišel.

#### Vyzvedávání pošty

Pošta se bude vyzvedávat pomocí protokolu IMAP, nainstalujeme dovecot-imapd.  
V /etc/dovecot/conf.d/ se musí mailbox přepnout na maildir a občas se musí ještě přenout disable_plaintext_\_auth na no.

restartujeme servisu.

aby fungovalo automatické objevování poštovního serveru v klientských aplikací, musí se v DNS serveru nastavit SRV záznam, ten bude distribuovat adresu služby SMTP. Stejným způsobem můžeme definovat i SRV záznam pro IMAP.

#### Roundcube

nainstalujeme roundcube-core. Před instalací je nutné vyvořid DB a usera stejně jako v případě apache.

Roundcube se nainstaluje do /var/lib/roundcube, my musíme udělat symbolický link z složky /var/www do této složky roundubu.

V nastavení apache pak ještě vytvoříme záznam pro tuto doménu, tak aby nám adresa roundcube.tanas.local.

Rouncube funguje obecně pro jakýkoliv server SMTP, ale v nastavení rouncdube můžeme nastavit default hosta.



