# ⏱️ Chronologie de l'attaque SYN Flood


## 1. Objectif


Cette chronologie permet de reconstituer l'évolution de l'incident à partir des entrées du journal TCP/HTTP.


L'analyse montre une progression en plusieurs étapes :


1. fonctionnement normal du serveur ;
2. apparition du trafic suspect ;
3. augmentation des requêtes SYN ;
4. dégradation des connexions légitimes ;
5. interruption progressive du service.


---


## 2. Fonctionnement normal — Entrées 47 à 51


### ⏱️ Environ 3,14 à 3,35 secondes


Au début de l'analyse, le serveur fonctionne normalement.


Les entrées 47 à 49 montrent un **TCP Three-Way Handshake** classique :


```text
47 — SYN
48 — SYN/ACK
49 — ACK

La connexion TCP est ensuite utilisée pour une communication HTTP.

Entrée 50

Le navigateur de l'utilisateur envoie la requête :

GET /sales.html HTTP/1.1
Entrée 51

Le serveur répond :

HTTP/1.1 200 OK
Interprétation

Cette séquence représente le fonctionnement normal d'une connexion TCP vers le serveur web :

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
  │ HTTP 200 OK
  ▼
Client
3. Début de l'activité suspecte — Entrées 52 à 54
⏱️ Environ 3,39 à 3,49 secondes

Une nouvelle adresse IP apparaît dans le journal :

203.0.113.0

Cette adresse communique avec le serveur web :

192.0.2.1

Les premières communications suivent encore un processus TCP normal :

52 — SYN
53 — SYN/ACK
54 — ACK
Interprétation

La première connexion de cette source ressemble à une connexion TCP normale.

Cependant, le comportement devient rapidement anormal lorsque la même source commence à envoyer de nouvelles requêtes SYN répétées.

4. L'attaque commence à perturber le trafic — Entrées 55 à 62
⏱️ Environ 3,54 à 4,02 secondes

Pendant que l'adresse 203.0.113.0 continue d'envoyer des requêtes SYN, un utilisateur légitime parvient encore à accéder au serveur.

L'adresse IP :

198.51.100.14

établit une connexion TCP :

55 — SYN
56 — SYN/ACK
58 — ACK

Le navigateur demande ensuite :

GET /sales.html HTTP/1.1

Le serveur répond :

HTTP/1.1 200 OK
Interprétation

À ce stade, l'attaque est active, mais le serveur est encore capable de traiter certaines demandes légitimes.

Cela montre que l'attaque est en cours d'escalade et que le service n'est pas encore complètement indisponible.

5. Augmentation rapide des requêtes SYN — Entrées 63 à 83
⏱️ Environ 4,09 à 7,44 secondes

Le nombre de requêtes SYN provenant de 203.0.113.0 augmente.

On observe une succession rapide de paquets :

SYN
SYN
SYN
SYN
SYN
SYN
...

Ces requêtes sont envoyées à intervalles rapprochés.

Le comportement diffère du trafic normal d'un utilisateur qui établit une connexion puis échange des données avec le serveur.

6. Apparition des premiers échecs de connexion

À mesure que les requêtes SYN augmentent, les connexions des utilisateurs légitimes commencent à rencontrer des difficultés.

Entrée 73 — RST/ACK

L'entrée 73 montre :

[RST, ACK]

Cette communication concerne un utilisateur légitime.

Le paquet RST, ACK indique une réinitialisation de la connexion TCP.

Interprétation

La tentative de connexion de l'utilisateur n'a pas pu être maintenue normalement.

7. Première erreur 504 Gateway Time-out
Entrée 77

Le journal contient :

HTTP/1.1 504 Gateway Time-out
Interprétation

Cette erreur indique qu'une passerelle a attendu trop longtemps une réponse du serveur web.

Le serveur commence donc à avoir des difficultés à traiter les demandes dans les délais attendus.

Cette erreur constitue un indicateur important de la dégradation du service.

8. Multiplication des requêtes SYN

Les entrées suivantes continuent de montrer des requêtes provenant de :

203.0.113.0

On observe de nombreuses tentatives de connexion :

SYN
SYN
SYN
SYN
SYN
SYN
...

Le trafic malveillant devient de plus en plus fréquent alors que les communications légitimes commencent à échouer.

Interprétation

Le volume important de requêtes SYN sollicite progressivement les ressources disponibles pour gérer les connexions TCP.

9. Dégradation importante du service — Entrées 119 à 124
⏱️ Environ 19,20 à 20,81 secondes

L'attaque continue et les requêtes SYN provenant de 203.0.113.0 restent nombreuses.

Les utilisateurs légitimes rencontrent encore des problèmes de connexion.

Entrée 121

Le serveur envoie :

[RST, ACK]

vers un utilisateur légitime.

Entrée 124

Le serveur répond également à l'adresse de l'attaquant avec :

[RST, ACK]
Interprétation

Le serveur rencontre désormais des difficultés importantes à gérer correctement les nombreuses demandes de connexion.

Les erreurs observées montrent que la disponibilité du service est fortement dégradée.

10. Perte du trafic légitime — À partir de l'entrée 125
⏱️ Environ 21,14 secondes

À partir de l'entrée 125, le journal présente principalement des requêtes SYN provenant de :

203.0.113.0

On observe une succession continue de requêtes :

125 — SYN
126 — SYN
127 — SYN
128 — SYN
129 — SYN
130 — SYN
131 — SYN
132 — SYN
133 — SYN
134 — SYN
135 — SYN
136 — SYN
137 — SYN
138 — SYN
139 — SYN
140 — SYN
...

Le même comportement continue jusqu'à la fin de la section analysée.

Interprétation

Le trafic légitime des employés n'apparaît pratiquement plus dans le journal.

Le serveur n'est donc plus en mesure de répondre correctement aux demandes normales des utilisateurs.

11. Vue globale de l'évolution de l'incident

L'évolution de l'incident peut être représentée ainsi :

                    TRAFIC NORMAL
                         │
                         ▼
                   Entrées 47–51
              TCP Handshake + HTTP 200
                         │
                         ▼
              APPARITION DE LA SOURCE
                     SUSPECTE
                         │
                         ▼
                   Entrées 52–54
              Premier échange TCP
                  avec 203.0.113.0
                         │
                         ▼
               AUGMENTATION DES SYN
                         │
                         ▼
                   Entrées 55–83
       Trafic malveillant + premiers échecs
                  des utilisateurs
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
                  Entrée 125+
             Trafic principalement
                   malveillant
                         │
                         ▼
                 INTERRUPTION DU
                      SERVICE
12. Indicateurs clés de l'incident
Entrées	Observation	Interprétation
47–51	SYN → SYN/ACK → ACK → HTTP 200 OK	Trafic normal
52–54	Première connexion de 203.0.113.0	Apparition de la source suspecte
55–62	Trafic légitime encore fonctionnel	Serveur encore disponible
63–83	Multiplication des requêtes SYN	Attaque en progression
73	RST/ACK	Échec d'une connexion
77	504 Gateway Time-out	Dégradation du service
119–124	Nouveaux échecs de connexion	Forte perturbation
125+	Principalement des requêtes SYN	Serveur fortement perturbé
13. Analyse de la progression de l'attaque

L'analyse chronologique permet de distinguer trois phases principales.

Phase 1 — Début de l'attaque

L'adresse 203.0.113.0 commence à communiquer avec le serveur.

Le serveur répond initialement normalement.

Le trafic légitime continue de fonctionner.

Phase 2 — Dégradation

Les requêtes SYN deviennent beaucoup plus fréquentes.

Les utilisateurs légitimes commencent à rencontrer :

des échecs de connexion ;
des paquets RST, ACK ;
des erreurs 504 Gateway Time-out.

Le serveur devient progressivement moins disponible.

Phase 3 — Interruption du service

À partir de l'entrée 125, les communications observées sont principalement constituées de requêtes SYN provenant de l'adresse suspecte.

Le trafic légitime disparaît pratiquement du journal.

Cette situation indique que le serveur est fortement perturbé et ne peut plus traiter correctement les demandes normales.

14. Identification de l'attaque

Les principaux indicateurs observés sont :

un nombre anormalement élevé de requêtes SYN ;
des requêtes répétées provenant de la même adresse IP ;
une fréquence élevée des demandes de connexion ;
des échecs de connexions légitimes ;
des paquets RST, ACK ;
des erreurs 504 Gateway Time-out ;
une disparition progressive du trafic légitime.

Ces indicateurs sont cohérents avec une :

TCP SYN Flood — attaque par déni de service direct (DoS).

Dans ce scénario, le trafic malveillant observé provient d'une seule adresse IP.

Il s'agit donc d'un DoS direct, et non d'un DDoS.

15. Impact observé

L'attaque entraîne progressivement une dégradation des performances du serveur web.

Les utilisateurs légitimes peuvent rencontrer :

des temps d'attente importants ;
des erreurs de connexion ;
des connexions réinitialisées ;
des erreurs 504 Gateway Time-out ;
une impossibilité d'accéder au site web.

L'impact principal est donc une perte de disponibilité du service web.

16. Conclusion

L'analyse chronologique montre une progression claire de l'incident.

Le serveur fonctionne normalement au début de l'enregistrement.

Une source externe, 203.0.113.0, commence ensuite à envoyer des requêtes TCP SYN répétées.

Le serveur continue initialement à répondre aux utilisateurs légitimes, mais l'augmentation rapide du trafic SYN entraîne progressivement des problèmes de connexion.

Les paquets RST, ACK et les erreurs 504 Gateway Time-out montrent que le service commence à se dégrader.

À partir de l'entrée 125, le trafic observé est presque exclusivement associé à l'activité malveillante.

L'ensemble des observations permet d'identifier l'incident comme une :

🚨 TCP SYN Flood — Déni de service direct (DoS)

Cette attaque affecte principalement la disponibilité du serveur web et empêche progressivement les utilisateurs légitimes d'accéder au service.
