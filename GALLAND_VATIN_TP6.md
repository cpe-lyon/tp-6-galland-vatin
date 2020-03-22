GALLAND Cyprien

VATIN Clément

# TP6 : gestion des disques, boot, gestion des logs

## Exercice 1 : Disques et partitions

**1)** Dans l'interface de configuration de la VM (éteinte), page stockage, onglet controlleur SATA, on crée un nouveau disque dur, de taille 5Go dynamiquement alloués.

On démarre ensuite la VM.

**2)** On souhaite afficher les disques, pour vérifier que le nouveau disque est bien pris en compte par le système : On à 2 commandes possibles :

```bash
ls /dev/ | grep "sd"
```

qui va chercher à afficher les fichiers contenant sd dans /dev (donc les disques durs, et la commande 

```bash
lsblk
```

qui affiche, entre autres, les disques durs.

Dans les deux cas, on trouve bien, en plus de sda partitionné en 2, le disque sdb de taille 5Go.

**3)** On veut maintenant partitionner ce disque  en deux : Pour cela on va utiliser fdisk, un utilitaire complet en ligne de commande.

```bash
sudo fdisk /dev/sdb
```

On choisit de créer une partition avec n

Ensuite on choisit une partition primaire.

Le premier secteur commence au début (2048)

Le premier secteur se termine à 2Go, soit 3906250.

De la même manière on crée une deuxième partition primaire de 3Go, en allant du début à la fin de l'espace encore disponible.

On enregistre enfin avec w, qui nous fait quitter fdisk.

On peut au passage vérifier avec ``lsblk`` la bonne création de nos disques (en l'occurence l'un de 1,9 et l'autre de 3,1 Go, soit une probable erreur de calcul de notre part, due sans doute à l'oubli de la prise en compte du début à 2048 et non à 1)

**4)** On va désormais formater les partitions : 

```bash
sudo mkfs.nfts -f /dev/sdb2 # permet de formater la partition de 3Go au format nfts

sudo mkfs.ext4 -b 4096 /dev/sdb1 # On formate l'autre en ext4 par blocs de 4096
```

**5)** La commande ``df -T`` n'affiche pas les informations concernant le nouveau disque dur : En effet, il n'a pas encore été monté, et ne peut donc pas être affiché.

**6)** On va faire en sorte de monter les partitions automatiquement au démarrage, grâce au fichier /etc/fstab.

La première chose à faire consiste à en créer une copie :

```bash
sudo cp /etc/fstab /etc/fstab.bak
```

On peut ensuite l'ouvrir : ``sudo nano /etc/fstab``.

A la suite on rajoute les deux lignes suivantes :

```bash
/dev/disk/by-partuuid/by-partuuid_de_la_premiere_partition /data auto defaults 0 0 

/dev/disk/by-partuuid/by-partuuid_de_la_deuxieme_partition /win nfts defaults 0 0 
```

Il est à noter que les dossiers /data et /win sont à crer nous même avec mkdir...

**7)** On utilise mount (commande temporaire) pour effectuer le montage :

```bash 
mount /dev/sdb1 /data

mount /dev/sdb2 /win
```

Puis on redemarre la vm. Avec ``lsblk`` on peut vérifier la configuration (car la commande affiche les disques et le lieu ou ils sont montés).

**8)** On commence par chercher la clef USB :

```bash
sudo fdisk -l
```

On remarque une nouvelle ligne : sdc1 et sdc2. (celle qui nous interesse est sdc1).

On crée le point de montage :

```bash
mkdir /media/usb
```

On a juste ensuite à monter sdc1 au point créé précedemment :

```bash
sudo mount /dev/sdc1 /media/usb
```

**9)** Pour crer un dossier partagé (sous vmware) :

On va dans *virtual machine settings*, puis *options*.

dans l'onglet shared folder, on coche *always enabled* puis on clique sur *add*.

Et pour le voir dans la vm, il suffit d'aller dans /mnt/hgfs/partage, ou il apparait.

## Exercice 2 : Personnalisation de GRUB

GRUB est le chargeur d’amorçage par défaut d’Ubuntu depuis la version 9.10. Il est largement paramètrable, via un fichier de paramètres /etc/default/grub et des sripts situés dans /etc/grub.d

Contenu de /etc/default/grub au début de l'exercice :

```bash
# If you change this file, run 'update-grub' afterwards to update

# /boot/grub/grub.cfg.

# For full documentation of the options in this file, see:

#   info -f grub -n 'Simple configuration'

GRUB_DEFAULT=0

GRUB_TIMEOUT_STYLE=hidden

GRUB_TIMEOUT=0

GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`

GRUB_CMDLINE_LINUX_DEFAULT="maybe-ubiquity"

