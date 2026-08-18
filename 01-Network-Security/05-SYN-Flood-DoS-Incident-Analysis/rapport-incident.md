# 🔐 Rapport d'incident de cybersécurité

## 1. Résumé de l'incident

### Description

Une interruption de service a été détectée sur le serveur web de l'organisation.

Les utilisateurs légitimes rencontrent des erreurs de dépassement du délai de connexion lorsqu'ils tentent d'accéder au site web.

Une analyse du trafic réseau a permis d'identifier un volume anormalement élevé de requêtes TCP SYN provenant d'une même adresse IP.

### Type d'incident

**Attaque TCP SYN Flood — Déni de service direct (DoS)**

### Système ciblé

**Serveur web :** `192.0.2.1`

### Adresse IP suspecte

**Source identifiée :** `203.0.113.0`

### Service ciblé

**TCP port 443 — HTTPS**

---

## 2. Identification de l'attaque

L'analyse du journal réseau montre que l'adresse IP `203.0.113.0` envoie de nombreuses requêtes TCP SYN vers le serveur web.

Le comportement devient anormal lorsque les requêtes SYN sont répétées à un rythme élevé.

Le serveur répond initialement aux demandes, mais la quantité croissante de requêtes finit par perturber les connexions des utilisateurs légitimes.

Les éléments observés dans le journal sont notamment :

* répétition importante de paquets SYN ;
* trafic provenant de la même adresse IP ;
* échecs de connexions des utilisateurs légitimes ;
* paquets `[RST, ACK]` ;
* erreurs `HTTP/1.1 504 Gateway Time-out` ;
* disparition progressive du trafic légitime.

Ces indicateurs sont cohérents avec une attaque **SYN Flood**.

---

## 3. Conclusion préliminaire

L'analyse des données disponibles permet d'identifier l'incident comme une **attaque TCP SYN Flood**.

Puisque le trafic malveillant observé provient d'une seule adresse IP dans ce scénario, il s'agit d'un **déni de service direct (DoS)** plutôt que d'un **déni de service distribué (DDoS)**.

---

## 4. Prochaine étape de l'analyse

La prochaine étape consiste à examiner plus précisément les communications TCP afin de comprendre :

* comment le Three-Way Handshake fonctionne normalement ;
* comment l'attaque perturbe ce processus ;
* comment les connexions des utilisateurs légitimes sont affectées ;
* comment le serveur finit par devenir indisponible.
