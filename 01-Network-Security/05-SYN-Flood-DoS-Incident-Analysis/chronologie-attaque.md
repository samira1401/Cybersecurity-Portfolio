# ⏱️ Chronologie de l'attaque SYN Flood

## 📌 Présentation de la chronologie

Ce document reconstitue la progression d'un incident de sécurité réseau à partir des données du journal TCP/HTTP capturé avec Wireshark.

L'objectif est de suivre l'évolution de l'incident depuis le fonctionnement normal du serveur jusqu'à la dégradation, puis à l'interruption progressive du service.

La chronologie permet d'identifier les principales phases suivantes :

1. Fonctionnement normal du serveur
2. Apparition d'une activité suspecte
3. Augmentation des requêtes TCP SYN
4. Dégradation des connexions légitimes
5. Apparition d'erreurs réseau et HTTP
6. Perte progressive du trafic légitime
7. Forte dégradation de la disponibilité du service

> **Note :** Cette analyse repose sur un scénario de cybersécurité simulé à des fins pédagogiques et de portfolio professionnel.

---

## 🕐 1. Fonctionnement normal — Entrées 47 à 51

### Temps : environ 3,14 à 3,35 secondes

Au début du journal, le serveur web fonctionne normalement.

Un utilisateur légitime établit une connexion TCP avec le serveur.

Les entrées 47 à 49 montrent un **TCP Three-Way Handshake** normal :

```text
47 — SYN
48 — SYN/ACK
49 — ACK
```

### Entrée 50 — Requête HTTP

Une fois la connexion TCP établie, le navigateur de l'utilisateur demande la page web :

```http
GET /sales.html HTTP/1.1
```

### Entrée 51 — Réponse du serveur

Le serveur répond :

```http
HTTP/1.1 200 OK
```

Cette réponse indique que la requête HTTP a été traitée avec succès.

### Interprétation

Cette première séquence représente un fonctionnement normal du service :

```text
Client
  │
  │ SYN
  ▼
Serveur
  │
  │ SYN/ACK
  ▼
Client
  │
  │ ACK
  ▼
Connexion TCP établie
  │
  │ GET /sales.html
  ▼
Serveur
  │
  │ HTTP/1.1 200 OK
  ▼
Client
```

À ce stade, aucun signe évident d'interruption de service n'est observé.

---

## 🚨 2. Apparition de l'activité suspecte — Entrées 52 à 54

### Temps : environ 3,39 à 3,49 secondes

Une nouvelle adresse IP apparaît dans le journal :

```text
203.0.113.0
```

Cette adresse communique avec le serveur web :

```text
192.0.2.1
```

Les entrées 52 à 54 montrent initialement un échange TCP complet :

```text
52 — SYN
53 — SYN/ACK
54 — ACK
```

### Interprétation

La première connexion de cette source ressemble encore à une connexion TCP normale.

Cependant, le comportement devient rapidement suspect lorsque la même adresse IP commence à envoyer de nouvelles requêtes SYN répétées vers le serveur.

Cette répétition constitue le premier indicateur important de l'activité malveillante.

---

## ⚠️ 3. L'attaque commence à perturber le trafic — Entrées 55 à 62

### Temps : environ 3,54 à 4,02 secondes

Pendant que l'adresse `203.0.113.0` continue d'envoyer des requêtes SYN, le serveur est encore capable de traiter certaines connexions légitimes.

Un utilisateur légitime utilisant l'adresse :

```text
198.51.100.14
```

établit une connexion TCP avec le serveur.

Les échanges sont :

```text
55 — SYN
56 — SYN/ACK
58 — ACK
```

Le navigateur demande ensuite :

```http
GET /sales.html HTTP/1.1
```

Le serveur répond :

```http
HTTP/1.1 200 OK
```

### Interprétation

À ce stade, l'activité suspecte est déjà présente, mais le serveur continue encore à répondre correctement à certains utilisateurs légitimes.

Cela montre que le service reste disponible malgré la présence du trafic anormal.

---

## 📈 4. Augmentation rapide des requêtes SYN — Entrées 63 à 83

### Temps : environ 4,09 à 7,44 secondes

Le nombre de requêtes SYN provenant de `203.0.113.0` augmente rapidement.

Le journal présente une succession de demandes de connexion :

```text
SYN
SYN
SYN
SYN
SYN
SYN
...
```

Ces requêtes sont envoyées à intervalles rapprochés.

Le comportement devient anormal car la même source génère continuellement de nouvelles tentatives de connexion vers le serveur.

### Interprétation

Cette augmentation importante du trafic SYN sollicite progressivement les ressources utilisées par le serveur pour gérer les connexions TCP.

