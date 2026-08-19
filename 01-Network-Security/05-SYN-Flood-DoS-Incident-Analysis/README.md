# 🔐 Analyse d'un incident SYN Flood / DoS

## 📌 Présentation du projet

Ce projet présente l'analyse d'un incident de cybersécurité affectant un serveur web.

L'objectif est d'analyser des journaux de trafic réseau afin d'identifier la cause probable d'une interruption de service, de comprendre le fonctionnement de l'attaque et d'évaluer son impact sur les utilisateurs légitimes.

L'analyse porte sur une **attaque TCP SYN Flood**, un type d'attaque par **déni de service (DoS)**.

> ⚠️ **Note :** Ce projet repose sur un scénario de cybersécurité simulé à des fins pédagogiques et professionnelles. Aucun système réel n'a été attaqué.

---

## 🎯 Objectifs

Ce projet a pour objectifs de :

* Analyser le trafic réseau TCP
* Comprendre le TCP Three-Way Handshake
* Identifier les requêtes SYN anormales
* Analyser les journaux réseau issus de Wireshark
* Identifier le type d'attaque
* Analyser l'impact sur le serveur web
* Analyser l'impact sur les utilisateurs légitimes
* Proposer des mesures de protection

---

## 🏢 Scénario

Une entreprise utilise un serveur web pour héberger une page de ventes accessible par ses employés.

Un problème est détecté lorsque les utilisateurs commencent à recevoir des **erreurs de dépassement du délai de connexion (timeout)**.

L'analyse du trafic réseau révèle un nombre anormalement élevé de requêtes TCP SYN provenant d'une adresse IP inconnue.

Le serveur semble progressivement incapable de répondre correctement aux demandes des utilisateurs légitimes.

---

## 🔎 Analyse

L'analyse des journaux montre de nombreuses requêtes TCP SYN provenant de l'adresse :

```text
203.0.113.0
```

vers le serveur web :

```text
192.0.2.1
```

sur le port :

```text
TCP/443 — HTTPS
```

L'adresse IP suspecte continue d'envoyer des requêtes SYN à un rythme élevé.

Au fur et à mesure que l'attaque progresse, les connexions des utilisateurs légitimes commencent à échouer.

Les journaux montrent notamment :

* Des requêtes SYN répétées
* Des connexions légitimes qui échouent
* Des paquets `[RST, ACK]`
* Des erreurs `HTTP/1.1 504 Gateway Time-out`
* Une disparition progressive du trafic légitime

Ces éléments sont compatibles avec une **attaque TCP SYN Flood**.

---
## 📸 Preuves visuelles

### 1. Trafic normal

![Trafic TCP normal](screenshots/01-trafic-normal.png)

> **Trafic normal — TCP Three-Way Handshake et requête HTTP réussie.**
> La connexion TCP est établie normalement et le serveur répond avec `HTTP/1.1 200 OK`.

### 2. Trafic SYN Flood

![Trafic SYN Flood](screenshots/02-syn-flood.png)

> **Trafic suspect — augmentation importante des requêtes TCP SYN.**
> Une même source génère de nombreuses tentatives de connexion vers le serveur HTTPS.

### 3. Dégradation du service

![Dégradation du service](screenshots/03-degradation-service.png)

> **Dégradation du service — échecs de connexions et erreur HTTP 504.**
> Des réinitialisations TCP et des erreurs de délai d'attente apparaissent pendant l'augmentation du trafic SYN.

---

## 🚨 Type d'attaque identifié

### **TCP SYN Flood — Déni de service direct (DoS)**

Dans ce scénario, le trafic malveillant observé provient d'une seule adresse IP.

L'incident est donc identifié comme un **DoS direct**, plutôt qu'un DDoS.

> **Remarque :** Cette classification est basée sur les données du scénario analysé. La présence d'une seule source observée dans le journal ne permet pas nécessairement d'exclure l'utilisation d'autres sources non visibles dans la capture.

---

## 🤝 Protocole TCP à trois voies

Une connexion TCP normale s'établit en trois étapes :

```text
Client                 Serveur
  │                       │
  │────── SYN ──────────►│
  │                       │
  │◄──── SYN/ACK ────────│
  │                       │
  │────── ACK ──────────►│
  │                       │
  ▼                       ▼
Connexion TCP établie
```

