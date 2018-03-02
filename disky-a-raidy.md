# Disky a Raidy

## Disky v linuxu

* V linuxu nejsou disky připojené jako fyzické jednotky pod písmenkem, ale jsou svázané s nějakou složkou, každý oddíl disku je taky zvlášť svázán s nějakou složkou
* Disky jsou značeny jako sdx, složka sdx označuje disk jako takový
* Sdx1, sdx2, sdx3 jsou partitions disku
* Partitiony historicky jsou 4, v současných systémech je jich i více, jsou tvořeny rozdělenou části 4
* Disky nemusí mít partition, kromě bootovacího disku, ten je mít musí

* Přítomné disky jsou v souboru /etc/fstab

* UUID - jednoznačný identifikátor disku

* UUID lze zjistit pomocí nástroje blkid \(btw lze spustit pouze jako root\)

* Pokud nevíme co napsat do sloupce options můžeme zadat defaults

* Dump a pass jsou nepodstatné u root file systému by měly být 0 1 u ostatních 0 0

## Připojování disku

1. blkid - vypíše přítomné disky včetně UUID, nově přidaný disk mezi disky pravděpodobně nebude
2. vylistujeme **/dev** a najdeme zařízení, které reprezentuje nový disk \(sdbx, sdcx, ....\)
3. lsblk - vypíše rozdělení disku
    \(optional\)
4. cfdisk /dev/sdx - nástroj pro vytváření rozdělení \(optional pokud nechceme více oddílů\)
5. mkfs.ext4 /dev/sdx - naformátujeme disk
6. zapíšeme záznam do **fstab **s UUID a parametrem defaults, jako mount point uvedeme /home
7. připojíme disk do nějaké dočasné složky `mount -t ext4 /dev/sdX /tmp/sdX`
8. zkopírujeme obsah adresáře home do dočasně připojené složky `cp -r /home/* /tmp/sdX`
9. aplikujeme záznamy z fstabu `mount -a`
10. odpojíme umístní /tmp/sdX -&gt; `umount /tmp/sdX`
11. **df** - zobrazuje zaplnění disků

{% panel style="warning", title="This is a warning panel" %}
Panel with title and warning style.
{% endpanel %}
