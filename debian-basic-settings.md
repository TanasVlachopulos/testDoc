# Debian Basic Settings

### Upravit bashrc

Upravit barvu rootovského promptu a nastavit aliasy.

```text
PS1='${debian_chroot:+($debian_chroot)}\[\033[01;31m\]\u@\h\[\033[00m\]:\[\033[01;33m\]\w\[\033[00m\]$ '

export LS_OPTIONS='--color=auto'
eval "`dircolors`"
alias ls='ls $LS_OPTIONS'
alias ll='ls $LS_OPTIONS -lisa'
```

### Upravit nastavení SSHD

Upravit nastavení SSHD démona tak aby se dalo přihlásit přes SSH jako root.

```text
echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
service ssh restart
```

## Nastavení sítě

Na všech interfacech zapnout dhcp klienta, nebo nastavit statickou adresu.

U novějšího Debianu jsou interfacy číslovány enp0s3, enp0s8, ...

```bash
# Dynamic DHCP
auto eth0
allow-hotplug eth0
iface eth0 inet dhcp

# Static
auto eth1
allow-hotplug eth1
iface eth1 inet static
        address 192.168.56.99
        netmask 255.255.255.0
        gateway 192.168.56.1
```

## Nastavení balíčků

Odstranit cdrom balíčky z APT.

```text
sed -i '/cdrom/d' /etc/apt/sources.list
```

## Nadbytečnosti

```text
# hello message
apt install figlet
figlet SUS > /etc/motd
```

## GIT

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



