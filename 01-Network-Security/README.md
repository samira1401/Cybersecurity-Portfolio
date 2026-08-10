
# 🌐 01 - Network Security

Ce dossier regroupe les projets et laboratoires pratiques liés à la sécurité des réseaux, l'analyse de trafic et l'inspection de paquets.

## 📂 Laboratoires inclus

| Projet | Description | Statut |
| :--- | :--- | :---: |
| **[01-DNS-Traffic-Analysis-Wireshark](./01-DNS-Traffic-Analysis-Wireshark)** | Analyse du trafic DNS, de la résolution de noms et de l'encapsulation de paquets avec Wireshark | ✅ *Terminé* |
| **[02-HTTP-Traffic-Analysis-Wireshark](./02-HTTP-Traffic-Analysis-Wireshark)** | Inspection du trafic Web clair (HTTP/80) et détection de formulaires/identifiants non chiffrés | ✅ *Terminé* |
| **[03-FTP-Security-Analysis-Wireshark](./03-FTP-Security-Analysis-Wireshark)** | Analyse de session FTP (Port 21) et interception d'identifiants en clair | ✅ *Terminé* |
| **[04-SMTP-Mail-Analysis-Wireshark](./04-SMTP-Mail-Analysis-Wireshark)** | Capture et analyse de flux d'envoi de courriels non sécurisés (Port 25) | ✅ *Terminé* |
| **[05-POP3-Session-Analysis-Wireshark](./05-POP3-Session-Analysis-Wireshark)** | Audit de récupération de courriels via POP3 (Port 110) et extraction de requêtes | ✅ *Terminé* |
| **[06-IMAP-Header-Analysis-Wireshark](./06-IMAP-Header-Analysis-Wireshark)** | Analyse forensic des commandes IMAP (Port 143), extraction d'en-têtes et métadonnées | ✅ *Terminé* |




---

## 🛠️ Protocoles Analysés & Risques Identifiés

| Protocole | Port par défaut | Risque majeur identifié | Statut Sécurité | Alternative Securisée |
| :--- | :---: | :--- | :---: | :---: |
| **HTTP** | 80 | Mots de passe, cookies et formulaires transmis en clair | 🔴 Critique | **HTTPS** (443) |
| **FTP** | 21 | Identifiants (`USER` / `PASS`) et fichiers interceptables | 🔴 Critique | **SFTP** (22) / **FTPS** (990) |
| **SMTP** | 25 | Corps des courriels et authentification non chiffrés | 🟠 Élevé | **SMTPS** (465) / **STARTTLS** |
| **POP3** | 110 | Extraction de courriels et identifiants en clair | 🟠 Élevé | **POP3S** (995) |
| **IMAP** | 143 | Commandes `FETCH`, entêtes et identifiants lisibles | 🟠 Élevé | **IMAPS** (993) |
| **DNS** | 53 | Risque d'usurpation (Spoofing) / Tracking de requêtes | 🟡 Moyen | **DoH** / **DoT** / **DNSSEC** |

---
## 🎯 Filtres de Recherche Avancés (Wireshark Display Filters)

Voici une sélection de filtres de recherche utilisés lors des investigations :

| Objectif d'investigation | Filtre Wireshark |
| :--- | :--- |
| **Identifiants en clair (IMAP/POP3/FTP)** | `imap.request` ou `pop.request.command == "PASS"` ou `ftp.request.command == "PASS"` |
| **Requêtes HTTP POST (Formulaires)** | `http.request.method == "POST"` |
| **Détection de balayage de ports (SYN Scan)** | `tcp.flags.syn == 1 && tcp.flags.ack == 0` |
| **Erreurs de résolution DNS** | `dns.flags.rcode != 0` |
| **Extraction par Socket (IP + Port)** | `ip.addr == 192.168.0.4 && tcp.port == 143` |

------
## 🧩 Cartographie MITRE ATT&CK

| Tactique MITRE | Technique | Application dans ce projet |
| :--- | :--- | :--- |
| **Credential Access (TA0006)** | Network Sniffing (`T1040`) | Interception d'identifiants transmis en clair sur FTP, POP3, IMAP, HTTP |
| **Reconnaissance (TA043)** | Active Scanning (`T1595`) | Détection d'analyse de ports et requêtes DNS volumineuses |
| **Discovery (TA0007)** | Network Service Discovery (`T1046`) | Identification de services non sécurisés exposés sur les ports standards (21, 25, 80, 110, 143) |

------
## 🚨 Détection Automatisée (Règles IDS Snort / Suricata)

Exemple de règle pour détecter l'envoi d'identifiants FTP non chiffrés sur le réseau :

