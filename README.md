## Installation

**Iso : TrueNAS 25.04.2.6**

- Version stable (release)
- Basée sur la branche actuelle recommandée pour la production
- Correctifs de bugs et stabilité
- Compatible avec la majorité des configurations
- Moins de risques de problèmes


1 Disque de 60 Go
2 disques de 80 Go Chacun


```
> Install / Upgrade
> Sélectionner le disque de 60 Go
> Yes
> Administrative user (truenas_admin).
> Mot de passe : toto
> Yes x2
> Laisser l'instalation se terminer
> Sélectionner Reboot System et valider
```

## configuration statique de l'adresse IP

```
> Sélectionner 1 + Entrée
    Entrée
    Ipv4_dhcp : Mettre NO
    Ipv6_auto : Mettre NO
    Alias : Mettre 192.168.20.40/24
    Save
    Appuyer sur p puis a

> Sélectionner Option 2  + Entrée
    Hostname : truenas1
    Domaine : bts.lan
    ipv4 gateway : 192.168.20.254
    nameserver : 192.168.20.10
    Save
```

## Enregistrement de la VM dans le DNS sur le serveur AD

Le faire sur la vm svrad22


## Accès à l'interface Web de Truenas

```
Sur la vm zabbix2, ouvrir le navigateur et rentrer https://truenas1.bts.lan
    Avancé
    Continuer vers truenas.bts.lan (non sécurisé)
    Identifiant : truenas_admin
    Mot de passe : toto
```

## Mise en place du MFA

### Activation de la fonctionnalité

```
> System
> Advanced Settings
> Dans l'onglet Global Two Factor Authentication, cliquer sur Configure
> Cocher  Enable Two Factor Authentication Globally
> Save
```

### Génération du code

```
En haut à droite, cliquer sur Truenas_admin
Two Factor authentification
Copier le code qui se trouve en bas du QR Code

Ouvrir le widget Bitwarden et se connecter au compte bitwarden d'un utilisateur qui a les droits admin
Créer
Identifiant
    Nom d'utilisateur : truenas_admin
    Mot de passe : P@ssword*
    Clé d'autentification : saisir la clé copiée précédemment
    Site Web Uri : https://truenas1.bts.lan
    Enregistrer
```

## Mettre en français

```
> System
> General Settings
> Sur le champs Localization, cliquer sur Configure
    >  Language : French
    > Console Keyboard Map : French (Azerty)
    > Timezone : Europe/Paris
    > Format : dd-mm-yy
    > Save 
```

## Création d'un certificat auto-signé



### Étape 1 — créer un certificat

```
Dans TrueNAS :

```
Idendifiants
Certificats

Dans le champs Autorité de cerification, cliquer sur Ajouter
    Nom :  CA-INT-bts
    Type : AC Interne
    Profil : CA
    Cocher la case Ajouter au magasin de confiance
    Suivant
    Typé de clé : RSA
    Longueur de la clé : 2048
    Algorythme Digest : SHA256
    Durée de vie : 365
    Suivant
    Pays : France
    Etat : Ile-De-France
    Localité : Paris
    Organisation :Imie
    Email : admin@bts.lan
    Nom commun : truenas1.bts.lan
    Nom alternatif de sujet: 
        192.168.20.40
        truenas1.bts.lan
    Suivant
    Usage : SERVER_AUTH
    Laisser cocher les deux champs
    Key Usage COnfig : Laisser tel quel
    Suivant
    Enregistrer
```

Dans le champs certifiats, cliquer sur Ajouter

```
 
    Nom : truenas-cert
    Type : certificat interne
    Profil : HTTPS RSA Certificate
    Cocher la case Ajouter au magasin de confiance
    Suivant
    Normalement les champs sont pré-remplis, sauf la durée de vie où on va mettre 365
    Suivant
     Pays : France
    Etat : Ile-De-France
    Localité : Paris
    Organisation :Imie
    Email : w.mbakoptchouhane@gmail.com
    Nom commun : truenas1.bts.lan
    Nom alternatif de sujet: 
        192.168.20.40
        truenas1.bts.lan
    Suivant x2
    Enregistrer

    Télécharger le certficat d'AUTORITE créé
    Sur un client windows :
        Une fois créer, cliquer sur le fichier. crt et k'installer (Attention: ordinateur local)
    Sur un client linux : Aller dans le navigateur, parametre, vie privée et sécurité, dans la zone certificats, cliquer sur afficher les certificats et cliquer sur importer. Ensuite important le fichier .crt
