# Postfix & Dovecot & Roundcube

Pro konfiguraci postfixu použijeme DNS servery z minula. Musíme ověřit, že oba servery mají srávně nastavený NX záznam, tak aby vracel adresu master DNS serveru, taky je nutné ověřit zda oba servery mají v /etc/resolv.conf nastavený jako primární DNS sebe sama.

Nainstalujeme postfix `apt install postfix`. Při instalaci na dotaz výchozí konfigurace zvolíme **internet site** a doménu můžeme ponechat libovolnout, například tanas.local. Konfig postfixu je v **/etc/postfix/main.cfg**.

Postfix funguje hned po instalaci funkčnost otestujeme pomocí telnetu:

```text
telnet smtp.tanas.local 25
helo tanas
mail from: tanas@tanas.local
rcpt to: tanas@tanas.eu
data
msg body
.
quit
```

Ve **/var/log/mail** jde ověřit jestli byl email odeslán, je zde například i ID, kt. server vygeneroval při převzetí pošty.

Doručená pošta je k prohlédnutí ve **/var/mail** je zde soubor s poštou pro každého uživatele.

V main configu je nutné nastavit **relayhost,** touto položkou lze definovat na jaký server se všechny zprávy budou přeposílat \(například smtp.vsb.cz\) a ten se o ně dále bude starat a bude řešit kam je přeposlat, kdyby tato položka nebyla nakonfigurováno můžeme rovnou do světa rozesílat emaily, ale ve školní síti je tato možnost blokována.

Položka **mynetworks** definuje sítě, z kterých bude možné emaily odesílat, musíme zde přidat adresu naší sítě, takže například 172.16.0.0/24. 

Položku **mydestination** je nutné nastavit, obsahuje domény o kt. se bude tento server starat, musíme zde tedy nastavit naši doménu _tanas.local,_ většinou zde tato doména už je protože se nastavuje při instalaci.

> Každý uživatel má ve výchozím stavu svoji poštovní schránku kt. se jmenuje stejně jako uživatelské jméno. Maily lze zobrazit v souboru /var/mail.

Ve výchozím stavu v systému běží poštovní klient mail box, který všechny přijaté emaily zapisuje do jednoho souboru /var/mail, což není úplně vhodné protože s narůstající velikosti souboru je velmi pomalé ho procházet, ještě zde existuje doručování do uživatelských složek, ale to je nutné povolit v nastavení postfixu.

### Aliasy

Není vždy úplně vhodné aby uživatel měl jako emailovou adresu svůj login, vhodné je vytvořit alias, ty se konfigurují v **/etc/aliases**. V tomto souboru jsou vždy dvojice:

```text
alias_adresa: originalni_adresa
alias_adresa: originalni_adresa1,originalni_adresa2
```

Soubor aliases může spoužit i pro definici emailových zkupit, pokud místo originální adresy zadáme více adres oddělených čárkou.

Pro potvrzení změn je nutné zadat příkaz `newaliases`, bez něj se změnu v souboru neaplikují.

Pokud vytvoříme takto alias tak maily sice chodit budou, ale v hlavičce stále bude email s našim loginem, proto je nutné provést tzv. kanonické mapování. V souboru **/etc/postfix/canonical** \(pozor soubor ve výchozím stavu neexistuje musíme ho vytvořit\), je nutné provést mapování a následně nad tímto souborem zavolat příkaz `postmap`

Soubor **/etc/postfix/canonical**:

```text
tanas@tanas.local vlachopulos.tanas@tanas.local
```

aktivace:

```text
postmap /etc/postfix/canonical
```

Příkaz postmap vygeneruje z konfiguračního souboru databázi, postfixu se ještě musí říct, aby tuto databázi používal.

```text
cat /etc/postfix/main.cf | grep canonical_maps
sender_canonical_maps = hash:/etc/postfix/canonical
recipient_canonical_maps = hash:/etc/postfix/canonical
```

Postfix může použít všelijaké typy databází pomocí `postconf -m` můžeme vypsat dostupné databáze.

Funkčnost si ověříme tím, že přes telnet pošleme e-mail ze svého orginálního mailu na alias a když se podíváme do složky s přijatou poštou, měli bychom vidět položku **Return-Path** a **X-Original-To** obojí nastavená na náš alias. 

### Způsob doručování

Volitelně lze pro doručování emalů místo defaultního **mailbox** použít nástroj **maildir**.

V **etc/postfix/main.cfg** nakonfigurujeme option **homedir**, je defaultně zakomentovaná, stačí ji odkomentovat a zakomentovat původní mailbox. e-maily budou následně doručovány do specifické složky tu je pravděpodobně ještě nutné vytvořit a dát jí práva.

```text
touch /var/mail/Maildir
chmod 777 /var/mail/Maildir
```

### Vyzvedávání pošty

Pošta se bude vyzvedávat pomocí protokolu IMAP, nainstalujeme `apt install dvecot-imapd`.  
V **/etc/dovecot/conf.d/** se musí mailbox přepnout na **maildir** nebo **mailbox**, to záleží na nastavení postfixu nastavení musí korespondovat, občas se musí ještě přenout disable **plaintext\_auth** na no.

Restartujeme servisu `service dovecot restart`.

Aby fungovalo automatické objevování poštovního serveru v klientských aplikací, musí se v DNS serveru nastavit SRV záznam, ten bude distribuovat adresu služby SMTP. Stejným způsobem můžeme definovat i SRV záznam pro IMAP. SRV záznamy se přidávají do konfigurace DNS zóny v **/etc/bind/db.tanas.local**

```text
_imap._tcp     SRV 0 1 143 imap.example.com.  ; imap
_imaps._tcp    SRV 0 1 993 imap.example.com.  ; imap sercure
_pop3._tcp     SRV 0 1 110 pop3.example.com.  ; pop3
_pop3s._tcp    SRV 0 1 995 pop3.example.com.  ; pop3 sercure
```

### Roundcube

 Před instalací je nutné ❗ nainstalovat mysql-server, pokud tak nebylo učiněno dříve. Nainstalujeme `apt install roundcube-core`.

Roundcube se nainstaluje do **/var/lib/roundcube**, my musíme udělat symbolický link z složky **/var/www/html** tak aby zde mohl apache nalézt webové rozhraní.

```text
cd /var/www/html
ln -s /var/lib/roundcube
```

Ve výchozím stavu by pak měl být Roundcube dostupný na webové adrese _IP\_serveru/rouncube,_ popřípadě v nastavení DNS můžeme nastavit záznam pro **roundcube.tanas.local**. V nastavení apache pak ještě vytvoříme záznam pro tuto doménu, tak aby nám adresa roundcube.tanas.local směrovala na host s roundube. 

 ❗ Server musí mít v **/etc/resolve.conf** nastaven jako resolver sebe sama, po všech změnách v apachi a DNS se musí služby restartovat a v DNS se musí změnit sekvenční číslo.

Rouncube funguje obecně pro jakýkoliv server SMTP, ale v nastavení rouncdube můžeme nastavit default hosta, ten se nastavuje v  **/var/www/roundcube/config/config.inc.php**, upraví se položka:

```text
$config['default_host'] = '127.0.0.1';
```



