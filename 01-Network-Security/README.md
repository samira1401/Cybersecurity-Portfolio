# 🌐 01 - Network Security

Ce dossier regroupe les projets et laboratoires pratiques liés à la sécurité des réseaux, l'analyse de trafic et l'inspection de paquets.

## 📂 Laboratoires inclus

| Projet | Description | Statut |
| :--- | :--- | :---: |
| **[01-DNS-Traffic-Analysis-Wireshark](./01-DNS-Traffic-Analysis-Wireshark)** | Analyse du trafic DNS, de la résolution de noms et de l'encapsulation de paquets avec Wireshark | ✅ *Complété* |




# 🔍 Projet 1 : Analyse de Trafic Réseau & Identification de Vulnérabilités (Wireshark)

## 📌 Objectif du Projet
L'objectif de cette étude est d'analyser des captures de trafic réseau (`.pcapng`) représentant divers protocoles fondamentaux (FTP, HTTP, DNS, SMTP, POP3, IMAP) afin d'évaluer leur niveau de sécurité, d'identifier la transmission de données sensibles en clair et de proposer des mesures de mitigation (durcissement).

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

## 🔬 Faits Marquants de l'Analyse (Highlights)

### 1. Interception d'identifiants (FTP / HTTP / Mail)
* **Constat :** L'utilisation des filtres Wireshark `http.request.method == "POST"`, `ftp.request.command == "PASS"`, ou `imap` permet la lecture directe des identifiants et des jetons de session sans déchiffrement.
* **Preuve :** *(Ajoutez ici le chemin vers votre capture d'écran, ex: `./screenshots/imap-clear-text.png`)*

### 2. Inspection des entêtes de courriels (IMAP / SMTP)
* **Constat :** Reconstitution réussie des échanges d'e-mails et extraction des entêtes techniques (`Subject`, `From`, `To`, `Date`) révélant la topologie du réseau et les adresses IP émettrices.

---

## 🛡️ Recommandations & Plan d'Action (Mitigation)
1. **Chiffrement Généralisé (TLS/SSL) :** Interdire l'usage des protocoles hérités en clair au profit de leurs équivalents chiffrés (HTTPS, IMAPS, POP3S, SMTPS avec TLS 1.3).
2. **Transfert de Fichiers Sécurisé :** Remplacer FTP par SSH File Transfer Protocol (SFTP) fondé sur SSH.
3. **Segmentation & Surveillance :** Mettre en place un IDS/IPS (ex: Snort/Suricata) pour détecter les flux non chiffrés suspects traversant le réseau d'entreprise.
