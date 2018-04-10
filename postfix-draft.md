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

#### Postup

Nainstalovat postfix. Doménové jmeno můžeme zvolit libovolné.

Konfig je v **/etc/postfix**.

V main configu je nutné nastavit **relayhost, **touto položkou lze definovat na jaký server se všechny zprávy budou přeposílat a ten se o ně dále bude starat a bude řešit kam je přeposlat, kdyby tato položka nebyla nakonfigurováno můžeme rovnou do světa rozesílat emaily, ale ve školní síti je tato možnost blokována.

Položka **mynetworks** definuje sítě, z kterých bude možné emaily odesílat, musíme zde přidat adresu naší sítě. Položka **mydestination** je nutné nastavit, obsahuje domény o kt. se bude tento server starat, musíme zde tedy nastavit naši doménu _tanas.local_.

> Každý uživatel má ve výchozím stavu svoji poštovní schránku kt. se jmenuje stejně jako uživatelské jméno. Maily lze zobrazit v souboru /var/mail.

Ve výchozím stavu v systému běží poštovní klient mail box, který všechny přijaté emaily zapisuje do jednoho souboru /var/mail, což není úplně vhodné protože s narůstající velikosti souboru je velmi pomalé ho procházet.

Není vždy úplně vhodné aby uživatel měl jako emailovou adresu svůj login, vhodné je vytvořit alias, ty se konfigurují v /etc/aliases. V tomto souboru jsou vždy dvojice:

```
alias_adresa: originalni_adresa
alias_adresa: originalni_adresa1,originalni_adresa2
```

Soubor aliases může spoužit i pro definici emailových zkupit, pokud místo originální adresy zadáme více adres oddělených čárkou.

Pro potvrzení změn je nutné zadat příkaz `newaliases`, bez něj se změnu v souboru neaplikují.



