# General CheatSheet

## Spustitelný příkaz

1. Vytvořit v adresáři /usr/bin/
2. Na začátek vložit interpreta skriptu \#!/bin/bash
3. Nastavit soubor jako spustitelný chmod a+x

## Nahrávání souborů pomocí SCP a RSYNC

```text
scp foobar.txt your_username@remotehost.edu:/some/remote/directory
```

Kopírování více souborů najednou

```text
scp {foobar.txt,foobar2.txt} your_username@remotehost.edu:/some/remote/directory
```

Kopírování složky na vzdálený server

```text
scp -r mdhash_cuda2 vla0054@merlin1.cs.vsb.cz:/home/fei/vla0054/mdhash_cuda2
```

Rekurzivní kopírování složky pomocí RSYNC:

```text
rsync --rsh="ssh -l vla0054" --verbose -r soj-cv02/ linedu.vsb.cz:/home/fei/vla0054/soj-cv02
```

## Nastavení SSH

Nastavení bezheslového přihlašování

1. vygenerujeme nový klíč, jako příponu použijeme libovolný identifikátor hosta

`ssh-keygen -t rsa -f ~/.ssh/id_rsa.vsb`

1. heslo necháme prázdné \(odklepneme enterem\)
2. vytvoříme soubor \`~/.ssh/config\` \(viz obsah souboru config\)
3. změníme práva souboru config \`chmod 600 ~/.ssh/config\`
4. pomocí nástroje copy-id nahrajeme na vzdálený server náš klíč

`ssh-copy-id -i ~/.ssh/id_rsa.vsb.pub vla0054@linedu.vsb.cz`

1. nyní by mělo fungovat přihlášení bez hesla 

`ssh vsb` a taky odesílání souborů

`rsync --rsh="ssh" --verbose -r soj-cv03/ vsb:/home/fei/vla0054/soj-cv03`

1. na vzdáleném serveru ověříme zda ve složce \`~/.ssh/authorized\_keys\` je náš klíč a zda tam nejsou nějaké duplicity

Obsah souboru config:

```text
Host vsb
    Hostname linedu.vsb.cz  \# adresa vzdáleného serveru
    IdentityFile ~/.ssh/id\_rsa.vsb  \# coubor s klíčem
    User vla0054  \# uživatel
