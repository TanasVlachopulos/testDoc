# Kompilace Kernelu

> !!! Pro kompilaci jádra je nutné asi 2 GB volného místa na disku. Pokud místo chybí stačí přimountovat nový disk a rozbalení a kompilaci provést na něm.

1. z webu [kernel.org](https://www.kernel.org/) stáhnout nejnovější kernel do nové složky
2. nainstalovat všechny potřebné balíky
   `apt install libelf-dev build-essential ncurses-dev xz-utils libssl-dev bc make gcc libncurses-dev`
3. rozbalit stáhnutý kernel `tar -xf`
4. do rozbalené složky vložit předpřipravený [config](http://seidl.cs.vsb.cz/iso/config_4.9.8) jako soubor **.config** \(optional step\) a upravit požadované řádky
5. v configu upravit pole **local version** aby šlo poznat že se jedná o námi upravený kernel
6. spustit `make menuconfig` zde lze upravit vlastnosti nového kernelu a upravit součásti. Menuconfig nezničí .config soubor, jenom ho načte a provedené změny do něj zpět uloží. Tento krok je nutný protože starý .config soubor neobsahuje options z novějších kernelů a pokud bychom nespustili menuconfig kernel by se nás v průběhu kompilace doptával.
7. zkompilovat kernel do balíčku `make deb-pkg`
8. po kompilaci \(chvíli to trvá\) v nadřazeném adresáři vytvoří několik balíčků .deb
9. nainstalujeme všechny balíčky .deb `dpkg -i linux-*.deb`
10. updatujeme grub `update-grub` a výsledek by měl být přibližně následující:

```
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-4.15.7vla0054
Found initrd image: /boot/initrd.img-4.15.7vla0054
Found linux image: /boot/vmlinuz-4.9.0-4-amd64
Found initrd image: /boot/initrd.img-4.9.0-4-amd64
done
```

#### Ověření

Při startu by měl být vidět custom název kernelu z pole **local version **v configu kernelu:

```
tanas@TanasPC:~$ ssh 192.168.1.128
tanas@192.168.1.128's password:
Linux sus 4.15.7vla0054 #2 Fri Mar 2 17:58:19 CET 2018 x86_64
```

Ověřit pomocí **uname**:

```
root@sus:~$ uname -a
Linux sus 4.15.7vla0054 #2 Fri Mar 2 17:58:19 CET 2018 x86_64 GNU/Linux
root@sus:~$ uname -r
4.15.7vla0054
root@sus:~$ uname -mrs
Linux 4.15.7vla0054 x86_64
```

#### Zdroje

http://www.abclinuxu.cz/clanky/navody/cesta-do-hlubin-kompilace-jadra-1

https://linode.com/docs/tools-reference/custom-kernels-distros/custom-compiled-kernel-debian-ubuntu/

https://www.cyberciti.biz/faq/debian-ubuntu-building-installing-a-custom-linux-kernel/