GRUB_CMDLINE_LINUX=""

# Uncomment to enable BadRAM filtering, modify to suit your needs

# This works with Linux (no patch required) and with any kernel that obtains

# the memory map information from GRUB (GNU Mach, kernel of FreeBSD ...)

#GRUB_BADRAM="0x01234567,0xfefefefe,0x89abcdef,0xefefefef"

# Uncomment to disable graphical terminal (grub-pc only)

#GRUB_TERMINAL=console

# The resolution used on graphical terminal

# note that you can use only modes which your graphic card supports via VBE

# you can see them in real GRUB with the command `vbeinfo'

#GRUB_GFXMODE=640x480

# Uncomment if you don't want GRUB to pass "root=UUID=xxx" parameter to Linux

#GRUB_DISABLE_LINUX_UUID=true

# Uncomment to disable generation of recovery mode menu entries

#GRUB_DISABLE_RECOVERY="true"

# Uncomment to get a beep at grub start

#GRUB_INIT_TUNE="480 440 1"
```

**1)** On commence par regarder si /etc/default/grub.d/50-curtin-settings.cfg est présent présent dans l'environnement. Ce n'est pas le cas. Si il avait été présent, on aurait modifié son extension avec mv.

**2)** On souhaite que le menu de GRUB s'affiche 10 secondes avant de lancer automatiquementle premier OS du menu.

Pour ce faire on modifie en sudo le fichier/etc/default/grub (fichier qui sera toujours modifié en sudo d'ailleurs).

On modifie la ligne ``GRUB_TIMEOUT=0`` en ``GRUB_TIMEOUT=10``.

Il faut aussi faire apparaitre le menu : On décommente la ligne ``grub_timeout_style=hidden``.

**3)** On met à jour en lancant ``update-grub``

**4)** En redémarrant la VM, on obtient bien le résultat escompté : une apparition du menu pendant 10 secondes, avant de lancer le 1er OS.

**5)** On souhaite désormais augmenter la résolution de GRUB et de notre VM.

Pour récupérer la taille, on appuie sur c dans le menu GRUB au lancement de la VM. Il n'y a ensuite plus qu'a modifier dans /etc/default/grub la ligne ``#GRUB_GFXMODE=640x480`` en ``#GRUB_GFXMODE=1024x768x32``.

**6)** On désire installer un fond d'écran : Pour ce faire :

```bash
sudo apt install grub2-splashimages #installation d'un paquet en proposant plusieurs

sudo update-grub #Pour que GRUB voit les fonds d'écran
```

Les thèmes disponibles sont stockés dans /usr/share/images/grub. Dans le fichier /etc/default/grub, on ajoute la ligne suivante :

``GRUB_BACKGROUND="/usr/share/images/grub/B-1B_over_the_pacific_ocean.tga"``.

**7)** Il est aussi possible d'installer d'autres thèmes, notament grâce à l'adresse https://www.gnome-look.org/p/1328894/.

**8)** On souhaite ajouter au menu de démarrage deux entrées, pour arréter et redemarrer la machine. Pour ce faire, on ouvre avec nano ou vim /etc/grub.d/40_custom, en mode sudo bien sur.

On y ajoute :

```bash 
menuentry "Reboot now"{

  reboot

}

menuentry "Shutdown now"{

  halt

}
```

Ensuite, un simple ``update-grub``, et ca fonctionne.

**9)** Enfin, on souhaite configurer GRUB pour avoir le clavier en francais.

```bash
sudo mkdir /boot/grub/layouts

sudo grub-kbdcomp -o /boot/grub/layouts/fr.gkb fr
```

Puis dans /etc/default/grub, on ajoute à la fin ``GRUB_TERMINAL_INPUT="at_keyboard"``.

De même dans /etc/grub.d/40_custom, on ajoute a la fin :

```bash
# Clavier fr

insmod keylayouts

keymap fr
```

On termine bien sur avec ``update-grub``.

## Exercice 3 : Noyau

Il s'agit maintenant de creer et d'installer un module pour le noyau :

**1)** On commence par l'installation du paquet build-essential, contenant tousles outils nécessaires à la compilation de programmes écrits en C (et pas seulement bien sur).

```bash
sudo apt install build-essential
```

**2)** On crée (hors de la vm) un fichier hello.c, dont voici le contenu :

```bash
#include <linux/module.h>

#include <linux/kernel.h>

MODULE_LICENSE("GPL");

MODULE_AUTHOR("John Doe");

MODULE_DESCRIPTION("Module hello world");

MODULE_VERSION("Version 1.00");

int init_module(void)

{
  
  printk(KERN_INFO "[Hello world] - La fonction init_module() est appelée.\n");

  return 0;

}

