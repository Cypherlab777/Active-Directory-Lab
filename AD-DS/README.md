# Lab Windows Server 2025 — Active Directory Domain Services

## 🎯 Objectif

Déployer mon premier contrôleur de domaine sous **Windows Server 2025** avec Hyper-V.

L'objectif de ce lab est de mettre en pratique :

* la préparation d'un serveur Windows Server ;
* l'installation d'Active Directory Domain Services ;
* la création d'un domaine ;
* DNS ;
* SYSVOL et NETLOGON ;
* la compréhension des principaux composants d'un contrôleur de domaine.

---

## 🏗️ Architecture

| Élément     | Configuration               |
| ----------- | --------------------------- |
| Hyperviseur | Hyper-V sur Windows 11      |
| Serveur     | `LAB-DC01`                  |
| OS          | Windows Server 2025         |
| Rôle        | Contrôleur de domaine + DNS |
| Domaine     | `Homelab.local`             |
| NetBIOS     | `HOMELAB`                   |
| Réseau      | vSwitch Internal            |

Le vSwitch **Internal** permet aux machines virtuelles du lab de communiquer entre elles et avec l'hôte Hyper-V, sans être directement connectées au réseau physique principal.

J'ai temporairement utilisé un vSwitch **External** pour effectuer l'activation de Windows et les mises à jour du serveur.

---

## 🖥️ Préparation du serveur

Configuration de la VM :

* Génération 2 ;
* 4 Go de RAM ;
* disque VHDX dynamique ;
* Windows Server 2025.

Avant l'installation d'Active Directory :

* renommage du serveur en `LAB-DC01` ;
* configuration d'une adresse IPv4 statique ;
* configuration du serveur DNS vers lui-même, car `LAB-DC01` héberge également le service DNS du domaine.

Un contrôleur de domaine doit conserver une adresse IP stable.

Active Directory dépend fortement de DNS pour localiser les contrôleurs de domaine et les différents services du domaine.

---

## 🧩 Installation d'AD DS

Depuis **Server Manager** :

`Manage → Add Roles and Features → Active Directory Domain Services`

L'installation du rôle **Active Directory Domain Services** ajoute les composants nécessaires à AD DS.

Une fois le rôle installé, le serveur doit ensuite être **promu en contrôleur de domaine**.

---

## 🏢 Création du domaine

Comme il s'agit du premier contrôleur de domaine de mon lab, j'ai choisi :

`Add a new forest`

Domaine :

`Homelab.local`

Nom NetBIOS :

`HOMELAB`

Cette opération crée une nouvelle forêt Active Directory ainsi que le premier domaine de cette forêt.

---

## 🌳 Functional Levels

Configuration choisie :

* Forest Functional Level : **Windows Server 2025**
* Domain Functional Level : **Windows Server 2025**

Les Functional Levels déterminent notamment les fonctionnalités Active Directory disponibles ainsi que les versions de Windows Server pouvant être utilisées comme contrôleurs de domaine.

Ils ne déterminent pas la version de Windows utilisée par les postes clients ou les serveurs membres du domaine.

J'ai choisi **Windows Server 2025** car mon environnement est neuf et tous les futurs contrôleurs de domaine du lab utiliseront Windows Server 2025.

Je n'ai donc pas besoin de conserver une compatibilité avec d'anciennes versions de Windows Server.

Autres options :

* DNS Server : activé ;
* Global Catalog : activé ;
* RODC : non utilisé, car le premier contrôleur de domaine d'un nouveau domaine ne peut pas être un Read-Only Domain Controller.

Un mot de passe **DSRM** est également défini.

DSRM (*Directory Services Restore Mode*) peut être utilisé pour certaines opérations de récupération ou de maintenance d'Active Directory.

Après la vérification des prérequis, le serveur est promu en contrôleur de domaine puis redémarre.

---

## 💾 NTDS.dit

La base Active Directory est stockée par défaut dans :

`C:\Windows\NTDS\ntds.dit`

Elle contient notamment :

* les utilisateurs ;
* les groupes ;
* les ordinateurs ;
* les Organizational Units ;
* les informations du domaine.

Chaque contrôleur de domaine possède sa propre copie de la base Active Directory.

`ntds.dit` constitue donc l'un des éléments essentiels d'un contrôleur de domaine.

---

## 📂 SYSVOL et NETLOGON

Active Directory crée également le dossier :

`C:\Windows\SYSVOL`

SYSVOL contient notamment les fichiers nécessaires aux **Group Policy Objects (GPO)** ainsi que certains scripts utilisés dans le domaine.

Après la promotion du serveur, deux partages réseau importants sont disponibles :

* `SYSVOL`
* `NETLOGON`

### SYSVOL

Depuis mon contrôleur de domaine, le partage est accessible avec :

`\\LAB-DC01\SYSVOL`

SYSVOL contient les fichiers que les ordinateurs et utilisateurs du domaine doivent pouvoir récupérer pour appliquer certaines configurations Active Directory.

Par exemple, lorsqu'une GPO est appliquée à un ordinateur ou à un utilisateur, une partie de sa configuration est récupérée depuis SYSVOL.

Ces fichiers doivent être disponibles sur les contrôleurs de domaine afin que les mêmes stratégies puissent être utilisées dans l'ensemble du domaine.

### NETLOGON

Le partage est accessible avec :

`\\LAB-DC01\NETLOGON`

NETLOGON permet notamment de rendre disponibles des **scripts de connexion** aux utilisateurs du domaine.

Par exemple, un script peut être exécuté automatiquement lorsqu'un utilisateur ouvre sa session.

Le partage NETLOGON expose notamment le dossier `scripts` présent dans l'arborescence SYSVOL.

SYSVOL et NETLOGON sont créés automatiquement lors de la promotion du serveur en contrôleur de domaine.

---

## ✅ Vérifications

Après l'installation, j'ai vérifié :

* la présence du domaine `Homelab.local` ;
* le rôle Active Directory Domain Services ;
* le rôle DNS ;
* la zone DNS du domaine ;
* le partage `\\LAB-DC01\SYSVOL` ;
* le partage `\\LAB-DC01\NETLOGON`.

---

## 📌 Points retenus

Ce lab m'a permis de comprendre que :

* un contrôleur de domaine doit utiliser une adresse IP stable ;
* DNS est essentiel au fonctionnement d'Active Directory ;
* installer le rôle AD DS ne suffit pas : le serveur doit ensuite être promu en contrôleur de domaine ;
* le premier contrôleur de domaine d'une nouvelle forêt crée également le premier domaine ;
* les Functional Levels déterminent notamment les fonctionnalités AD DS disponibles et les versions de Windows Server pouvant être utilisées comme contrôleurs de domaine ;
* `ntds.dit` contient la base Active Directory ;
* SYSVOL contient notamment les fichiers nécessaires aux GPO ;
* NETLOGON est utilisé notamment pour mettre à disposition des scripts de connexion ;
* NETLOGON est lié à l'arborescence SYSVOL ;
* DNS, `ntds.dit`, SYSVOL et NETLOGON font partie des composants importants à comprendre lors du déploiement d'un contrôleur de domaine.

---

## 📚 Sources

* Microsoft Learn — Active Directory Domain Services
* Microsoft Learn — Windows Server 2025
* Cours *Windows Server 2025 Administration* — Kevin Brown
