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
```

#### Postup

Nainstalovat postfix. Doménové jmeno můžeme zvolit libovolné.

Konfig je v /etc/postfix

V main configu je nutné nastavit **relayhost, **touto položkou lze definovat na jaký server se všechny zprávy budou přeposílat a ten se o ně dále bude starat a bude řešit kam je přeposlat, kdyby tato položka nebyla nakonfigurováno můžeme rovnou do světa rozesílat emaily, ale ve školní síti je tato možnost blokována.

Položka **mynetworks** definuje sítě, z kterých bude možné emaily odesílat, musíme zde přidat adresu naší sítě. Položka **mydestination** je nutné nastavit, obsahuje domény o kt. se bude tento server starat, musíme zde tedy nastavit naši doménu _tanas.local_.