```
```
Aller Dans Système
Paramètres généraux
Cliquer sur Paramètres dans la zone UI
Certificate SSL GUI : Sélectionner truenas_cert
Enregistrer
Dans la pop-up qui s'affiche, redémarrer les services
```

    


## activer les UUID disque VMware
Dans l'EXSI, 
> Eteindre la VM truenas1
> Cliquer sur la vm truenas1
> Actions
> Edit settings
> VM Options
> Advanced
> Configuration Parameters : cliquer sur Edit configuration
> Add parameter
> Key : disk.EnabledUUID
> Value : TRUE
> Save

## Création de volumes

> Stockage
> Disques
    > Sélectionner le premier disque de 80Go
    > Cliquer sur Effacement x2
    > Cocher Confirmer
    > Cliquer sur Continuer
    > Faire la même chose pour l'autre disques
    > Cliquer sur Fermer
> Stockage
> Créer un volume
> Nom : volume1
> Suivant
> Layout : RAID1 
    les données sont écrites sur les deux disques simultanément,
    chaque disque contient une copie complète.
    Donc :
    si un disque tombe en panne,
    le NAS continue de fonctionner,
    tu remplaces le disque défectueux,
    le miroir se reconstruit.
    Avec 2 disques, c’est le seul vrai RAID redondant simple.
> Dans la zone Option avancée :
    > Cliquer sur Sélection manuelle des disques
    > Les deux disques vont apparaitre dans la zone VDEVs
    > Cliquer sur Enregistrer la sélection
    > Cliquer sur Enregistrer et passer en revue
    > Créer un volume
    > Cocher Confirmer et continuer




## Joindre TrueNas à l'Active Directory

> Identifiants
> Service d'annuaire
> Configurer Active Directory
    > Nom du domaine : bts.lan
    > Nom de compte de domaine : Administrateur
    > Mot de passe du compte de domaine : P@ssw0rd
    > Cocher  Activer (requiert le mot de passe ou le principal Kerberos) 
    > Enregistrer

## Création de jobs de sauvegardes

### Créer un dataset pour les sauvegardes

> Datasets
> Cliquer sur Volume 1
> Ajouter un dataset
    > Nom : backup
    > Préréglage dataset :  SMB
    > Laisser cocher Créer un partage SMB
    > Laisser Nom SMB : Bakcup
    > Enregistrer
    > Démarrer


### Créer un groupe NAS_BACKUP dans l'active Directory

> Outils
> Utilisateurs et ordinateurs Active Directory
> Deplier bts.lan
> Cliquer droit sur Users
> Nouveau
> Groupe
> Nom du groupe : nas_backup
> Ok
> Double cliquer sur nas_backup
> onglet Membres
> Ajouter
> Ajouter Administrateur, Celina et William
> Appliquer
> Ok

### Mettre en place les permissions sur le partage nouvellement créé

> Datasets
> backup
> Dans la zone > Autorisations, cliquer sur Modifier
> Ajouter une entrée
> Qui : Groupe
> Groupe : bts\nas_backup
> Cocher Appliquer les autorisations de manière récursive
> Confirmer
> Continuer
> Sauvegarder ACL

ATTENTION ; Vérifier dans Systèmes > Services que SMB est activé

### Test d'accès depuis la VM Zabbix

Installer CIFS :

```bash
sudo su
```

```bash
apt install cifs-utils -y
```

Créer point de montage :

```bash
mkdir /backup
```

Créer point de montage :

```bash
mount -t cifs //truenas.bts.lan/backup /backup -o username="Administrateur",domain=bts
```

Vérifier que le montage fonctionne :

```bash
df -h
```

Test :

```bash
touch /backup/test.txt
```

```bash
ls /backup
```
Les fichiers créés sur la VM Zabbix vont apparaitre



Dans la vm Truenas, sélectionner l'option 8
ls /mnt/volume1/backup



Pour afficher les fichiers depuis une VM Linux

```bash
sudo apt install cifs-utils -y
mkdir /backup
mount -t cifs //truenas.bts.lan/backup /backup -o username="Administrateur",domain=bts
ls /backup
cat /backup/test.txt
```

Pour afficher les fichiers depuis une vm Windows


Dans l'explorateur Windows 
\\truenas1.bts.lan\backup


### Rendre le montage persistent : 

Créer un fichier contenant les identifiants

```bash
nano /root/.smbcredentials
```

```
username=administrateur
password=P@ssword
domain=bts
```

Sécuriser le fichier

```bash
chmod 600 /root/.smbcredentials
```

Ajouter le montage dans /etc/fstab

```bash
nano /etc/fstab
```

Ajouter :

```
//truenas.bts.lan/backup /backup cifs credentials=/root/.smbcredentials,iocharset=utf8,vers=3.0,_netdev 0 0
```

Tester sans redémarrer

```bash
mount -a
```

```bash
df -h
```

On voit /backup


Vérifier après reboot

```bash
reboot
```

```bash
df -h
```

On voit /backup

### Mapper un lecteur réseau SMB depuis Windows

Tester la connectivité : 
ping truenas.bts.lan


net use Z: \\truenas.bts.lan\backups /user:bts\administrateur /persistent:yes

dir Z:


Tester l'écriture
echo "test sauvegarde" > Z:\test2.txt

Vérifier dans TrueNas
ls /mnt/volume1/backup


### Gestion des quotas utilisateurs

> Datasets
> Dans la zone Gestion de l'espace du dataset, cliquer sur Gérer les quotas du groupe
> Ajouter
> Quota de données sur les utilisateurs (Exemples : 500 KiB, 500M, 2 TB) : 1 GB
> Appliquer des quotas aux utilisateurs sélectionnés 
> Appliquer aux utilisateurs : Sélectionner le(s) utilisateurs
> Enregistrer

PS : TrueNAS ne permet PAS nativement d’appliquer automatiquement un quota à tous les utilisateurs AD d’un coup
On peut mettre alors en place un quota d'un groupe (bts\utilisateurs du domaine)
