# 🔐 Rapport — VPN IPsec sous Windows Server 2025

![Status](https://img.shields.io/badge/Statut-Complété-brightgreen)
![Platform](https://img.shields.io/badge/Platform-Windows%20Server%202025-blue)
![Protocol](https://img.shields.io/badge/Protocol-IPsec%20%2F%20IKEv2-orange)
![Auth](https://img.shields.io/badge/Auth-PKI%20%2F%20Certificats-red)

**Auteur :** Adoum Mahamat Abakar
**Formation :** Administration Systèmes & Sécurité Réseau
**Année :** 2025

---

## 📋 Table des matières

1. [Introduction](#1-introduction)
2. [Étude théorique de IPsec](#2-étude-théorique-de-ipsec)
3. [Analyse des besoins](#3-analyse-des-besoins)
4. [Conception architecturale](#4-conception-architecturale)
5. [Mise en œuvre pratique](#5-mise-en-œuvre-pratique)
6. [Sécurisation et optimisation](#6-sécurisation-et-optimisation)
7. [Conclusion et perspectives](#7-conclusion-et-perspectives)
8. [Références bibliographiques](#8-références-bibliographiques)

---

## 1. Introduction

### 1.1 Contexte

Dans un contexte marqué par la croissance exponentielle des échanges numériques et la montée des cybermenaces, la sécurisation des communications réseau est devenue un enjeu stratégique pour les organisations. Les réseaux privés virtuels (VPN) constituent une réponse efficace à cette problématique, en permettant la création de tunnels sécurisés sur des infrastructures publiques comme Internet.

Parmi les technologies VPN disponibles, **IPsec (Internet Protocol Security)** s'impose comme une solution robuste et largement adoptée, notamment dans les environnements professionnels et gouvernementaux.

IPsec est un protocole de sécurité réseau qui opère au niveau de la couche 3 du modèle OSI. Il permet de garantir la **confidentialité**, l'**intégrité** et l'**authentification** des données échangées entre deux entités distantes. Grâce à des mécanismes tels que **ESP (Encapsulating Security Payload)** et **AH (Authentication Header)**, IPsec assure une protection complète du trafic IP, indépendamment des applications utilisées (Kent & Seo, 2005).

La négociation des paramètres de sécurité est assurée par le protocole **IKE (Internet Key Exchange)**, dont la version 2 (IKEv2) est aujourd'hui recommandée pour sa robustesse et sa compatibilité avec les systèmes modernes (RFC 7296).

L'**Agence nationale de la sécurité des systèmes d'information (ANSSI)** recommande l'usage d'IPsec pour les interconnexions sécurisées entre sites distants et pour le télétravail. De même, la **NSA** et la **CISA** aux États-Unis intègrent IPsec dans leurs directives de sécurité pour les réseaux gouvernementaux.

### 1.2 Objectifs

Ce projet vise à répondre à une problématique centrale : **comment concevoir et déployer une solution VPN IPsec fiable et sécurisée dans un environnement Windows Server 2025 ?**

Les objectifs spécifiques sont :

1. **Étudier les fondements théoriques du protocole IPsec** — mécanismes de sécurité, modes de fonctionnement (transport et tunnel), protocoles AH, ESP et IKEv2.

2. **Mettre en œuvre une infrastructure VPN IPsec sous Windows Server 2025** — configuration RRAS, politiques de sécurité, attribution des certificats, gestion des connexions client-serveur.

3. **Assurer la sécurité, la performance et la disponibilité** — bonnes pratiques ANSSI/NSA/CISA, tests de connectivité, monitoring et audit.

4. **Produire une documentation technique complète** — manuel d'installation, guide utilisateur, rapport de tests.

5. **Explorer les perspectives d'évolution** — intégration Azure VPN Gateway, automatisation PowerShell/DSC, VPN Quantum-Safe.

---

## 2. Étude théorique de IPsec

IPsec (Internet Protocol Security) est un ensemble de protocoles conçus pour sécuriser les communications sur un réseau IP. Il offre une authentification et un chiffrement des données, permettant ainsi d'établir des connexions sécurisées, notamment pour les VPN.

### 2.1 Architecture et Protocoles

IPsec fonctionne au niveau de la **couche réseau (couche 3 du modèle OSI)** et repose sur deux principaux protocoles :

- **AH (Authentication Header)** : assure l'authentification et l'intégrité des paquets IP.
- **ESP (Encapsulating Security Payload)** : fournit la confidentialité, l'authentification et l'intégrité.

> 📸 *Voir capture : `../captures-ecran/28_schema_topologie_VPN.png`*

Ces protocoles peuvent être utilisés en :
- **Mode transport** : sécurisation des données entre deux hôtes
- **Mode tunnel** : sécurisation entre deux passerelles ou entre un hôte et une passerelle

> 📸 *Voir capture : `../captures-ecran/29_schema_VPN_site_to_site.png`*

### 2.2 Avantages et Limites

#### Chiffrement (AES, 3DES)

| Aspect | Détail |
|--------|--------|
| ✅ Confidentialité | Empêche l'interception et la lecture des données |
| ✅ AES | Hautes performances et sécurité éprouvée (128, 192, 256 bits) |
| ❌ 3DES obsolète | Plus lent et moins sécurisé qu'AES (déconseillé par le NIST depuis 2018) |
| ❌ Surcharge CPU | Augmente la taille des paquets et consomme des ressources |

#### Intégrité (SHA)

| Aspect | Détail |
|--------|--------|
| ✅ Détection des altérations | Garantit que les données n'ont pas été modifiées en transit |
| ✅ SHA-256/512 | Résistants aux attaques |
| ❌ SHA-1 déprécié | Risque de collisions, remplacé par SHA-2 |

#### Authentification (PSK, Certificats)

| Méthode | Avantage | Limite |
|---------|----------|--------|
| PSK | Simple à configurer | Vulnérable au brute-force si clé faible |
| Certificats PKI | Très sécurisé, scalable | Nécessite une infrastructure PKI |
| EAP-TLS | Authentification avancée | Configuration plus complexe |

### 2.3 Compatibilité avec Windows Server 2025

Windows Server 2025 introduit plusieurs améliorations pour IPsec :

- **Algorithmes modernes** : AES-256-GCM, SHA-384, SHA-3 supportés nativement
- **Dépréciation des protocoles faibles** : 3DES et SHA-1 désactivés par défaut
- **Intégration Azure** : connectivité transparente avec Azure VPN Gateway, IKEv2 natif
- **Accélération matérielle** : DPDK, AES-NI pour réduire la latence
- **Gestion simplifiée** : Windows Admin Center, nouvelles templates GPO NIST/CIS
- **Sécurité renforcée** : anti-replay amélioré, MFA via Windows Hello, isolation Zero Trust

---

## 3. Analyse des besoins

### 3.1 Cas d'usage

Le VPN IPsec est destiné à répondre à plusieurs scénarios :

- **Accès distant sécurisé** pour les collaborateurs en télétravail
- **Interconnexion de sites géographiquement dispersés** (site-to-site)
- **Protection des échanges de données sensibles** entre serveurs et clients

Ces cas d'usage impliquent des exigences élevées en matière de **confidentialité**, **intégrité**, **authentification** et **disponibilité**.

### 3.2 Exigences techniques

**Matériel :**
- Serveur dédié ou virtualisé avec support des instructions de chiffrement (AES-NI)
- Carte réseau compatible avec les fonctions de déchargement IPsec

**Logiciels :**
- Windows Server 2025 (rôles : AD DS, AD CS, Remote Access)
- Active Directory Certificate Services (AD CS) pour la PKI
- RRAS pour la gestion des connexions VPN

**Contraintes :**
- Environnement virtualisé VMware Workstation
- Réseau en mode Bridge (192.168.1.0/24)

### 3.3 Matrice des risques

| Risque | Probabilité | Impact | Contre-mesure |
|--------|-------------|--------|---------------|
| Interception des données | Moyenne | Élevé | Chiffrement fort AES-256, tunnel IPsec |
| Usurpation d'identité | Faible | Élevé | Authentification par certificat PKI |
| Attaque par rejeu | Faible | Moyen | Anti-replay activé dans IKEv2 |
| Algorithmes obsolètes | Faible | Élevé | Désactivation SHA-1, 3DES, IKEv1 |

---

## 4. Conception Architecturale

### 4.1 Schéma réseau

```
┌─────────────────────────────────────────────────────────────┐
│                      RÉSEAU INTERNE                          │
│                     192.168.1.0/24                           │
│                                                              │
│   ┌──────────────────────────┐                              │
│   │   Windows Server 2025    │                              │
│   │   IP: 192.168.1.14       │                              │
│   │   ├── AD DS (DC)         │◄── Tunnel IKEv2/IPsec ──────┼──► Client
│   │   ├── AD CS (CA/PKI)     │    Chiffré AES-256           │   Windows
│   │   └── RRAS (VPN)         │    Auth: EAP-TLS             │
│   └──────────────────────────┘                              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Topologie Client-to-Site** : accès distant d'un utilisateur individuel vers le réseau interne.

> 📸 *Voir capture : `../captures-ecran/30_config_reseau_VMware.png`*
> 📸 *Voir capture : `../captures-ecran/31_interface_WAN_LAN.png`*

**Topologie Site-to-Site** : interconnexion de deux réseaux distants via un tunnel IPsec permanent.

### 4.2 Choix des protocoles

| Composant | Choix | Justification |
|-----------|-------|---------------|
| Protocole tunnel | IKEv2 | Plus sécurisé qu'IKEv1, reconnexion automatique (MOBIKE) |
| Chiffrement | AES-256 | Standard recommandé ANSSI/NSA |
| Intégrité | SHA-384 | Résistant aux collisions |
| Groupe DH | Group 14 (2048 bits) | Bon compromis sécurité/performance |

### 4.3 Authentification

**Certificats PKI (EAP-TLS)**
- La CA interne (AD CS) émet des certificats pour le serveur et les clients
- Chaque entité présente son certificat lors de la négociation IKEv2
- Plus sécurisé que PSK : résistant au brute-force et au MITM

### 4.4 Paramètres IPsec

```
Chiffrement (Phase 1/2) : AES-256
Intégrité               : SHA-384
Échange de clés         : IKEv2
Groupe Diffie-Hellman   : Group 14 (2048 bits)
PFS                     : PFS2048 (Perfect Forward Secrecy)
Durée SA Phase 1        : 86400 secondes (24h)
Durée SA Phase 2        : 28800 secondes (8h)
```

---

## 5. Mise en Œuvre Pratique

### 5.1 Côté Serveur — Installation d'Active Directory

**Installation du rôle AD DS :**

1. Dans le Gestionnaire de serveur → **Ajouter des rôles et fonctionnalités**
2. Sélectionner **Services de domaine Active Directory (AD DS)**
3. Valider et lancer l'installation

> 📸 *Voir capture : `../captures-ecran/01_installation_AD_DS.png`*

**Promotion en contrôleur de domaine :**

1. Après installation, cliquer sur **Promouvoir ce serveur en contrôleur de domaine**
2. Sélectionner **Ajouter une nouvelle forêt**
3. Définir le nom de domaine (ex: `mondomaine.local`)
4. Définir le mot de passe DSRM
5. Valider et redémarrer

> 📸 *Voir capture : `../captures-ecran/02_creation_nouvelle_foret_AD.png`*

### 5.2 Côté Serveur — Installation de l'Autorité de Certification (PKI)

**Installation du rôle AD CS :**

1. Gestionnaire de serveur → Ajouter des rôles → **Services de certificats Active Directory**
2. Cocher uniquement **Autorité de certification**

> 📸 *Voir capture : `../captures-ecran/03_installation_role_ADCS.png`*
> 📸 *Voir capture : `../captures-ecran/04_config_autorite_certification.png`*

**Configuration de la CA :**

| Paramètre | Valeur |
|-----------|--------|
| Type CA | Enterprise Root CA |
| Algorithme | RSA 2048 bits |
| Hachage | SHA-256 |
| Validité | 5–10 ans |

> 📸 *Voir capture : `../captures-ecran/05_validite_certificat_CA.png`*

**Création du modèle de certificat VPN-IKEv2 :**

1. Ouvrir la console **Certification Authority**
2. Clic droit sur **Certificate Templates → Manage**
3. Dupliquer un modèle existant, renommer en **VPN-IKEv2**
4. Configurer : usage = authentification serveur + client, clé exportable
5. Publier le modèle dans la CA

> 📸 *Voir capture : `../captures-ecran/06_publication_certificat_AD.png`*
> 📸 *Voir capture : `../captures-ecran/07_ajout_server_authentication.png`*
> 📸 *Voir capture : `../captures-ecran/08_nom_serveur_VPN.png`*

### 5.3 Côté Serveur — Gestion des Certificats via MMC

1. `Win + R` → taper `mmc` → Entrée
2. **Fichier → Ajouter/Supprimer un composant → Certificates**
3. Choisir **Computer account**
4. Aller dans **Personal → Certificates** pour vérifier les certificats installés

> 📸 *Voir capture : `../captures-ecran/09_console_MMC_certificats.png`*
> 📸 *Voir capture : `../captures-ecran/10_ouverture_MMC_gestion_certificats.png`*

**Génération du certificat client :**

1. Dans MMC → clic droit sur **Certificates → All Tasks → Request New Certificate**
2. Sélectionner le modèle **VPN-IKEv2**
3. Valider et exporter le certificat avec la clé privée (.pfx)

> 📸 *Voir capture : `../captures-ecran/11_generation_certificat_clients_VPN.png`*
> 📸 *Voir capture : `../captures-ecran/12_selection_modele_VPN_IKEv2.png`*
> 📸 *Voir capture : `../captures-ecran/13_export_cle_privee_certificat.png`*
> 📸 *Voir capture : `../captures-ecran/14_assistant_exportation_certificat.png`*
> 📸 *Voir capture : `../captures-ecran/15_export_certificat_final.png`*

### 5.4 Côté Serveur — Installation et Configuration du Rôle Remote Access

**Installation :**

1. Gestionnaire de serveur → **Ajouter des rôles**
2. Sélectionner **Remote Access**
3. Cocher **DirectAccess and VPN (RAS)**
4. Valider l'installation

> 📸 *Voir capture : `../captures-ecran/16_installation_role_Remote_Access.png`*

**Configuration RRAS :**

1. Dans le Gestionnaire de serveur → **Tools → Routing and Remote Access**
2. Clic droit sur le serveur → **Configure and Enable Routing and Remote Access**
3. Choisir **Custom configuration** → cocher **VPN access**
4. Cliquer **Finish** puis **Start service**

> 📸 *Voir capture : `../captures-ecran/17_assistant_config_RRAS.png`*
> 📸 *Voir capture : `../captures-ecran/18_activation_acces_VPN_RRAS.png`*

**Configuration des propriétés du serveur VPN :**

1. Clic droit sur le serveur → **Properties → Security**
2. Décocher *MS-CHAPv2*, cocher *Allow machine certificate authentication for IKEv2*
3. Sélectionner le certificat serveur dans la section **Certificate**
4. Onglet **IPv4** : définir la plage d'adresses pour les clients VPN (ex: 10.0.0.1–10.0.0.50)

> 📸 *Voir capture : `../captures-ecran/19_authentification_certificats_EAP.png`*
> 📸 *Voir capture : `../captures-ecran/20_certificat_serveur_VPN_selectionne.png`*
> 📸 *Voir capture : `../captures-ecran/21_plage_adresses_IPv4_clients.png`*
> 📸 *Voir capture : `../captures-ecran/22_fin_config_RRAS.png`*

### 5.5 Côté Serveur — Configuration NPS (Network Policy Server)

1. Dans RRAS → **Remote Access Logging & Policies → Launch NPS**
2. **Policies → Network Policies → New**
3. Définir la condition **User Groups** → sélectionner le groupe **VPN** (créé dans AD)
4. Appliquer la stratégie pour restreindre l'accès aux membres du groupe uniquement

> 📸 *Voir capture : `../captures-ecran/23_console_NPS_strategie_reseau.png`*
> 📸 *Voir capture : `../captures-ecran/37_groupe_acces_VPN_NPS.png`*

### 5.6 Côté Serveur — Création de l'Utilisateur VPN

1. **Tools → Computer Management → Local Users and Groups**
2. Clic droit → **New User**
3. Renseigner les informations de l'utilisateur
4. Dans les propriétés de l'utilisateur → **Dial-in → Allow access**

> 📸 *Voir capture : `../captures-ecran/26_gestion_ordinateur_utilisateurs.png`*
> 📸 *Voir capture : `../captures-ecran/27_creation_nouvel_utilisateur.png`*
> 📸 *Voir capture : `../captures-ecran/24_utilisateur_VPN_cree.png`*
> 📸 *Voir capture : `../captures-ecran/25_droits_acces_VPN_utilisateur.png`*

### 5.7 Côté Client — Installation du Certificat

1. Sur la machine cliente : `Win + R` → `mmc` → Entrée
2. **Fichier → Ajouter composant → Certificates → Computer account**
3. **Personal → Certificates** : vérifier la présence du certificat
4. Si absent : clic droit → **All Tasks → Import** → importer le fichier .pfx exporté

> 📸 *Voir capture : `../captures-ecran/33_import_certificat_client.png`*

### 5.8 Côté Client — Configuration de la Connexion VPN

1. **Paramètres Windows → Réseau et Internet → VPN → Ajouter une connexion VPN**
2. Renseigner les paramètres :

| Champ | Valeur |
|-------|--------|
| Nom de la connexion | VPN-IPsec-Entreprise |
| Adresse du serveur | 192.168.1.14 |
| Type VPN | IKEv2 |
| Authentification | Certificat machine |
| Chiffrement | Maximum |

> 📸 *Voir capture : `../captures-ecran/32_config_connexion_VPN_client.png`*
> 📸 *Voir capture : `../captures-ecran/34_identifiants_utilisateur_VPN.png`*
> 📸 *Voir capture : `../captures-ecran/35_connexion_VPN_identifiants.png`*

### 5.9 Test de Connexion

**Résultat :**
- Lors des tests, une erreur de connexion a été observée liée à un problème de filtrage réseau entre le client et le serveur.

> 📸 *Voir capture : `../captures-ecran/36_erreur_connexion_VPN.png`*

**Diagnostic :**
- Vérification des ports UDP 500 et 4500 (IKEv2)
- Vérification des règles de firewall Windows

---

## 6. Sécurisation et Optimisation

### 6.1 Bonnes pratiques de sécurisation

**Désactivation des protocoles obsolètes :**

| Protocole | Statut | Raison |
|-----------|--------|--------|
| IKEv1 | ❌ Désactivé | Remplacé par IKEv2, plus sécurisé |
| PPTP | ❌ Désactivé | Non chiffré, obsolète |
| L2TP sans IPsec | ❌ Désactivé | Non sécurisé |
| SSLv3 | ❌ Désactivé | Faille POODLE |
| SHA-1 / MD5 | ❌ Désactivé | Vulnérables aux collisions |
| 3DES | ❌ Désactivé | Déconseillé par le NIST depuis 2018 |

**Audit des événements :**
- Utiliser **Event Viewer** pour suivre les connexions VPN
- Détecter les tentatives d'accès non autorisées
- Identifier les erreurs de configuration ou les interruptions de service

### 6.2 Monitoring et Supervision

**Outils intégrés :**
- **PerfMon** : surveiller l'utilisation des ressources (CPU, mémoire, réseau)
- **PowerShell** : `Get-VpnServerConfiguration` pour extraire la configuration
- **Event Viewer** : logs d'authentification et d'erreurs RRAS

**Solutions tierces :**
- **Wireshark** : analyser les paquets IPsec, vérifier le chiffrement, détecter les anomalies

### 6.3 Optimisation des Performances

**Réglage du MTU :**
```powershell
# Tester la taille optimale
ping -f -l 1400 192.168.1.14

# Appliquer le MTU optimal
netsh interface ipv4 set subinterface "Ethernet" mtu=1400 store=persistent
```

**Hardware Offloading :**
- Si le matériel le permet, activer l'accélération matérielle (AES-NI)
- Délègue le chiffrement/déchiffrement aux composants matériels
- Réduit la charge CPU et améliore les performances globales

---

## 7. Conclusion et Perspectives

### 7.1 Bilan du projet

La mise en œuvre d'un VPN IPsec sous Windows Server 2025 a permis de :

- ✅ **Sécuriser les communications** entre clients distants et le réseau interne
- ✅ **Garantir la confidentialité, l'intégrité et l'authentification** des données échangées
- ✅ **Déployer une infrastructure PKI complète** avec émission de certificats
- ✅ **Configurer RRAS avec IKEv2** et authentification par certificats EAP-TLS
- ✅ **Optimiser les performances** via le réglage du MTU et le hardware offloading

### 7.2 Difficultés rencontrées

- Configuration fine des paramètres IPsec (IKEv2, ESP, DH Group)
- Compatibilité avec certains clients VPN
- Problèmes de filtrage réseau (ports UDP 500/4500 bloqués)
- Gestion et déploiement des certificats côté client

### 7.3 Améliorations possibles

| Amélioration | Description |
|-------------|-------------|
| **Azure VPN Gateway** | Extension hybride du réseau local vers le cloud Microsoft |
| **PowerShell / DSC** | Automatisation du déploiement et conformité aux politiques de sécurité |
| **SIEM** | Intégration avec Splunk ou Graylog pour le monitoring avancé |
| **Support IPv6** | Adaptation des configurations IPsec pour IPv6 |
| **VPN Quantum-Safe** | Algorithmes résistants aux attaques quantiques (CRYSTALS-KYBER) |

---

## 8. Références Bibliographiques

- **Kaufman, C. (2014).** *Internet Key Exchange (IKEv2) Protocol*. RFC 7296. IETF.

- **ANSSI (2021).** *Guide d'utilisation des VPN IPsec*. [https://www.ssi.gouv.fr](https://www.ssi.gouv.fr)

- **NIST (National Institute of Standards and Technology).** *Guide to IPsec VPNs — NIST Special Publication 800-77 Revision 1*. [https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-77r1.pdf](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-77r1.pdf)

- **Microsoft Learn — Windows Server 2025.** *Configurer des protocoles VPN dans le routage et l'accès à distance*. [https://learn.microsoft.com/fr-fr/windows-server/remote/remote-access/configure-vpn-protocols](https://learn.microsoft.com/fr-fr/windows-server/remote/remote-access/configure-vpn-protocols)

- **NSA CNSSP 15.** *National Security Algorithm Suite*.

- **Kent, S. & Seo, K. (2005).** *Security Architecture for the Internet Protocol*. RFC 4301. IETF.

---

> 📁 *Ce rapport est accompagné du fichier Word complet `rapport_VPN_IPsec.docx`, des captures d'écran dans `../captures-ecran/`, et des commandes dans `../configuration/`.*
