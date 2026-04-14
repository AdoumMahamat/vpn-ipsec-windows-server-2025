# 🔐 VPN IPsec sous Windows Server 2025

![Status](https://img.shields.io/badge/Statut-Complété-brightgreen)
![Platform](https://img.shields.io/badge/Platform-Windows%20Server%202025-blue)
![Protocol](https://img.shields.io/badge/Protocol-IPsec%20%2F%20IKEv2-orange)
![Auth](https://img.shields.io/badge/Auth-PKI%20%2F%20Certificats-red)

> Mise en place d'un tunnel VPN sécurisé basé sur IPsec/IKEv2 permettant à des utilisateurs distants d'accéder au réseau interne de manière chiffrée et authentifiée via une infrastructure à clés publiques (PKI).

---

## 📋 Table des matières

- [Contexte](#-contexte)
- [Architecture](#-architecture)
- [Technologies utilisées](#-technologies-utilisées)
- [Étapes de mise en œuvre](#-étapes-de-mise-en-œuvre)
- [Paramètres de sécurité](#-paramètres-de-sécurité)
- [Résultats](#-résultats)
- [Problèmes rencontrés](#-problèmes-rencontrés)
- [Améliorations possibles](#-améliorations-possibles)
- [Auteur](#-auteur)

---

## 🎯 Contexte

Dans le cadre d'un projet académique en **Administration Systèmes et Sécurité Réseau**, ce projet vise à concevoir et déployer une solution VPN d'entreprise complète. L'objectif est de permettre un accès distant sécurisé au réseau interne tout en garantissant la confidentialité, l'intégrité et l'authenticité des communications.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    RÉSEAU INTERNE                        │
│                                                          │
│   ┌─────────────────────┐                               │
│   │  Windows Server 2025│                               │
│   │  ├── Active Directory│                              │
│   │  ├── AD CS (PKI/CA) │◄──── Tunnel IPsec/IKEv2 ─────┼──► Client Windows
│   │  └── RRAS (VPN)     │                               │
│   └─────────────────────┘                               │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

**Composants :**
| Rôle | Machine | Description |
|------|---------|-------------|
| Serveur VPN | Windows Server 2025 | RRAS + AD CS + Active Directory |
| Client VPN | Windows 10/11 | Connexion distante via IKEv2 |

**Topologies couvertes :**
- 🔹 **Client-to-Site** — accès distant d'un utilisateur individuel
- 🔹 **Site-to-Site** — interconnexion de deux réseaux distants

---

## 🛠️ Technologies utilisées

| Technologie | Rôle |
|-------------|------|
| Windows Server 2025 | Système d'exploitation serveur |
| IPsec / IKEv2 | Protocole de tunnelisation sécurisé |
| Active Directory (AD DS) | Gestion des identités et des utilisateurs |
| AD CS (Certificate Authority) | Infrastructure PKI — émission des certificats |
| RRAS | Service VPN et routage distant |
| EAP-TLS | Méthode d'authentification par certificat |

---

## 📌 Étapes de mise en œuvre

### Phase 1 — Préparation de l'infrastructure
- [x] Installation et configuration d'**Active Directory (AD DS)**
- [x] Promotion du serveur en **contrôleur de domaine**
- [x] Installation de l'**Autorité de Certification (AD CS)**

### Phase 2 — Gestion des certificats PKI
- [x] Configuration de la **CA racine**
- [x] Génération et déploiement des **certificats serveur et client**
- [x] Validation de la chaîne de confiance

### Phase 3 — Configuration du VPN
- [x] Installation du rôle **Remote Access (RRAS)**
- [x] Configuration du **tunnel IPsec/IKEv2**
- [x] Définition des **politiques de sécurité IPsec**

### Phase 4 — Configuration côté client
- [x] Installation du certificat client
- [x] Création et configuration de la **connexion VPN**
- [x] Tests de connectivité et validation

---

## 🔒 Paramètres de sécurité

```
Chiffrement   : AES-256
Intégrité     : SHA-384
Échange clés  : IKEv2 + Diffie-Hellman Group 14
Authentification : EAP-TLS (Certificats PKI)
Protocoles désactivés : PPTP, L2TP sans IPsec, MD5, DES
```

**Bonnes pratiques appliquées :**
- ✅ Désactivation des algorithmes cryptographiques obsolètes
- ✅ Audit des événements d'authentification activé
- ✅ Renouvellement automatique des certificats configuré
- ✅ Filtrage des connexions non autorisées

---

## ✅ Résultats

- ✔️ Tunnel VPN IPsec/IKEv2 établi avec succès
- ✔️ Authentification par certificat PKI fonctionnelle
- ✔️ Chiffrement AES-256 actif sur toutes les communications
- ✔️ Accès au réseau interne validé depuis le client distant
- ✔️ Logs d'événements configurés pour le monitoring

---

## ⚠️ Problèmes rencontrés

| Problème | Cause | Solution |
|----------|-------|----------|
| Échec de connexion VPN | Certificat mal déployé côté client | Réémission et réinstallation du certificat |
| Problèmes de filtrage réseau | Firewall Windows bloquant les ports IPsec | Ouverture des ports UDP 500 et 4500 |
| Erreur d'authentification EAP | Mauvaise configuration du profil VPN | Reconfiguration de la méthode EAP-TLS |

---

## 🚀 Améliorations possibles

- [ ] **Intégration Azure VPN Gateway** — extension vers le cloud Microsoft
- [ ] **Automatisation PowerShell / DSC** — déploiement scriptable et reproductible
- [ ] **Monitoring avancé** — intégration avec SIEM (Splunk, Graylog)
- [ ] **Support IPv6** — migration vers une stack réseau moderne
- [ ] **VPN Quantum-Safe** — préparation à la cryptographie post-quantique

---

## 📁 Structure du projet

```
VPN-IPsec-WindowsServer2025/
├── README.md
├── rapport/
│   └── rapport_VPN_IPsec.docx
├── configuration/
│   ├── commandes_PowerShell.md
│   └── notes_de_config.md
└── captures-ecran/
    ├── config_VPN.png
    ├── erreur_connexion.png
    └── installation_AD.png
```

---

## 👤 Auteur

**Adoum Mahamat Abakar**
Étudiant en Informatique — option Administration Systèmes & Sécurité Réseau

[![GitHub](https://img.shields.io/badge/GitHub-AdoumMahamat-black?logo=github)](https://github.com/AdoumMahamat)

---

> 📚 *Projet réalisé dans le cadre de la formation en Administration Systèmes et Sécurité Réseau*