L'attaque commence alors à avoir un impact sur le fonctionnement normal du service.

---

## ❌ 5. Apparition des premiers échecs de connexion

À mesure que les requêtes SYN augmentent, certaines connexions légitimes commencent à rencontrer des difficultés.

### Entrée 73 — RST/ACK

L'entrée 73 contient :

```text
[RST, ACK]
```

Ce paquet concerne une connexion avec un utilisateur légitime.

### Interprétation

Le paquet `RST, ACK` indique une réinitialisation de la connexion TCP.

Cela constitue un signe que certaines communications légitimes ne peuvent plus être maintenues normalement.

Le serveur commence donc à rencontrer des difficultés pour gérer correctement toutes les connexions entrantes.

---

## ⏱️ 6. Première erreur HTTP 504 — Entrée 77

L'entrée 77 contient :

```http
HTTP/1.1 504 Gateway Time-out
```

Cette erreur indique qu'une passerelle n'a pas reçu de réponse du serveur web dans le délai attendu.

### Interprétation

Le serveur commence à avoir des difficultés à traiter les demandes suffisamment rapidement.

Cette erreur constitue un indicateur important de la dégradation du service.

La situation peut être résumée ainsi :

```text
Augmentation des SYN
        ↓
Pression sur les ressources TCP
        ↓
Difficultés à traiter les connexions
        ↓
Échecs de connexions
        ↓
504 Gateway Time-out
```

---

## 🔥 7. Multiplication des requêtes SYN

Après les premières erreurs, l'activité provenant de `203.0.113.0` continue.

Le journal montre de nombreuses requêtes :

```text
203.0.113.0 → 192.0.2.1
```

```text
SYN
SYN
SYN
SYN
SYN
...
```

Le trafic suspect devient progressivement dominant.

Pendant ce temps, les connexions légitimes commencent à rencontrer davantage de problèmes.

### Interprétation

La répétition massive des requêtes SYN augmente la pression sur le mécanisme de gestion des connexions TCP du serveur.

Le service devient progressivement moins disponible pour les utilisateurs légitimes.

---

## 🟡 8. Dégradation importante du service — Entrées 119 à 124

### Temps : environ 19,20 à 20,81 secondes

L'attaque continue pendant plusieurs secondes.

Les requêtes SYN provenant de `203.0.113.0` restent nombreuses.

Les utilisateurs légitimes rencontrent désormais davantage de difficultés à établir ou maintenir leurs connexions.

### Entrée 121

Le serveur envoie :

```text
[RST, ACK]
```

vers un utilisateur légitime.

### Entrée 124

Le serveur répond également à l'adresse de l'attaquant avec :

```text
[RST, ACK]
```

### Interprétation

Ces événements montrent que le serveur rencontre désormais des difficultés importantes à gérer les nombreuses demandes de connexion.

La disponibilité du service est fortement dégradée.

---

## 🚫 9. Perte progressive du trafic légitime — À partir de l'entrée 125

### Temps : environ 21,14 secondes

À partir de l'entrée 125, le journal présente principalement des requêtes SYN provenant de :

```text
203.0.113.0
```

On observe une succession continue de requêtes :

```text
125 — SYN
126 — SYN
127 — SYN
128 — SYN
129 — SYN
130 — SYN
...
152 — SYN
```

Le même comportement continue jusqu'à la fin de la section analysée.

### Interprétation

Le trafic légitime des employés disparaît progressivement du journal.

À partir de cette phase, le serveur ne semble plus capable de traiter correctement les demandes normales des utilisateurs.

Cette situation représente une forte perturbation de la disponibilité du service web.

---

## 📊 10. Vue globale de l'évolution de l'incident

```text
                    TRAFIC NORMAL
                         │
                         ▼
                   Entrées 47–51
              TCP Handshake + HTTP
                  GET + 200 OK
                         │
                         ▼
              APPARITION DE LA SOURCE
                     SUSPECTE
                         │
                         ▼
                   Entrées 52–54
                    203.0.113.0
                         │
                         ▼
               AUGMENTATION DES SYN
                         │
                         ▼
                   Entrées 55–83
       Trafic suspect + trafic légitime
                         │
                         ▼
                 RST/ACK + 504
                Gateway Time-out
                         │
                         ▼
                  Entrées 119–124
              Forte dégradation du
                     service
                         │
                         ▼
                  Entrées 125+
             Trafic principalement
                   malveillant
                         │
                         ▼
              PERTE DE DISPONIBILITÉ
                  DU SERVICE WEB
```

---

## 📋 11. Indicateurs clés de l'incident

