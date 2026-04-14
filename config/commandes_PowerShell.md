# ⚡ Commandes PowerShell — VPN IPsec Windows Server 2025

> Ensemble des commandes PowerShell utilisées lors de la mise en place du VPN IPsec/IKEv2 sur Windows Server 2025.

---

## 📋 Table des matières

- [Active Directory](#-active-directory)
- [Autorité de Certification (PKI)](#-autorité-de-certification-pki)
- [Remote Access / RRAS](#-remote-access--rras)
- [Configuration IPsec](#-configuration-ipsec)
- [Certificats](#-certificats)
- [Firewall](#-firewall)
- [Vérification et Monitoring](#-vérification-et-monitoring)

---

## 🗂️ Active Directory

```powershell
# Installation du rôle AD DS
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Promotion du serveur en contrôleur de domaine (nouvelle forêt)
Install-ADDSForest `
    -DomainName "mondomaine.local" `
    -DomainNetbiosName "MONDOMAINE" `
    -InstallDns `
    -Force

# Vérifier l'état d'AD
Get-ADDomainController

# Créer un utilisateur VPN
New-ADUser `
    -Name "VPN-User" `
    -SamAccountName "vpnuser" `
    -AccountPassword (ConvertTo-SecureString "MotDePasse123!" -AsPlainText -Force) `
    -Enabled $true
```

---

## 🔐 Autorité de Certification (PKI)

```powershell
# Installation du rôle AD CS
Install-WindowsFeature -Name ADCS-Cert-Authority -IncludeManagementTools

# Configuration de la CA racine d'entreprise
Install-AdcsCertificationAuthority `
    -CAType EnterpriseRootCa `
    -CryptoProviderName "RSA#Microsoft Software Key Storage Provider" `
    -KeyLength 2048 `
    -HashAlgorithmName SHA256 `
    -CACommonName "MonDomaine-CA" `
    -Force

# Publier un modèle de certificat
Add-CATemplate -TemplateName "VPN-IKEv2"

# Vérifier les templates disponibles
Get-CATemplate
```

---

## 🌐 Remote Access / RRAS

```powershell
# Installation du rôle Remote Access
Install-WindowsFeature DirectAccess-VPN -IncludeManagementTools

# Installation de RRAS uniquement (sans DirectAccess)
Install-RemoteAccess -VpnType VPN

# Vérifier l'état du service RRAS
Get-RemoteAccess

# Démarrer le service RRAS
Start-Service RemoteAccess

# Activer RRAS au démarrage
Set-Service RemoteAccess -StartupType Automatic
```

---

## 🔒 Configuration IPsec / IKEv2

```powershell
# Configurer les paramètres IPsec du serveur VPN
Set-VpnServerConfiguration `
    -TunnelType IKEv2 `
    -EncryptionType MaximumEncryption

# Vérifier la configuration VPN serveur
Get-VpnServerConfiguration

# Configurer les algorithmes cryptographiques IKEv2
Set-VpnServerIPsecConfiguration `
    -EncryptionMethod AES256 `
    -IntegrityCheckMethod SHA384 `
    -DHGroup Group14 `
    -PfsGroup PFS2048 `
    -SALifeTimeSeconds 28800 `
    -MMSALifeTimeSeconds 86400

# Désactiver les protocoles VPN obsolètes (PPTP, L2TP)
Set-VpnServerConfiguration -PassThru | 
    Set-VpnServerConfiguration -TunnelType IKEv2
```

---

## 📜 Certificats

```powershell
# Ouvrir la console MMC des certificats
mmc

# Lister les certificats installés sur le serveur
Get-ChildItem -Path Cert:\LocalMachine\My

# Importer un certificat .pfx côté client
Import-PfxCertificate `
    -FilePath "C:\Certificats\client-vpn.pfx" `
    -CertStoreLocation Cert:\LocalMachine\My `
    -Password (ConvertTo-SecureString "MotDePasse" -AsPlainText -Force)

# Exporter un certificat (sans clé privée)
Export-Certificate `
    -Cert (Get-ChildItem -Path Cert:\LocalMachine\My | Where-Object {$_.Subject -like "*VPN*"}) `
    -FilePath "C:\Certificats\vpn-cert.cer"

# Vérifier l'expiration des certificats
Get-ChildItem -Path Cert:\LocalMachine\My | 
    Select-Object Subject, NotAfter | 
    Sort-Object NotAfter
```

---

## 🛡️ Firewall

```powershell
# Ouvrir les ports nécessaires pour IPsec/IKEv2
New-NetFirewallRule -DisplayName "IKEv2 VPN UDP 500" `
    -Direction Inbound -Protocol UDP -LocalPort 500 -Action Allow

New-NetFirewallRule -DisplayName "IKEv2 VPN UDP 4500" `
    -Direction Inbound -Protocol UDP -LocalPort 4500 -Action Allow

New-NetFirewallRule -DisplayName "IPsec ESP" `
    -Direction Inbound -Protocol 50 -Action Allow

# Autoriser le ping ICMP (pour les tests)
netsh advfirewall firewall add rule name="ICMP Allow" `
    protocol=icmpv4:8,any dir=in action=allow

# Vérifier les règles firewall VPN
Get-NetFirewallRule | Where-Object {$_.DisplayName -like "*VPN*"}
```

---

## 📊 Vérification et Monitoring

```powershell
# Voir les connexions VPN actives
Get-VpnServerConnection

# Voir les logs d'événements VPN
Get-EventLog -LogName System -Source RemoteAccess -Newest 20

# Vérifier la connectivité réseau
Test-NetConnection -ComputerName 192.168.1.14 -Port 500

# Statistiques IPsec
Get-NetIPsecMainModeSA
Get-NetIPsecQuickModeSA

# MTU — Réglage pour éviter la fragmentation
netsh interface ipv4 set subinterface "Ethernet" mtu=1400 store=persistent

# Vérifier l'état de RRAS
Get-Service RemoteAccess | Select-Object Name, Status, StartType
```

---

> 💡 **Conseil :** Toujours exécuter PowerShell en tant qu'**Administrateur** pour ces commandes.
