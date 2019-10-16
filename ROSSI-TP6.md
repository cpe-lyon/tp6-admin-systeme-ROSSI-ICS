# ROSSI Thomas

# TP 6 - Gestion des disques / Tâches d’administration
# Exercide 1

**1. Dans l’interface de configuration de votre VM, créez un second disque dur, de 5 Go dynamiquement
alloués ; puis démarrez la VM**

**2. Vérifiez que ce nouveau disque dur est bien détecté par le système**

```
thomas@ubuntu-server:~$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0    7:0    0 54,7M  1 loop /snap/lxd/12181
loop1    7:1    0   89M  1 loop /snap/core/7713
loop2    7:2    0 54,6M  1 loop /snap/lxd/11985
loop3    7:3    0 89,1M  1 loop /snap/core/7917
sda      8:0    0   20G  0 disk
├─sda1   8:1    0    1M  0 part
└─sda2   8:2    0   20G  0 part /
sdb      8:16   0    5G  0 disk
sr0     11:0    1 1024M  0 rom
```

On voit bien le disque de 5 Go qui est sdb

**3. Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83),
et une seconde partition de 3 Go en NTFS (n°7)**

```
fdisk /dev/sdb
n
p
+2G

n
p

w
```

**4. A ce stade, les partitions ont été créées, mais elles n’ont pas été formatées avec leur système de fichiers.
A l’aide de la commande mkfs, formatez vos deux partitions**

```
mkfs.ext4 -b 4096 /dev/sdb1
mkfs.ntfs /dev/sdb2
```

**5. Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, ne fonctionne-telle pas sur notre disque ?**

Car les partitions de notre disques de sont pas montés

**6. Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la
machine, respectivement dans les points de montage /data et /win (vous pourrez vous passer des
UUID en raison de l’impossibilité d’effectuer des copier-coller)**

```
/dev/sdb1 /data ext4 defaults 0 0

/dev/sdb2 /win  ntfs defaults 0 0
```

Ajout de ces lignes au fichier /etc/fstab

**7. Utilisez la commande mount puis redémarrez votre VM pour valider la configuration**

```
mount -a
```

Et après redémarrage les partitions sont bien montés

**8. Montez votre clé USB dans la VM**

```
mkdir /USB
mount /dev/sdc /USB
```

**9. Créez un dossier partagé entre votre VM et votre système hôte (rem. il peut être nécessaire d’installer
les Additions invité de VirtualBox**

Création d'un répertoire partager

# Exercice 2. Partitionnement LVM

**1. On va réutiliser le disque de 5 Gio de l’exercice précédent. Commencez par démonter les systèmes de
fichiers montés dans /data et /win s’ils sont encore montés, et supprimez les lignes correspondantes
du fichier /etc/fstab**

FAIT

**2. Supprimez les deux partitions du disque, et créez une patition unique de type LVM**

```
sudo fstab /dev/sdb
d
d

t
8e

w
```

**3. A l’aide de la commande pvcreate, créez un volume physique LVM. Validez qu’il est bien créé, en
utilisant la commande pvdisplay.**

```
pvcreate /dev/sdb1

pvdisplay
"/dev/sdb1" is a new physical volume of "<5,00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb1
  VG Name
  PV Size               <5,00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               bJ0d8Z-ZnmJ-iqc9-9c88-bM7n-wdu2-4omxEF
```

**4. A l’aide de la commande vgcreate, créez un groupe de volumes, qui pour l’instant ne contiendra que
le volume physique créé à l’étape précédente. Vérifiez à l’aide de la commande vgdisplay**

```
vgcreate vg_test /dev/sdb1
Volume group "vg_test" successfully created

vgdisplay
  --- Volume group ---
  VG Name               vg_test
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <5,00 GiB
  PE Size               4,00 MiB
  Total PE              1279
  Alloc PE / Size       0 / 0
  Free  PE / Size       1279 / <5,00 GiB
  VG UUID               DtQ0LX-Sk0k-EGOJ-olMv-95oH-PUKU-qSxQa2
```

**5. Créez un volume logique appelé lvData occupant l’intégralité de l’espace disque disponible**

```
lvcreate -l 100%FREE -n 1vData vg_test
```

**6. Dans ce volume logique, créez une partition que vous formaterez en ext4, puis procédez comme dans
l’exercice 1 pour qu’elle soit montée automatiquement, au démarrage de la machine, dans /data**

```
fdisk /dev/mapper/vg_test-1vData
```

crée la partition

```
mkfs.ext4 /dev/mapper/vg_test-1vData
```

Formatage

Et on l'ajoute dans ```/etc/fstab``` comme fait précedement


**7. Eteignez la VM pour ajouter un second disque (peu importe la taille pour cet exercice). Redémarrez
la VM, vérifiez que le disque est bien présent. Puis, répétez les questions 2 et 3 sur ce nouveau disque.**

FAIT 

**8. Utilisez la commande vgextend <nom_vg> <nom_pv> pour ajouter le nouveau disque au groupe de
volumes**

```
vgextend vg_test /dev/sdc1
```

**9. Utilisez la commande lvresize (ou lvextend) pour agrandir le volume logique. Enfin, il ne faut pas
oublier de redimensionner le système de fichiers à l’aide de la commande resize2fs**


```
lvextend -l 100%FREE /dev/vg_test/1vData
```

Pour étendre partition

```
resize2fs /dev/vg_test/1vData
```

Pour étendre système fichier

# Exercice 3

**1. Programmez une tâche qui affiche un rappel pour la réunion qui aura lieu dans 3 minutes. Vérifiez
entre temps que la tâche est bien programmée.**


**2. Est-ce que le message s’est affiché ? Si la réponse est non, essayez de trouver la cause du problème (par
exemple en vous aidant des logs, du manuel...)**


**3. Pour tester le fonctionnement de cron, commencez par programmer l’exécution d’une tâche simple,
l’affichage de “Il faut réviser pour l’examen !”, toutes les 3 minutes.**


**4. Programmez l’exécution d’une commande tous les jours, toute l’année, tous les quarts d’heure**


**5. Programmez l’exécution d’une commande toutes les cinq minutes à partir de 2 (2, 7, 12, etc.) à 18
heures les 1er et 15 du mois**


**6. Programmez l’exécution d’une commande du lundi au vendredi à 17 heures**


**7. Modifiez votre crontab pour que les messages ne soient plus envoyés par mail, mais redirigés dans un
fichier de log situé dans votre dossier personnel**


**8. Videz votre crontab**
