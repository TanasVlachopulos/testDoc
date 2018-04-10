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