| Entrées | Observation                         | Interprétation                   |
| ------- | ----------------------------------- | -------------------------------- |
| 47–51   | SYN → SYN/ACK → ACK → HTTP 200 OK   | Trafic normal                    |
| 52–54   | Première connexion de `203.0.113.0` | Apparition de la source suspecte |
| 55–62   | Trafic légitime encore fonctionnel  | Serveur encore disponible        |
| 63–83   | Multiplication des requêtes SYN     | Attaque en progression           |
| 73      | `[RST, ACK]`                        | Réinitialisation d'une connexion |
| 77      | `504 Gateway Time-out`              | Dégradation du service           |
| 119–124 | Nouveaux échecs de connexion        | Forte perturbation               |
| 125+    | Principalement des requêtes SYN     | Service fortement perturbé       |

---

## 🧩 12. Analyse des trois phases principales

### Phase 1 — Fonctionnement normal

Les entrées 47 à 51 montrent une communication normale entre un utilisateur et le serveur.

Le TCP Three-Way Handshake est correctement réalisé :

```text
SYN → SYN/ACK → ACK
```

Le navigateur peut ensuite demander :

```http
GET /sales.html HTTP/1.1
```

Le serveur répond :

```http
HTTP/1.1 200 OK
```

Le service fonctionne donc normalement.

### Phase 2 — Dégradation

L'adresse `203.0.113.0` commence à générer de nombreuses requêtes SYN.

Le serveur continue temporairement à répondre à certains utilisateurs légitimes.

Cependant, des signes de dégradation apparaissent :

```text
[RST, ACK]
```

et :

```http
HTTP/1.1 504 Gateway Time-out
```

Le nombre de connexions problématiques augmente progressivement.

### Phase 3 — Forte perturbation du service

À partir de l'entrée 125, le trafic observé est principalement constitué de requêtes SYN provenant de `203.0.113.0`.

Le trafic légitime disparaît pratiquement du journal.

Le serveur devient alors fortement perturbé et les utilisateurs légitimes ne peuvent plus accéder normalement au service.

---

## 🚨 13. Identification de l'attaque

Les principaux indicateurs observés sont :

* Un nombre anormalement élevé de requêtes SYN
* Des requêtes répétées provenant de la même adresse IP
* Une fréquence élevée de tentatives de connexion
* Des échecs de connexions légitimes
* Des paquets `[RST, ACK]`
* Des erreurs `504 Gateway Time-out`
* Une disparition progressive du trafic légitime
* Une perte progressive de disponibilité du serveur

Ces indicateurs sont cohérents avec une :

> **TCP SYN Flood — attaque par déni de service direct (DoS).**

Dans ce scénario simulé, le trafic malveillant observé provient d'une seule adresse IP :

```text
203.0.113.0
```

L'incident est donc classifié comme un **DoS direct**, et non comme un DDoS.

---

## 💥 14. Impact observé

L'attaque entraîne progressivement une dégradation de la disponibilité du serveur web.

Les utilisateurs légitimes peuvent rencontrer :

* Des temps d'attente importants
* Des erreurs de connexion
* Des connexions réinitialisées
* Des erreurs `504 Gateway Time-out`
* Une impossibilité d'accéder au site web

### Impact technique

La multiplication des requêtes SYN augmente la charge liée à la gestion des connexions TCP.

Les ressources disponibles pour traiter les connexions légitimes deviennent progressivement insuffisantes.

### Impact utilisateur

Les employés peuvent rencontrer des difficultés pour accéder à la page :

```text
/sales.html
```

Ils peuvent notamment observer des délais d'attente, des erreurs ou des connexions interrompues.

### Impact organisationnel

Une interruption ou une forte dégradation d'un service web peut entraîner :

* Une baisse de productivité
* Une interruption des activités dépendantes du service
* Une dégradation de l'expérience utilisateur
* Une augmentation de la charge de travail des équipes IT et sécurité

L'impact principal observé dans ce scénario reste toutefois la **perte de disponibilité du service web**.

---

## 🔎 15. Indicateurs permettant de confirmer l'incident

Plusieurs éléments du journal doivent être analysés ensemble.

### Indicateur 1 — Source répétitive

La même adresse IP :

```text
203.0.113.0
```

apparaît de manière répétée dans les communications suspectes.

### Indicateur 2 — Requêtes SYN répétées

Le serveur reçoit continuellement :

```text
SYN
SYN
SYN
SYN
SYN
...
```

### Indicateur 3 — Dégradation des connexions légitimes

Des utilisateurs légitimes commencent à recevoir des réponses telles que :

```text
[RST, ACK]
```

### Indicateur 4 — Timeout HTTP

Le journal contient :

