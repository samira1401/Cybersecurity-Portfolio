# 🔎 Analyse du trafic réseau avec Wireshark

## 1. Objectif de l'analyse

L'objectif de cette analyse est d'examiner le trafic TCP/HTTP enregistré afin d'identifier les communications normales, de détecter les comportements anormaux et de comprendre comment l'attaque affecte le serveur web.

L'analyse porte principalement sur :

* les adresses IP source et destination ;
* le protocole TCP ;
* les paquets SYN, SYN/ACK et ACK ;
* les requêtes HTTP ;
* les erreurs de connexion ;
* l'évolution du trafic pendant l'incident.

---

## 2. Identification des machines

Le journal contient plusieurs adresses IP.

### Serveur web

```text
192.0.2.1
```

Cette adresse IP correspond au serveur web de l'organisation.

### Ordinateurs des employés

Les ordinateurs des employés utilisent des adresses appartenant à la plage :

```text
198.51.100.0/24
```

### Source suspecte

L'adresse identifiée comme source du trafic malveillant est :

```text
203.0.113.0
```

---

## 3. Analyse du trafic TCP normal

Une connexion TCP normale commence par un processus appelé **Three-Way Handshake**.

Dans le journal, on peut observer une séquence normale :

```text
SYN
 ↓
SYN/ACK
 ↓
ACK
```

Par exemple, les entrées 47 à 49 montrent :

```text
47 → SYN
48 → SYN/ACK
49 → ACK
```

La connexion TCP est alors établie.

Le navigateur peut ensuite envoyer une requête HTTP :

```text
GET /sales.html HTTP/1.1
```

Le serveur répond :

```text
HTTP/1.1 200 OK
```

Cette séquence représente un fonctionnement normal du service web.

---

## 4. Début de l'activité suspecte

À partir de l'entrée 52, le trafic provenant de l'adresse :

```text
203.0.113.0
```

devient intéressant.

Les premières communications suivent encore un processus TCP normal :

```text
52 → SYN
53 → SYN/ACK
54 → ACK
```

Cependant, l'adresse IP continue ensuite à envoyer de nouvelles requêtes SYN.

Cette répétition rapide constitue un comportement anormal.

---

## 5. Augmentation des requêtes SYN

Les entrées suivantes montrent que la même adresse IP continue d'envoyer des paquets SYN :

```text
57 → SYN
59 → SYN
61 → SYN
64 → SYN
66 → SYN
68 → SYN
70 → SYN
72 → SYN
74 → SYN
76 → SYN
```

Le serveur reçoit donc de nombreuses demandes de connexion provenant de la même source.

Pendant cette période, les utilisateurs légitimes peuvent encore communiquer avec le serveur.

Par exemple, l'adresse :

```text
198.51.100.14
```

réussit à établir une connexion et à demander :

```text
GET /sales.html HTTP/1.1
```

Le serveur répond avec :

```text
HTTP/1.1 200 OK
```

Cela montre que l'attaque n'a pas encore complètement empêché le fonctionnement du serveur.

---

## 6. Apparition des premières communications échouées

À mesure que les requêtes SYN augmentent, les connexions des utilisateurs légitimes commencent à échouer.

Le journal montre notamment des paquets :

```text
[RST, ACK]
```

Un paquet RST/ACK peut indiquer qu'une connexion TCP est réinitialisée.

Le journal montre également une erreur :

```text
HTTP/1.1 504 Gateway Time-out
```

Cette erreur indique qu'une passerelle a attendu trop longtemps une réponse du serveur.

Ces événements montrent que le serveur commence à avoir des difficultés à traiter normalement les demandes.

---

## 7. Analyse du trafic malveillant

Le comportement de l'adresse `203.0.113.0` devient particulièrement évident dans la suite du journal.

On observe une succession de requêtes :

```text
SYN
SYN
SYN
SYN
SYN
SYN
SYN
SYN
...
```

Les requêtes sont envoyées à intervalles très rapprochés.

Contrairement à une navigation web normale, ce trafic ne montre pas une succession normale de connexions suivies de requêtes HTTP.

Le trafic est principalement constitué de tentatives de connexion TCP.

---

## 8. Dégradation du service

Les communications légitimes deviennent progressivement moins fréquentes et davantage de connexions échouent.

Les principaux indicateurs sont :

```text
SYN répétés
       ↓
Augmentation des demandes de connexion
       ↓
Ressources du serveur sollicitées
       ↓
Échecs de connexions légitimes
       ↓
RST/ACK
       ↓
504 Gateway Time-out
```

Cette progression indique que le serveur rencontre des difficultés à répondre aux utilisateurs légitimes.

---

## 9. Entrée de journal 125 et suivantes

À partir de l'entrée de journal **125**, le trafic observé change de manière importante.

Les entrées sont principalement constituées de paquets SYN provenant de :

```text
203.0.113.0
```

Les communications normales des employés ne sont plus observées.

Cela indique que le serveur n'est plus capable de traiter correctement le trafic légitime.

---

## 10. Conclusion de l'analyse Wireshark

L'analyse du journal montre plusieurs indicateurs concordants :

* nombre élevé de requêtes SYN ;
* répétition rapide des demandes ;
* même adresse IP source ;
* perturbation des connexions légitimes ;
* paquets RST/ACK ;
* erreurs 504 Gateway Time-out ;
* disparition du trafic légitime à partir de l'entrée 125.

L'ensemble de ces éléments est cohérent avec une **attaque TCP SYN Flood**.

Dans le scénario analysé, une seule source est responsable du trafic malveillant observé. L'incident correspond donc à un **déni de service direct (DoS)**.

---

## 🧠 Compétences démontrées

Cette analyse permet de démontrer les compétences suivantes :

* lecture de journaux réseau ;
* analyse TCP ;
* compréhension du Three-Way Handshake ;
* analyse des paquets SYN ;
* identification d'un comportement réseau anormal ;
* analyse des erreurs TCP/HTTP ;
* identification d'une attaque DoS ;
* corrélation entre événements réseau et impact utilisateur.
