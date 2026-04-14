# 📝 Notes de Configuration — VPN IPsec Windows Server 2025

> Notes techniques détaillées sur la configuration du tunnel VPN IPsec/IKEv2 avec authentification par certificats PKI.

---

## 📋 Table des matières

- [Environnement de travail](#-environnement-de-travail)
- [Paramètres IPsec](#-paramètres-ipsec)
- [Configuration Active Directory](#-configuration-active-directory)
- [Configuration PKI / CA](#-configuration-pki--ca)
- [Configuration RRAS](#-configuration-rras)
- [Configuration Client VPN](#-configuration-client-vpn)
- [Ports et Protocoles](#-ports-et-protocoles)
- [Problèmes rencontrés](#-problèmes-rencontrés)
- [Bonnes pratiques](#-bonnes-pratiques)

---

## 🖥️ Environnement de travail

| Composant | Détail |
|-----------|--------|
| Hyperviseur | VMware Workstation |
| Serveur | Windows Server 2025 |
| Client | Windows 10 |
| Réseau | Mode Bridge — 192.168.1.0/24 |
| IP Serveur | 192.168.1.14 |
| IP Client | 192.168.1.x |
| Domaine | mondomaine.local |

---

## 🔒 Paramètres IPsec

### Algorithmes cryptographiques

| Paramètre | Valeur choisie | Raison |
|-----------|----------------|--------|
| Chiffrement | AES-256 | Standard recommandé ANSSI/NSA |
| Intégrité | SHA-384 | Résistant aux collisions |
| Échange de clés | IKEv2 | Plus sécurisé qu'IKEv1 |
| Groupe DH | Group 14 (2048 bits) | Bon compromis sécurité/perf |
| PFS | PFS2048 | Perfect Forward Secrecy activé |
| Authentification | EAP-TLS (Certificats PKI) | Plus sûr que PSK |

### Durée de vie des SA (Security Associations)

```
Phase 1 (IKE SA)   : 86400 secondes (24h)
Phase 2 (IPsec SA) : 28800 secondes (8h)
```

### Protocoles désactivés

- ❌ PPTP — non chiffré, obsolète
- ❌ L2TP sans IPsec — non sécurisé
- ❌ MD5 — vulnérable aux collisions
- ❌ DES / 3DES — algorithmes faibles
- ❌ SHA-1 — déprécié, risque de collision
- ❌ IKEv1 — remplacé par IKEv2

---

## 🗂️ Configuration Active Directory

```
Type            : Nouvelle forêt
Nom de domaine  : mondomaine.local
NetBIOS         : MONDOMAINE
Niveau fonctionnel : Windows Server 2025
DNS intégré     : Oui
Catalogue global : Oui (sur le DC)
```

**Étapes clés :**
1. Installation du rôle AD DS via le Gestionnaire de serveur
2. Promotion en contrôleur de domaine → "Ajouter une nouvelle forêt"
3. Définition du mot de passe DSRM (mode restauration)
4. Redémarrage et vérification via `dcdiag`

---

## 🔐 Configuration PKI / CA

```
Type CA         : Enterprise Root CA
Nom CA          : MonDomaine-CA
Algorithme clé  : RSA 2048 bits
Hachage         : SHA-256
Validité CA     : 5 ans
```

**Modèle de certificat VPN-IKEv2 :**

```
Nom du modèle   : VPN-IKEv2
Usage           : Authentification serveur + client
Exportable      : Oui (clé privée exportable)
Fournisseur     : Microsoft RSA SChannel Cryptographic Provider
Longueur clé    : 2048 bits
Période validité: 1 an
Auto-enrollment : Activé
```

**Points importants :**
- Le certificat doit contenir le **FQDN du serveur** dans le Subject Alternative Name
- La CA doit être accessible depuis le client pour la validation de la chaîne
- Le certificat racine de la CA doit être installé dans **Trusted Root CA** du client

---

## 🌐 Configuration RRAS

```
Rôle installé   : Remote Access (VPN only)
Type VPN        : IKEv2
Pool d'adresses : 10.0.0.1 — 10.0.0.50
DNS assigné     : 192.168.1.14 (le serveur AD)
Authentification: EAP-TLS (certificat)
Chiffrement     : Maximum (AES-256)
```

**Configuration réseau RRAS :**
- Interface externe : carte réseau connectée au réseau physique
- Interface interne : carte réseau connectée au LAN
- NAT activé si accès Internet requis depuis le client VPN

**Méthode d'authentification configurée :**
```
EAP → Microsoft: Smart Card or other Certificate (EAP-TLS)
✅ Verify the server's identity by validating the certificate
✅ Connect to these servers : mondomaine.local
```

---

## 💻 Configuration Client VPN

**Prérequis côté client :**
- [ ] Certificat racine CA installé dans "Trusted Root Certification Authorities"
- [ ] Certificat client installé dans "Personal"
- [ ] Connexion réseau fonctionnelle vers le serveur (ping 192.168.1.14)

**Paramètres de la connexion VPN :**
```
Nom connexion   : VPN-IPsec-Entreprise
Adresse serveur : 192.168.1.14 (ou FQDN)
Type VPN        : IKEv2
Authentification: Certificat machine
Chiffrement     : Maximum
```

**Commande PowerShell côté client :**
```powershell
Add-VpnConnection `
    -Name "VPN-IPsec-Entreprise" `
    -ServerAddress "192.168.1.14" `
    -TunnelType IKEv2 `
    -AuthenticationMethod MachineCertificate `
    -EncryptionLevel Maximum `
    -RememberCredential $true
```

---

## 🔌 Ports et Protocoles

| Port / Protocole | Usage | Direction |
|------------------|-------|-----------|
| UDP 500 | IKE (négociation SA) | Entrée + Sortie |
| UDP 4500 | NAT-T (IPsec derrière NAT) | Entrée + Sortie |
| IP Protocol 50 | ESP (données chiffrées) | Entrée + Sortie |
| IP Protocol 51 | AH (authentification) | Entrée + Sortie |
| TCP 443 | SSTP (alternatif) | Entrée |

> ⚠️ Ces ports doivent être ouverts dans le **Firewall Windows** du serveur ET éventuellement dans le routeur/pare-feu physique.

---

## ⚠️ Problèmes rencontrés

### 1. Échec de connexion VPN — "Authentication failed"
```
Cause    : Certificat client non reconnu par le serveur
Solution : Réexporter le certificat client depuis la CA
           et le réinstaller dans Cert:\LocalMachine\My
```

### 2. Erreur "The network connection between your computer and the VPN server could not be established"
```
Cause    : Ports UDP 500/4500 bloqués par le firewall
Solution : Créer les règles entrantes dans Windows Defender Firewall
           netsh advfirewall firewall add rule name="IKEv2" 
           protocol=UDP localport=500 dir=in action=allow
```

### 3. Certificat non affiché lors de la connexion client
```
Cause    : Certificat installé dans le mauvais magasin
Solution : Importer dans Cert:\LocalMachine\My 
           (pas CurrentUser)
```

### 4. Paquets ICMP perdus entre les VMs
```
Cause    : Windows Firewall bloque les pings par défaut
Solution : netsh advfirewall firewall add rule 
           name="ICMP Allow" protocol=icmpv4:8,any 
           dir=in action=allow
```

---

## ✅ Bonnes pratiques

**Sécurité :**
- Toujours utiliser **AES-256 + SHA-384** minimum
- Activer le **Perfect Forward Secrecy (PFS)**
- Renouveler les certificats avant expiration
- Désactiver tous les protocoles obsolètes (PPTP, L2TP, MD5, SHA-1)
- Activer l'**audit des connexions VPN** dans les GPO

**Performance :**
- Ajuster le **MTU à 1400** pour éviter la fragmentation IPsec
- Activer le **Hardware Offloading (AES-NI)** si disponible
- Limiter le pool d'adresses VPN au nombre d'utilisateurs réels

**Maintenance :**
- Surveiller les logs RRAS : `Observateur d'événements → System → RemoteAccess`
- Vérifier régulièrement l'expiration des certificats
- Faire des snapshots VMware avant toute modification majeure

---

> 📚 *Notes rédigées dans le cadre du projet VPN IPsec — Formation Administration Systèmes & Sécurité Réseau*