```

### Odkazy:

[http://nerderati.com/2011/03/17/simplify-your-life-with-an-ssh-config-file/](http://nerderati.com/2011/03/17/simplify-your-life-with-an-ssh-config-file/)

[http://www.thegeekstuff.com/2008/11/3-steps-to-perform-ssh-login-without-password-using-ssh-keygen-ssh-copy-id/](http://www.thegeekstuff.com/2008/11/3-steps-to-perform-ssh-login-without-password-using-ssh-keygen-ssh-copy-id/)

## Email

```text
apt-get install heirloom-mailx
```

**Putty**

* zapnutí numerického bloku v textových editorech - Terminal -&gt; features -&gt; Disable application keypad mode
* vypnuti potvrzeni zavreni - Window -&gt; behaviour -&gt; warn before closing window

**Příkazy** 

Cmd \| Desc

----\|-----

nl \| Čísla řádků

\`for i in $\(seq 1 $END\); do echo $i; done\` \| For cyklus v určitém rozsahu \(včetně počátečního a konečného čísla\)

\`for i in {a..z}; do echo &i; done\` \| iterace skrze abecedu

htop \| Procesy běžící v systému

GET\| Obdoba wget, ale data nikam neukládá, pracuje se s ním jako s proměnnou \(z balíku libwww-perl\)

du \| Zjistí velikost souboru nebo složky

df \| zobrazi vyuziti disku

\`$\(printf "user%0.3d" $number\)\` \| Tisk s číslem formátovaným na 000, 001, 002, …

\`tar -zcf nazev-archivu.tar.gz /zdroj\` \| zatarování složky

\`tar -zxf nazev-archivu\` \| odtarování složku do aktuálního adresáře

\`date +"%d-%m-%Y-%H:%M"\` \| čas s formátováním

\`ln -s cesta-k-adresari nazev-linku\` \| vytvoření symbolického linku

\`tail -f /var/log/messages\` \| Žívý vypis logu na turrisu

\`mysql -u root -p\` \| Přihlášení k mysql

\`ps -ef\` \| Výpis procesů a akcí kt. Proces provedl

\`dpkg-query -l 'foo\*'\` \| Výpis instalovaných balíčků

\`du -hsx \* \| sort -rh \| head -n 10\` \| vylistování složky s velikostí složek v čitelné podobě

**čas**

[http://www.cyberciti.biz/faq/linux-unix-formatting-dates-for-display/](http://www.cyberciti.biz/faq/linux-unix-formatting-dates-for-display/)

**Zajímavé lokality**

/etc/debian\_version Verze debianu

/etc/motd uvítací zpráva při spuštění systému

**Síť**

Cmd \| Desc

----\|------

\`ip link set eth0 up/down\` \| vypnuti a zapnuti interfacu

\`ip addr add 192.138.1.0/24 dev eth0\` \|pridat na rozhranni adresu

\`ip addr del 192.138.1.0/24 dev eth0\` \| odebrat z rozhranni adresu

\`ip route add default via 192.168.1.0\` \| pridani vychozi brany

\`netstat -tulpn\` \| Zobrazení využitých portů

Přesměrování portu:

\`iptables -t nat -A PREROUTING -p tcp --dport 10010 -j DNAT --to 172.16.0.10:80\`

Zrušení přesměrování

\`iptables -t nat -D PREROUTING -p tcp --dport 10010 -j DNAT --to 172.16.0.10:80\`

## Nastavení prostředí

**Nastavení APT**

Při instalaci z dvd je jako zdroj apt balíčku nastaveno defaultně dvd následující úpravou source listu se jako zdroj balíčků nastaví online repozitář v \`/etc/apt/source.list\`

\`\`\`

\# deb cdrom:\[Debian GNU/Linux 8.3.0 \_Jessie\_ - Official amd64 DVD Binary-1 20160123-19:03\]/ jessie contrib main

\# deb cdrom:\[Debian GNU/Linux 8.3.0 \_Jessie\_ - Official amd64 DVD Binary-1 20160123-19:03\]/ jessie contrib main

\# deb [http://security.debian.org/](http://security.debian.org/) jessie/updates main contrib

\# deb-src [http://security.debian.org/](http://security.debian.org/) jessie/updates main contrib

deb [http://httpredir.debian.org/debian](http://httpredir.debian.org/debian) jessie main

deb-src [http://httpredir.debian.org/debian](http://httpredir.debian.org/debian) jessie main

deb [http://httpredir.debian.org/debian](http://httpredir.debian.org/debian) jessie-updates main

deb-src [http://httpredir.debian.org/debian](http://httpredir.debian.org/debian) jessie-updates main

deb [http://security.debian.org/](http://security.debian.org/) jessie/updates main

deb-src [http://security.debian.org/](http://security.debian.org/) jessie/updates main

\`\`\`

\*\*autodopňování parametrů příkazů:\*\*

1. nainstalovat balík apt-get install bash-completion
2. do souboru ~/.bashrc přidat:

\`\`\`

\# enable bash completion in interactive shells

if ! shopt -oq posix; then

if \[ -f /usr/share/bash-completion/bash\_completion \]; then

```text
. /usr/share/bash-completion/bash\_completion
```

elif \[ -f /etc/bash\_completion \]; then

```text
. /etc/bash\_completion
```

fi

fi

\`\`\`

\*\*barevný shell:\*\*

do souboru ~/.bashrc pro roota přidat pro červenou barvu:

\`\`\`

\# Colored prompt

PS1='${debian\_chroot:+\($debian\_chroot\)}\\[\033\[01;31m\\]\u@\h\\[\033\[00m\\]:\\[\033\[01;33m\\]\w\[\033\[00m\\]$ '

\`\`\`

do souboru ~/.bashrc pro uživatele přidat:

\`\`\`

force\_color\_prompt=yes

\`\`\`

Barvy: [http://misc.flogisoft.com/bash/tip\_colors\_and\_formatting](http://misc.flogisoft.com/bash/tip_colors_and_formatting)

\#\#\#VirtualBox

Instalace dodatečných funkcí v grafickém prostředí

Umožňuje plynulé přecházení myši, sdílení schránky a přirozené rozlišení monitoru

1. Spustíme grafické prostředí
2. V liště VB &gt; zařízení &gt; Vložit obraz z CD disku s přídavky pro hosta
3. Potvrdit vyskakovací okno
4. Z CD ROM zkopírovat soubor VBoxLinuxAdditions.run někam do PC
5. Učinit soubor spustitelným chmod +x VBoxLinuxAdditions.rum
6. Spustit script ./VBoxLinuxAdditions.run
7. Počkat a restartovat

\#\#\#Funny utility

Cmd \| Desc

----\|------

figlet \| asci art text

\`vlc -V aa film.\*\` \| zobrazí pixel art

### GIT

```text
git config --global user.name "Tanas Vlachopulos"
git config --global user.email "tanas@tanas.eu"
git config --global color.u auto
git config --global push.default simple
git config --global alias.st status
git config --global alias.ci commit
git config --global alias.co checkout
git config --global alias.br branche
git config --global alias.ll 'log --oneline --graph --all --decorate'
```