```http
HTTP/1.1 504 Gateway Time-out
```

### Indicateur 5 — Disparition du trafic légitime

À partir de l'entrée 125, les communications observées sont principalement associées à l'activité suspecte.

La corrélation de ces indicateurs renforce l'hypothèse d'une attaque SYN Flood.

---

## 🧠 16. Relation entre l'attaque et l'impact

L'évolution de l'incident peut être représentée de la manière suivante :

```text
Requêtes SYN répétées
        │
        ▼
Augmentation des tentatives
de connexion TCP
        │
        ▼
Pression sur les ressources
de gestion des connexions
        │
        ▼
Difficultés à traiter
les connexions légitimes
        │
        ▼
RST/ACK
        │
        ▼
504 Gateway Time-out
        │
        ▼
Dégradation du service
        │
        ▼
Perte de disponibilité
```

Cette corrélation permet de relier les observations du journal à l'impact constaté par les utilisateurs.

---

## 🎯 17. Conclusion de la chronologie

L'analyse chronologique montre une progression claire de l'incident.

Au début du journal, le serveur fonctionne normalement et les utilisateurs peuvent établir des connexions TCP puis accéder à la page `/sales.html`.

Une source externe, `203.0.113.0`, commence ensuite à générer des requêtes TCP SYN répétées.

Le serveur continue initialement à répondre aux utilisateurs légitimes, mais l'augmentation du trafic SYN entraîne progressivement des problèmes de connexion.

Les paquets `[RST, ACK]` et les erreurs `504 Gateway Time-out` constituent des indicateurs de dégradation du service.

À partir de l'entrée 125, le trafic observé est principalement associé aux requêtes SYN provenant de la source suspecte.

L'ensemble des éléments observés est cohérent avec une :

> 🚨 **TCP SYN Flood — Déni de service direct (DoS)**

Cette attaque affecte principalement la **disponibilité du serveur web** et empêche progressivement les utilisateurs légitimes d'accéder normalement au service.

---

## 📌 18. Synthèse professionnelle

| Élément                                 | Résultat                              |
| --------------------------------------- | ------------------------------------- |
| **Type d'incident**                     | TCP SYN Flood                         |
| **Catégorie**                           | Déni de service (DoS)                 |
| **Source suspecte**                     | `203.0.113.0`                         |
| **Serveur ciblé**                       | `192.0.2.1`                           |
| **Port ciblé**                          | TCP `443`                             |
| **Protocole observé**                   | TCP / HTTP                            |
| **Première activité suspecte**          | Entrée 52                             |
| **Premiers signes de dégradation**      | Entrées 73 et 77                      |
| **Forte dégradation**                   | Entrées 119–124                       |
| **Perte importante du trafic légitime** | À partir de l'entrée 125              |
| **Impact principal**                    | Perte de disponibilité du serveur web |
| **Classification**                      | DoS direct                            |

---

## 🛡️ 19. Pourquoi cette chronologie est importante

Une chronologie permet à un analyste de sécurité de ne pas seulement identifier une anomalie, mais également de comprendre **comment l'incident s'est développé dans le temps**.

Dans cette analyse, elle permet de relier :

```text
Situation normale
        ↓
Apparition du trafic suspect
        ↓
Augmentation des SYN
        ↓
Premiers échecs de connexion
        ↓
504 Gateway Time-out
        ↓
Dégradation du service
        ↓
Perte du trafic légitime
```

Cette méthode d'analyse est importante dans le cadre d'une investigation de sécurité, car elle permet de relier les événements réseau à leurs conséquences sur le service.

---

## 📚 20. Données utilisées pour l'analyse

Cette chronologie est basée sur le journal TCP/HTTP fourni dans le cadre du scénario.

Les principaux éléments analysés sont :

* Numéro des entrées
* Horodatage des paquets
* Adresse IP source
* Adresse IP destination
* Protocole
* Ports TCP
* Drapeaux TCP tels que `SYN`, `ACK` et `RST`
* Requêtes HTTP
* Réponses HTTP
* Erreurs de connexion
* Évolution du trafic légitime et suspect

Les données peuvent être consultées dans le fichier :

```text
evidence/Wireshark-TCP-HTTP-Log.xlsx
```

---

## ⚠️ Disclaimer

Ce projet est réalisé à des fins pédagogiques et professionnelles dans le cadre d'un portfolio de cybersécurité.

Le scénario, les adresses IP et les données réseau utilisés dans cette analyse appartiennent à un environnement de formation simulé.

Aucun système réel n'a été ciblé ou attaqué dans le cadre de cette analyse.

---

## 👤 Auteur

**Samira Bel**

Portfolio Cybersécurité
