# 🔍 Laboratoire 01 — Analyse du trafic DNS avec Wireshark

## 🎯 Objectif
Analyser la séquence de résolution de noms DNS, l'encapsulation des paquets dans la pile OSI (Ethernet II, IPv4, UDP, DNS) et l'interaction avec une infrastructure de protection Web (WAF) à l'aide de Wireshark.

---

## 🛠️ Outils & Environnement
- **Analyseur de paquets :** Wireshark
- **Ligne de commande :** Windows CMD (`ipconfig /flushdns`, `ipconfig /flushdns`, `nslookup`)
- **Cible de test :** `www.polymtl.ca`

---

## 📝 Démarche expérimentale & Observations

### 1. Déclenchement de la requête DNS (Terminal)
Exécution de la commande `nslookup www.polymtl.ca` pour forcer la résolution du nom de domaine.

![Commande nslookup](./nslookup-cmd.png)

* **Serveur DNS interrogé :** `209.197.128.2` (`dns1.distributel.net`)
* **Redirection (CNAME) :** `qlpbaqf.ng.impervadns.net`
* **Adresse IP retournée :** `45.60.105.40`

---

### 2. Capture et analyse dans Wireshark
Application du filtre `udp.port == 53` pour isoler les paquets de résolution de noms.

![Analyse DNS Wireshark](./dns-analysis.png)

#### Analyse de la pile d'encapsulation (Réponse `0x0002`) :
- **Ethernet II :** Adresses MAC de la carte réseau locale et du routeur/passerelle.
- **IPv4 :** Source `209.197.128.2` (Serveur DNS) $\rightarrow$ Destination `192.168.68.105` (Hôte).
- **UDP :** Port source `53` (DNS) $\rightarrow$ Port destination dynamique `59268`.
- **DNS Protocol :**
  - **Queries :** Question posée sur l'enregistrement `A` pour `www.polymtl.ca`.
  - **Answers :** Retour d'un enregistrement CNAME (`impervadns.net`) et de l'enregistrement A (`45.60.105.40`).

---

## 🛡️ Perspective Analyste SOC
- **Protection Cloud & WAF :** L'existence de l'alias CNAME vers Imperva démontre l'utilisation d'un WAF cloud / service anti-DDoS pour protéger le serveur d'origine de Polytechnique Montréal.
- **Surveillance DNS :** L'analyse des flux UDP/53 est essentielle en SOC pour identifier la fuite de données (DNS Tunneling), les redirections malveillantes (DNS Poisoning) ou l'activité de botnets (DGA).
