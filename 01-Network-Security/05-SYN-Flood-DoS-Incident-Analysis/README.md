# 🔐 Analyse d'un incident SYN Flood / DoS

## 📌 Présentation du projet

Ce projet présente l'analyse d'un incident de cybersécurité affectant un serveur web.

L'objectif est d'analyser des journaux de trafic réseau afin d'identifier la cause probable d'une interruption de service, de comprendre le fonctionnement de l'attaque et d'évaluer son impact sur les utilisateurs légitimes.

L'analyse porte sur une **attaque TCP SYN Flood**, un type d'attaque par **déni de service (DoS)**.

> ⚠️ Ce projet repose sur un scénario de cybersécurité simulé à des fins pédagogiques et professionnelles. Aucun système réel n'a été attaqué.

---

## 🎯 Objectifs

Ce projet a pour objectifs de :

* analyser du trafic réseau TCP ;
* comprendre le TCP Three-Way Handshake ;
* identifier des requêtes SYN anormales ;
* analyser des journaux réseau de type Wireshark ;
* identifier le type d'attaque ;
* analyser l'impact sur le serveur web ;
* analyser l'impact sur les utilisateurs légitimes ;
* proposer des mesures de protection.

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
443
```

L'adresse IP suspecte continue d'envoyer des requêtes SYN à un rythme élevé.

Au fur et à mesure que l'attaque progresse, les connexions des utilisateurs légitimes commencent à échouer.

Les journaux montrent notamment :

* des requêtes SYN répétées ;
* des connexions légitimes qui échouent ;
* des paquets `[RST, ACK]` ;
* des erreurs `HTTP/1.1 504 Gateway Time-out` ;
* une disparition progressive du trafic légitime.

Ces éléments sont compatibles avec une **attaque TCP SYN Flood**.

---

## 🚨 Type d'attaque identifié

**TCP SYN Flood — Déni de service direct (DoS)**

Dans ce scénario, le trafic malveillant provient d'une seule adresse IP.

L'incident est donc identifié comme un **DoS direct**, plutôt qu'un DDoS.

---

## 🤝 TCP Three-Way Handshake

Une connexion TCP normale s'établit en trois étapes :

```text
1. SYN
Client → Serveur

2. SYN/ACK
Client ← Serveur

3. ACK
Client → Serveur
```

Après l'établissement de la connexion TCP, le navigateur peut envoyer une requête HTTP telle que :

```text
GET /sales.html HTTP/1.1
```

Le serveur peut alors répondre :

```text
HTTP/1.1 200 OK
```

---

## 💥 Impact

L'augmentation importante des requêtes SYN entraîne une forte sollicitation des ressources du serveur.

Les utilisateurs légitimes peuvent alors rencontrer :

* des délais d'attente ;
* des connexions interrompues ;
* des erreurs HTTP ;
* des difficultés à accéder au site web.

À partir de l'entrée de journal **125**, le trafic légitime disparaît pratiquement du journal et les requêtes de l'attaque deviennent prédominantes.

---

## 🛡️ Mesures de protection

Plusieurs mesures peuvent contribuer à réduire l'impact d'une attaque SYN Flood :

* utiliser des **SYN Cookies** ;
* mettre en place du **rate limiting** ;
* renforcer les règles du pare-feu ;
* utiliser des solutions **IDS/IPS** ;
* mettre en place une protection contre les attaques DDoS ;
* surveiller les volumes de trafic TCP ;
* configurer des alertes sur les comportements anormaux.

Le simple blocage d'une adresse IP ne constitue pas une solution complète, car un attaquant peut utiliser d'autres adresses.

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
Interruption du service
```

Cette investigation m'a permis de développer mes compétences en analyse de trafic réseau, analyse de logs et identification d'incidents de sécurité.

---

## 🛠️ Technologies et compétences

### Technologies

* Wireshark
* TCP/IP
* TCP
* HTTP/HTTPS

### Compétences

* Analyse de trafic réseau
* Analyse de journaux
* TCP Three-Way Handshake
* Identification d'une attaque DoS
* Analyse SYN Flood
* Investigation d'incident
* Analyse des impacts
* Mesures de mitigation

---

## ⚠️ Avertissement

Ce projet est réalisé à des fins pédagogiques et professionnelles dans le cadre de mon portfolio de cybersécurité.

Les adresses IP et les données réseau utilisées appartiennent à un scénario simulé.

Aucun système réel n'a été ciblé.

---

## 👤 Auteur

**[Samira.Bel]**

Portfolio Cybersécurité