Après l'établissement de la connexion TCP, le navigateur peut envoyer une requête HTTP telle que :

```http
GET /sales.html HTTP/1.1
```

Le serveur peut alors répondre :

```http
HTTP/1.1 200 OK
```

Cette séquence représente le fonctionnement normal d'une connexion TCP suivie d'une communication HTTP.

---

## 💥 Impact

L'augmentation importante des requêtes SYN entraîne une forte sollicitation des ressources utilisées par le serveur pour gérer les connexions TCP.

Les utilisateurs légitimes peuvent alors rencontrer :

* Des délais d'attente
* Des connexions interrompues
* Des erreurs HTTP
* Des difficultés à accéder au site web

À partir de l'entrée **125**, le trafic légitime disparaît pratiquement du journal et les requêtes associées à l'activité suspecte deviennent prédominantes.

L'impact principal observé est donc une **perte progressive de disponibilité du service web**.

---

## 🛡️ Mesures de protection

Plusieurs mesures peuvent contribuer à réduire l'impact d'une attaque SYN Flood :

* Utiliser des **SYN Cookies**
* Mettre en place une **limitation du débit (rate limiting)**
* Renforcer les règles du pare-feu
* Utiliser des solutions **IDS/IPS**
* Mettre en place une protection contre les attaques DDoS
* Surveiller les volumes de trafic TCP
* Configurer des alertes sur les comportements anormaux
* Mettre en place une procédure de réponse à incident

Le simple blocage d'une adresse IP ne constitue pas une solution complète, car un attaquant peut utiliser d'autres adresses ou des adresses usurpées.

La meilleure approche repose donc sur une **défense en profondeur**, combinant plusieurs mécanismes de détection, de filtrage et de protection.

Pour plus de détails, voir :

`mitigation.md`

---

## 🧠 Ce que j'ai appris

Ce projet m'a permis de comprendre comment relier les événements observés dans les journaux réseau aux conséquences visibles par les utilisateurs.

L'analyse permet de suivre la progression de l'incident :

```text
Requêtes SYN répétées
        ↓
Trafic anormal
        ↓
Pression sur les ressources du serveur
        ↓
Échecs des connexions légitimes
        ↓
RST/ACK et erreurs 504
        ↓
Dégradation du service
        ↓
Perte de disponibilité
```

Cette enquête m'a permis de développer mes compétences en :

* Analyse de trafic réseau
* Analyse de logs
* Identification d'incidents de sécurité
* Analyse TCP/IP
* Analyse d'une attaque DoS
* Investigation réseau
* Évaluation de l'impact
* Proposition de mesures d'atténuation

---

## 🛠️ Technologies et compétences

### Technologies

* **Wireshark**
* **TCP/IP**
* **TCP**
* **HTTP/HTTPS**

### Compétences

* Analyse de trafic réseau
* Analyse de journaux
* Compréhension du TCP Three-Way Handshake
* Identification d'une attaque DoS
* Analyse d'une attaque SYN Flood
* Investigation d'incident
* Analyse des impacts
* Mesures d'atténuation

---

## 📂 Documentation du projet

Les différentes étapes de l'analyse sont documentées dans les fichiers suivants :

```text
evidence/
└── Wireshark-TCP-HTTP-Log.xlsx

rapport-incident.md
analyse-wireshark.md
chronologie-attaque.md
mitigation.md
```

### Documents associés

* **`rapport-incident.md`** — Présentation générale de l'incident
* **`analyse-wireshark.md`** — Analyse détaillée du trafic réseau
* **`chronologie-attaque.md`** — Reconstitution chronologique de l'attaque
* **`mitigation.md`** — Mesures de détection, mitigation et réponse à incident
* **`evidence/Wireshark-TCP-HTTP-Log.xlsx`** — Données réseau utilisées pour l'analyse

---

## ⚠️ Avertissement

Ce projet est réalisé à des fins pédagogiques et professionnelles dans le cadre de mon portfolio de cybersécurité.

Les adresses IP et les données réseau utilisées appartiennent à un scénario simulé.

Aucun système réel n'a été ciblé ou attaqué.

---

## 👤 Auteur

**Samira Bel**

Portfolio Cybersécurité