```snort
alert tcp $HOME_NET any ->$EXTERNAL_NET 21 (msg:"SEC-SOC - Transmission d'identifiants FTP non chiffrés"; flow:to_server,established; content:"PASS"; nocase; sid:1000001; rev:1;)

## 🛡️ Recommandations & Plan d'Action (Mitigation)
1. **Chiffrement Généralisé (TLS/SSL) :** Interdire l'usage des protocoles hérités en clair au profit de leurs équivalents chiffrés (HTTPS, IMAPS, POP3S, SMTPS avec TLS 1.3).






## 📂 Laboratoires inclus

| Projet | Description | Statut |
| :--- | :--- | :---: |
| **[01-DNS-Traffic-Analysis-Wireshark](./01-DNS-Traffic-Analysis-Wireshark)** | Analyse du trafic DNS, de la résolution de noms et de l'encapsulation de paquets avec Wireshark | ✅ *Terminé* |
| **[02-HTTP-Traffic-Analysis-Wireshark](./02-HTTP-Traffic-Analysis-Wireshark)** | Inspection du trafic Web clair (HTTP/80) et détection de formulaires/identifiants non chiffrés | ✅ *Terminé* |
| **[03-FTP-Security-Analysis-Wireshark](./03-FTP-Security-Analysis-Wireshark)** | Analyse de session FTP (Port 21) et interception d'identifiants en clair | ✅ *Terminé* |
| **[04-SMTP-Mail-Analysis-Wireshark](./04-SMTP-Mail-Analysis-Wireshark)** | Capture et analyse de flux d'envoi de courriels non sécurisés (Port 25) | ✅ *Terminé* |
| **[05-POP3-Session-Analysis-Wireshark](./05-POP3-Session-Analysis-Wireshark)** | Audit de récupération de courriels via POP3 (Port 110) et extraction de requêtes | ✅ *Terminé* |
| **[06-IMAP-Header-Analysis-Wireshark](./06-IMAP-Header-Analysis-Wireshark)** | Analyse forensic des commandes IMAP (Port 143), extraction d'en-têtes et métadonnées | ✅ *Terminé* |

---

## 🛠️ Protocoles Analysés & Risques Identifiés

| Protocole | Port par défaut | Risque majeur identifié | Statut Sécurité | Alternative Sécurisée |
| :--- | :---: | :--- | :---: | :---: |
| **HTTP** | 80 | Mots de passe, cookies et formulaires transmis en clair | 🔴 Critique | **HTTPS** (443) |
| **FTP** | 21 | Identifiants (`USER` / `PASS`) et fichiers interceptables | 🔴 Critique | **SFTP** (22) / **FTPS** (990) |
| **SMTP** | 25 | Corps des courriels et authentification non chiffrés | 🟠 Élevé | **SMTPS** (465) / **STARTTLS** |
| **POP3** | 110 | Extraction de courriels et identifiants en clair | 🟠 Élevé | **POP3S** (995) |
| **IMAP** | 143 | Commandes `FETCH`, entêtes et identifiants lisibles | 🟠 Élevé | **IMAPS** (993) |
| **DNS** | 53 | Risque d'usurpation (Spoofing) / Tracking de requêtes | 🟡 Moyen | **DoH** / **DoT** / **DNSSEC** |

---

## 🧩 Cartographie MITRE ATT&CK

| Tactique MITRE | Technique | Application dans ce projet |
| :--- | :--- | :--- |
| **Credential Access (TA0006)** | Network Sniffing (`T1040`) | Interception d'identifiants transmis en clair sur FTP, POP3, IMAP, HTTP |
| **Reconnaissance (TA043)** | Active Scanning (`T1595`) | Détection d'analyse de ports et requêtes DNS volumineuses |
| **Discovery (TA0007)** | Network Service Discovery (`T1046`) | Identification de services non sécurisés exposés sur les ports standards |

---

## 🎯 Filtres de Recherche Avancés (Wireshark Display Filters)

| Objectif d'investigation | Filtre Wireshark |
| :--- | :--- |
| **Identifiants en clair (IMAP/POP3/FTP)** | `imap.request` ou `pop.request.command == "PASS"` ou `ftp.request.command == "PASS"` |
| **Requêtes HTTP POST (Formulaires)** | `http.request.method == "POST"` |
| **Détection de balayage de ports (SYN Scan)** | `tcp.flags.syn == 1 && tcp.flags.ack == 0` |
| **Erreurs de résolution DNS** | `dns.flags.rcode != 0` |

---

## 🚨 Détection Automatisée (Exemple Règle Snort)

```snort
# Détection de transmission de mot de passe FTP en clair
alert tcp $HOME_NET any ->$EXTERNAL_NET 21 (msg:"SEC-SOC - Transmission d'identifiants FTP non chiffrés"; flow:to_server,established; content:"PASS"; nocase; sid:1000001; rev:1;)
2. **Transfert de Fichiers Sécurisé :** Remplacer FTP par SSH File Transfer Protocol (SFTP) fondé sur SSH.
3. **Segmentation & Surveillance :** Mettre en place un IDS/IPS (ex: Snort/Suricata) pour détecter les flux non chiffrés suspects traversant le réseau d'entreprise.
