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
/dev/sdb1 /data ext4 defaults 0 0

/dev/sdb2 /win nfts defaults 0 0
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