void cleanup_module(void)

{

  printk(KERN_INFO "[Hello world] - La fonction cleanup_module() est appelée.\n");

}
```

**3)** De même on crée un fichier makefile :

```bash
obj-m += hello.o

all:

  make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:

  make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean

install:

  cp ./hello.ko /lib/modules/$(shell uname -r)/kernel/drivers/misc
```

Ces deux fichiers sont ensuite transmis à la VM via le dossier partagé précedemment créé.

**4)** On compile le module : ``make``.

On l'istalle avec ``sudo make install``

**5)** Le module est intallé à l'emplacement suivant :

/lib/modules/5.3.0-42-generic/kernel/drivers/misc/hello.ko

On le charge via :

```bash
sudo insmod /lib/modules/5.3.0-42-generic/kernek/drivers/misc/hello.ko

lsmod | grep "hello" #Montre que le module à bien été lancé
```

Pour vérifier que la phrase "La fonction init_module() est appelée" est bien inscrite dans le journal du noyau, on effectue la commande :

```bash
cat /var/log/kern.log | grep "init_module"
```

Et on trouve bien la ligne désirée.

**6)** Non traitée

**7)** On décharge ensuite le module : 

```
sudo rmmod /lib/modules/5.3.0-42-generic/kernel/drivers/misc/hello.ko
```

Puis en cherchant dans le journal du noyeau on trouve la phrase "La fonction cleanup_module() est appelée", preuve du bon déchargement du module.

**8)** Non traitée

## Exercice 4 : Exécution de commandes en différé : at et cron

**1)** On souhaite programmer une tâche affichant un rappel pour une réunion dans 3 minutes :

```bash
at 2.41 PM 

touch /home/usr1/test

ctrl + D
```

On crée ainsi le fichier test dans le dossier de l'utilisateur usr1.

Rien ne sert d'utiliser echo, car on éxécute via /bin/sh, et non via l'interface utilisateur...

On va donc creer un script en sh, contenant juste une ligne : ``echo "rappel"``.

Ensuite on fait chmod 777, et on teste l'execution en executant le fichier.

Cela étant fait, on ouvre ``crontab -e``, puis avec 2 on spécifie le choix de vim.

Une dernière




// A rédiger //


ex 4





http://hardware-libre.fr/2014/03/8-exemples-pour-maitriser-linux-cron/
https://www.linuxtricks.fr/wiki/cron-et-crontab-le-planificateur-de-taches

problème : comment renvoyer sur le terminal actuel ?
on a tapé avant tty qui nous donne le terminal actuel et on mets cette valeur dans le programme
on fait * * * * ./test.sh > /dev/tty1

tache toutes les 3mins ?
*/3 * * * ./tache.sh > /dev/tty

tache tous les 15 mins ?

*/15 * * * ./tache2.sh

taches q5

2/5 18 1,15 * ./tache3.sh > /dev/tty

tache q6

pour ne plus envoyer par mail on ajoute en première ligne : 
MAILTO=""

00 17 1-5 * ./tache4.sh > /dev/tty

chrontab -r supprime les rappels pour toi, sinon chrontab -r usr supprime pour autre usr, mais nescessite root.

Ex5 htop : tous les programmes en cours en temps réel
On passe a tty2 avec ctrl alt F2.
W renvoit les programmes actifs lancés par l'utilisateur.
au login, la dernière connexion apparait
sinon, on peut utiliser le log de connexion :
less /var/log/auth.log

uname -r pour plusieurs infos, uname -r pour juste le noyau

sudo lshw -C CPU -json

journalctl --list-boots pour afficher les derniers demarages
pour afficher après le boot, on utilise journalctl en root avec --until et --since. en donnant les dates entre deux boots. sinon, on peut utiliser --boot[=ID] donc par exemple journalctl --boot -0 pour le dernier, -1 pour l'avant dernier etc.

pour les derniers démarages, on peut faire : journalctl --list-boot | tail -n 5 pour les 5 derniers

pour prevenir d'une maintenance, la meilleure solution est de l'écrire directement dans
/etc/motd

sinon dans le /etc/profile
echo "attention, maintenance de prévue le ..."

Tload montre l'activité du processeur,donc on devrait voir une grande utilisation puis plus rien quand on interrompt !

ex6 pas fait


ex7

tar * compresse que les fichiers non cachés.
tar . compresse tout le dossier actuel, donc fichiers cachés compris

la commande archive tout notre home ?

en s'enlevant les droits en exectution, on a permission denied.En effet, il s'execute dans le dossier, hors il n'a pas les droits ! Il faut donc les droits d'execution sur un dossier pour le compresser.
