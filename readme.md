# X260 Multiboot (Debian, MacOS, Windows 11)

## Configuration hardware & software

Hardware : 

- Intel i5 6200U  
- 8Go de RAM  
- Ecran 12.5' (1366 x 768)  
- SSD Samsung 870 EVO (500Go)  
- Intel HD Graphics 520
  
  
Paramètres BIOS :

**BIOS en version 1.49 (R02ET76W)**

- Security -> Security Chip: Enabled (Intel PTT);
- Memory Protection -> Execution Prevention: Enabled;
- Virtualization -> Intel Virtualization Technology: Enabled;
- Virtualization -> Vt-directed IO: Disabled;
- Internal Device Access -> Bottom Cover Tamper Detection: must be Disabled;
- Anti-Theft -> Computrace -> Current Setting: Disabled;
- Secure Boot -> Secure Boot: Disabled;
- UEFI/Legacy Boot: UEFI Only;
- CSM Support: No

## Étapes d'installation

Prérequis :
- USB de 16 Go
- Cloner ce repo
- [Python](https://www.python.org/)
- [Rufus](https://rufus.ie/)
- [MediaCreationTool pour Windows](https://www.microsoft.com/fr-fr/software-download/windows11)
- [ISO Debian](https://www.debian.org/)

-----

### Installer MacOS

**1. Formatter l'USB**

Ouvrez rufus et formattez l'USB en disque d'amorçage EFI en système FAT32

**2. Copier le dossier EFI**

Copiez le dossier "EFI" présent dans le dossier MACOS

**3. Télécharger le recovery de Ventura**

Allez dans le dossier TOOLS et ouvrez une fenêtre de commande (Maj + clic droit -> Ouvrir une fenêtre de commandes ici) puis entrez la commande :  
`py macrecovery.py -b Mac-4B682C642B45593E -m 00000000000000000 download`  
Le dossier "com.apple.recovery.boot" apparaitra et sera à copier à la racine de votre USB

**4. Récapitulatif**

Vous devez avoir à la racine de votre USB les dossiers :
- EFI
- com.apple.recovery.boot

**5. Booter sur l'USB**

Appuyez sur la touche F12 au démarrage de votre machine, un menu de sélection apparaitra, choisissez votre USB

**6. Partitionnement du disque**

Une fois l'installateur lancé, il faudra supprimer et partitionner notre disque dur (200Go pour MAC en format APFS)

**7. Installation**

Laissez le système s'installer tranquillement, il devrait redémarrer plusieurs fois
(Pour ne pas à avoir à tout le temps appuyer sur F12 au démarrage, définissez un ordre de BOOT dans le BIOS)

[!IMPORTANT]
> Vous ne pouvez pas démarrer MacOS sans votre USB

### Installation de Windows 11

**1. Création de l'USB**

Lancez l'outil de téléchargement de Windows 11 et préparez une clé USB d'installation. Laissez le programme tourner

**2. Booter sur l'USB**

Appuyez sur la touche F12 au démarrage de votre machine, un menu de sélection apparaitra, choisissez votre USB

**3. Partitionnement du disque**

Une fois dans l'installateur de Windows choisissez le mode d'installation avancé. Vous arriverez sur une page qui vous permettra de sélectionner sur quelle partition installer Windows. 
- Choisissez la plus grande partition (celle de 200Go environ non reconnue étant celle de MacOS). 
- Donnez lui la taille de votre choix (dans mon cas 204 800 Mo (200Go) me suffira)
- Laissez Windows créer ses partitions supplémentaires (Une réservée de 16Mo et une autre de 200Mo)

**4. Installation**

Laissez Windows s'installer

[!WARNING]
> Attention à partir de maintenant vous démarrerez automatiquement sur Windows

**5. Tips**

Si vous n'avez pas envie de connecter votre compte Microsoft à Windows, au moment où l'installation vous proposera de vous connecter appuyez sur les touche MAJ+F10, une fenêtre de commande s'ouvrira et vous pourrez taper la commande suivante :  

`start ms-cxh:localonly`

### Installation de Debian

**1. Création de l'USB**

Ouvrez Rufus, sélectionnez l'ISO de Debian et cliquez sur démarrer

**2. Booter sur l'USB**

Appuyez sur la touche F12 au démarrage de votre machine, un menu de sélection apparaitra, choisissez votre USB

**3. Partitionnement du disque**

*Je considère que vous savez installer un Debian sans que j'ai à tout détailler*

- Dans le mode de partitionnement choisissez : Manuel
- Choisissez la partition restante pour l'installation de Debian (dans mon cas c'est une partition d'un peu moins de 90Go)

> Format: ext4  
> Point de montage : /

[!IMPORTANT]
> Je n'ai pas l'utilité d'une partition SWAP alors je n'en ai pas paramétré

**4. Installation**

Laissez Debian s'installer

### Modification de l'EFI et désinstallation de GRUB

Bootez sur Windows (vous pouvez le faire depuis Debian mais je suis plus à l'aise sur PowerShell)

**1. Monter la partition EFI**

- Lancez un Terminal en mode Administrateur
- Saisissez la commande DISKPART
- Entrez les commandes suivantes :
    - select disk 0
    - list vol
    - repérez le volume de 200 Mo
    - select vol X (ou X est le numéro du volume EFI)
    - assign letter=S

La partition EFI a du apparaitre dans l'explorateur Windows (elle est protégée, vous n'y aurez pas accès depuis l'explorateur)

**2. Modification du contenu de l'EFI**

Dans le dossier REFIND vous trouverez plusieurs sous dossier (BOOT, OC et DEBIAN), nous allons les copier dans notre partition EFI :  
`cd S:\EFI\`  
`xcopy /e C:\Users\\**VotreNomUtilisateur**\Downloads\Multiboot\REFIND\ .\`

Maintenant si vous tapez la commande `ls`vous devez avoir quelques chose comme :
- Boot
- Debian
- Microsoft
- OC
  
**3. Rebooter sur Debian**

A ce stade si vous redémarrez votre PC vous allez arriver sur la sélection du système à démarrer (MacOS par défaut), bootez sur Debian

Pour désinstaller GRUB ouvrez un terminal et tapez la commande :  
`su`  
Entrez le mot de passe root que vous avec défini lors de l'installation de Debian
et entrez la commande :  
`apt remove --purge grub-efi-amd64 grub-efi-amd64-signed grub-common grub-efi-amd64-bin`

### Profiter des systèmes

Si tout s'est bien passé vous pouvez booter sur le système de votre choix

Vérifiez quand même que les systèmes démarrent correctement

-----

## Modification du système sélectionné par défaut

Pour modifier le système sélectionné par défaut dans refind, il va falloir toucher un fichier de configuration (ça veut dire que s'il y a une erreur c'est pas cool) alors vérifiez et revérifiez tout ce que vous faites

**1. Démarrer sur Debian**

Ce sera plus simple de modifier le fichier sur Debian

**2. Modification du fichier refind.conf**

- Ouvrez un Terminal
- Tapez la commande `su` et entrez la mot de passe root
- Tapez la commande `nano /boot/efi/efi/boot/refind.conf`
- Cherchez la ligne 'default_selection  Hackintosh' et remplacez 'Hackintosh' par :
    - Debian
    - Hackintosh
    - Windows
- Sauvegardez le fichier