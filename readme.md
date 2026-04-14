# 🔐 VPN IPsec sous Windows Server 2025

## 📌 Objectif
Mettre en place un VPN sécurisé permettant à des utilisateurs distants d’accéder au réseau interne via IPsec.

---

## 🧱 Architecture

- 1 serveur Windows Server 2025 (RRAS)
- 1 client Windows
- Authentification par certificat (PKI)

---

## ⚙️ Technologies utilisées

- Windows Server 2025
- IPsec / IKEv2
- Active Directory
- Certificate Authority (AD CS)
- RRAS (Routing and Remote Access)

---

## 🚀 Étapes principales

1. Installation de Active Directory
2. Installation de Certificate Authority (CA)
3. Génération des certificats
4. Installation du rôle Remote Access
5. Configuration du VPN (RRAS)
6. Configuration client VPN
7. Test de connexion

---

## 📸 Résultats

- Tunnel VPN établi
- Communication sécurisée
- Authentification par certificat

---

## ⚠️ Problèmes rencontrés

- Échec de connexion VPN
- Problèmes réseau / filtrage

---

## 🛡️ Sécurité

- Utilisation de AES-256
- SHA-384
- Authentification par certificat
- Désactivation des protocoles faibles

---

## 📈 Améliorations possibles

- Intégration avec Azure VPN
- Automatisation avec PowerShell
- Monitoring avancé

---

## 👨‍💻 Auteur

Adoum Mahamat Abakar
