# 🛡️ Mesures de mitigation — TCP SYN Flood

## 📌 Présentation

Ce document présente les mesures de protection et de mitigation pouvant être mises en place à la suite de l'identification d'une attaque TCP SYN Flood.

L'objectif est de réduire l'impact de l'attaque, restaurer la disponibilité du serveur web, protéger les utilisateurs légitimes et améliorer la capacité de détection des futures attaques.

> **Note :** Les mesures présentées dans ce document sont basées sur le scénario de cybersécurité simulé utilisé dans ce projet.

---

## 🎯 Objectifs de la mitigation

Les principaux objectifs sont :

* Restaurer la disponibilité du serveur web
* Réduire l'impact du trafic SYN malveillant
* Protéger les connexions des utilisateurs légitimes
* Détecter rapidement une nouvelle attaque
* Limiter la consommation des ressources TCP
* Améliorer la surveillance du trafic réseau
* Mettre en place une procédure de réponse à incident

---

## 🚨 1. Mesures immédiates

Lorsqu'une attaque SYN Flood est détectée, les premières actions doivent viser à limiter rapidement son impact.

### 1.1 Identifier la source du trafic suspect

Dans le scénario analysé, l'adresse IP suspecte est :

```text
203.0.113.0
```

Le serveur ciblé est :

```text
192.0.2.1
```

Le service ciblé est :

```text
TCP/443 — HTTPS
```

L'identification de la source permet aux équipes de sécurité de commencer l'analyse et de déterminer quelles mesures de filtrage peuvent être appliquées.

---

## 🔥 2. Filtrage réseau et pare-feu

Une première mesure consiste à utiliser les équipements de sécurité réseau afin de filtrer le trafic anormal.

Dans ce scénario, une règle temporaire pourrait être mise en place pour limiter ou bloquer le trafic provenant de la source identifiée.

### Exemple conceptuel

```text
Source      : 203.0.113.0
Destination : 192.0.2.1
Port        : TCP/443
Action      : Bloquer ou limiter
```

### ⚠️ Limitation

Le blocage d'une seule adresse IP ne constitue pas une protection complète contre les attaques DoS ou DDoS.

Un attaquant peut utiliser :

* Plusieurs adresses IP
* Des adresses IP différentes
* Des infrastructures compromises
* Des techniques d'usurpation d'adresse source

Le filtrage doit donc être combiné avec d'autres mécanismes de protection.

---

## 🚦 3. Rate Limiting

Le **rate limiting** permet de limiter le nombre de nouvelles connexions ou de requêtes provenant d'une même source pendant une période donnée.

Par exemple, un équipement réseau peut détecter qu'une source génère un nombre anormalement élevé de nouvelles connexions TCP.

Le principe peut être représenté ainsi :

```text
Trafic normal
      ↓
Connexion autorisée


Trafic excessif
      ↓
Limitation
      ↓
Requêtes supplémentaires rejetées ou ralenties
```

Cette technique permet de réduire l'impact d'un grand nombre de tentatives de connexion.

---

## 🍪 4. SYN Cookies

Les **SYN Cookies** constituent une mesure importante contre les attaques SYN Flood.

Normalement, lorsqu'un serveur reçoit un SYN, il doit conserver certaines informations concernant la connexion en attente.

Un grand nombre de connexions incomplètes peut donc consommer les ressources disponibles.

Avec les SYN Cookies, le serveur peut limiter la quantité d'état conservée pour les connexions qui n'ont pas encore terminé le handshake TCP.

### Principe

```text
Client
   │
   │ SYN
   ▼
Serveur
   │
   │ SYN/ACK avec cookie
   ▼
Client
   │
   │ ACK
   ▼
Serveur
   │
   │ Validation
   ▼
Connexion établie
```

### Avantage

Les SYN Cookies permettent de réduire le risque d'épuisement des ressources liées aux connexions TCP incomplètes.

---

## 🧱 5. Protection du pare-feu

Le pare-feu peut être configuré afin d'identifier les comportements TCP anormaux.

Les règles peuvent notamment surveiller :

* Le nombre de SYN reçus
* Le nombre de connexions simultanées
* Le nombre de connexions incomplètes
* Les connexions répétitives provenant d'une même source
* Les anomalies de trafic vers TCP/443

Une alerte peut être générée lorsque le trafic dépasse un seuil défini.

### Exemple conceptuel

```text
Nombre de SYN normal
        ↓
Trafic autorisé


Nombre de SYN anormal
        ↓
Détection
        ↓
Alerte
        ↓
Limitation / Filtrage
```

---

## 👁️ 6. IDS / IPS

Un système **IDS (Intrusion Detection System)** peut être utilisé pour détecter les comportements réseau suspects.

Un système **IPS (Intrusion Prevention System)** peut aller plus loin en intervenant automatiquement sur certains trafics malveillants.

Dans le contexte de cette attaque, un IDS/IPS peut surveiller :

```text
Volume de SYN
       ↓
Fréquence des connexions
       ↓
Adresse source
       ↓
Destination
       ↓
Évolution du trafic
```

Une augmentation anormale du nombre de SYN peut alors déclencher une alerte.

---

## 📊 7. Surveillance et monitoring

La surveillance continue du trafic réseau permet de détecter rapidement les anomalies.

Les éléments suivants peuvent être surveillés :

* Nombre de connexions TCP
* Nombre de paquets SYN
* Nombre de paquets SYN/ACK
* Nombre de paquets RST
* Connexions TCP incomplètes
* Utilisation CPU du serveur
* Utilisation mémoire
* Nombre de connexions simultanées
* Temps de réponse du serveur
* Taux d'erreurs HTTP
* Erreurs `504 Gateway Time-out`

### Flux de surveillance

```text
Trafic réseau
      ↓
Collecte des logs
      ↓
Analyse
      ↓
Détection d'anomalie
      ↓
Alerte sécurité
      ↓
Investigation
```

---

## 🚨 8. Détection basée sur les indicateurs observés

Les observations réalisées pendant cette investigation peuvent servir à définir des indicateurs de détection.

### Indicateur 1 — Nombre élevé de SYN

Une augmentation importante du nombre de paquets SYN peut indiquer une tentative d'attaque.

### Indicateur 2 — Source répétitive

Une même adresse IP générant continuellement des connexions peut être considérée comme suspecte selon le contexte.

### Indicateur 3 — Connexions incomplètes

Un nombre important de connexions TCP qui restent dans une phase initiale peut constituer un indicateur.

### Indicateur 4 — Augmentation des erreurs

Une augmentation des erreurs telles que :

```http
HTTP/1.1 504 Gateway Time-out
```

peut signaler une dégradation du service.

### Indicateur 5 — RST/ACK

Une augmentation inhabituelle des paquets :

```text
[RST, ACK]
```

peut également être corrélée avec des problèmes de connexion.

---

## 🌐 9. Protection DDoS externe

Pour les infrastructures exposées à Internet, une solution de protection DDoS peut être utilisée.

Le principe consiste à filtrer une partie du trafic avant qu'il n'atteigne directement l'infrastructure de l'organisation.

### Architecture conceptuelle

```text
Internet
   │
   ▼
Protection DDoS
   │
   ├── Trafic légitime ───────► Serveur
   │
   └── Trafic malveillant ───► Filtrage
```

Cette approche est particulièrement importante lorsque le volume de trafic dépasse les capacités des équipements locaux.

---

## 🔄 10. Redondance et haute disponibilité

La disponibilité peut également être améliorée grâce à une architecture redondante.

Par exemple :

```text
                 Internet
                    │
                    ▼
              Load Balancer
               /          \
              /            \
             ▼              ▼
        Serveur Web 1   Serveur Web 2
```

Si un serveur rencontre une surcharge, un autre serveur peut continuer à traiter une partie du trafic légitime.

La redondance ne remplace cependant pas les mécanismes de protection contre les attaques.

---

## 🧪 11. Tests et validation

Après la mise en place des mesures de protection, il est nécessaire de vérifier leur efficacité.

Les tests peuvent notamment vérifier :

* La détection des volumes anormaux de SYN
* Le fonctionnement du rate limiting
* La génération des alertes
* Le comportement du pare-feu
* La disponibilité du serveur
* La capacité à maintenir les connexions légitimes
* La remontée correcte des logs

Les tests doivent être réalisés dans un environnement autorisé et contrôlé.

> **Important :** Aucun test de charge ou d'attaque ne doit être effectué contre une infrastructure réelle sans autorisation explicite.

---

## 📝 12. Procédure de réponse à incident

Une procédure de réponse à incident peut être structurée en plusieurs étapes.

### Étape 1 — Détection

Identifier une augmentation anormale du trafic SYN.

```text
Détection d'une anomalie
```

### Étape 2 — Analyse

Examiner :

* Les adresses IP sources
* Les destinations
* Les ports
* Les flags TCP
* Le volume du trafic
* Les erreurs générées
* L'impact sur les utilisateurs

### Étape 3 — Containment

Limiter temporairement le trafic malveillant afin de réduire son impact.

Exemples :

* Rate limiting
* Filtrage réseau
* Blocage temporaire
* Activation d'une protection DDoS

### Étape 4 — Restauration

Vérifier que le serveur revient à un fonctionnement normal.

Les indicateurs à surveiller sont notamment :

```text
↓ SYN anormaux
↓ erreurs HTTP
↓ connexions échouées
↑ disponibilité
↑ connexions légitimes réussies
```

### Étape 5 — Post-incident

Après l'incident :

* Conserver les logs
* Documenter les événements
* Identifier la cause
* Évaluer l'efficacité des protections
* Mettre à jour les règles de sécurité
* Améliorer la surveillance

---

## 🧠 13. Recommandations pour ce scénario

Pour le scénario étudié, les mesures prioritaires seraient :

| Priorité          | Mesure          | Objectif                                |
| ----------------- | --------------- | --------------------------------------- |
| 🔴 Haute          | Rate limiting   | Réduire les requêtes excessives         |
| 🔴 Haute          | SYN Cookies     | Limiter l'épuisement des ressources TCP |
| 🔴 Haute          | Monitoring      | Détecter rapidement les anomalies       |
| 🔴 Haute          | Firewall        | Filtrer le trafic suspect               |
| 🟠 Moyenne        | IDS/IPS         | Détecter et bloquer certaines attaques  |
| 🟠 Moyenne        | Protection DDoS | Filtrer le trafic à grande échelle      |
| 🟡 Complémentaire | Redondance      | Améliorer la disponibilité              |

---

## 🔎 14. Corrélation avec l'analyse Wireshark

Les mesures de mitigation proposées sont directement liées aux observations réalisées dans le journal réseau.

L'analyse a montré :

```text
203.0.113.0
       │
       ▼
Nombreuses requêtes SYN
       │
       ▼
Pression sur les connexions TCP
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

Les mesures de protection cherchent donc à interrompre cette chaîne le plus tôt possible.

Par exemple :

```text
Nombreuses requêtes SYN
       │
       ▼
Détection
       │
       ▼
Rate limiting / Firewall
       │
       ▼
Réduction du trafic malveillant
       │
       ▼
Ressources disponibles
       │
       ▼
Connexions légitimes maintenues
```

---

## 🎯 15. Résultat attendu

Une stratégie de mitigation correctement mise en place doit permettre :

* De détecter rapidement une augmentation anormale des SYN
* De limiter le nombre de connexions excessives
* De préserver les ressources du serveur
* De maintenir les connexions des utilisateurs légitimes
* De réduire les erreurs HTTP
* De réduire les réinitialisations de connexion
* D'améliorer la disponibilité du service
* De faciliter l'investigation par les équipes de sécurité

L'objectif n'est pas uniquement de bloquer une adresse IP, mais de mettre en place une **défense en profondeur**.

---

## 🛡️ 16. Défense en profondeur

Une protection efficace contre les attaques réseau repose sur plusieurs niveaux de défense.

```text
                  Internet
                     │
                     ▼
              Protection DDoS
                     │
                     ▼
                  Firewall
                     │
                     ▼
                 IDS / IPS
                     │
                     ▼
              Rate Limiting
                     │
                     ▼
                SYN Cookies
                     │
                     ▼
                Serveur Web
                     │
                     ▼
                 Monitoring
```

Chaque niveau apporte une protection complémentaire.

Cette approche permet d'éviter qu'une seule mesure de sécurité constitue un point de défaillance unique.

---

## 📌 17. Limites des mesures de mitigation

Aucune mesure ne garantit à elle seule une protection complète.

### Blocage d'une adresse IP

Le blocage peut être efficace contre une source connue, mais il devient moins efficace si plusieurs sources sont utilisées.

### Rate limiting

Une limitation trop stricte peut également affecter des utilisateurs légitimes.

### SYN Cookies

Les SYN Cookies réduisent le risque d'épuisement des ressources liées aux connexions TCP incomplètes, mais ils ne remplacent pas une stratégie globale de protection.

### IDS/IPS

Les systèmes de détection doivent être correctement configurés afin de limiter les faux positifs et les faux négatifs.

### Protection DDoS

Une solution externe peut améliorer considérablement la résilience, mais elle doit être correctement configurée et intégrée à l'architecture réseau.

La meilleure approche reste donc une combinaison de plusieurs contrôles de sécurité.

---

## 📈 18. Amélioration continue

Après chaque incident, les mesures de sécurité doivent être réévaluées.

L'équipe de sécurité peut analyser :

* Ce qui a permis de détecter l'attaque
* Le temps nécessaire pour identifier l'incident
* Le temps nécessaire pour restaurer le service
* Les règles de sécurité qui ont fonctionné
* Les règles qui doivent être améliorées
* Les logs disponibles
* Les alertes générées
* L'impact sur les utilisateurs

Cette démarche permet d'améliorer progressivement la capacité de réponse de l'organisation.

---

## 🧠 19. Leçons apprises

Cette investigation montre qu'une attaque SYN Flood peut progressivement transformer un trafic réseau apparemment normal en incident de disponibilité.

Les principaux enseignements sont :

* La surveillance du trafic TCP est essentielle.
* Une augmentation anormale des SYN doit être investiguée.
* Les événements réseau doivent être corrélés avec les erreurs rencontrées par les utilisateurs.
* Le rate limiting peut réduire l'impact des connexions excessives.
* Les SYN Cookies peuvent protéger les ressources liées aux connexions incomplètes.
* Les IDS/IPS permettent d'améliorer la détection.
* Une protection DDoS peut être nécessaire pour les infrastructures fortement exposées.
* Une stratégie de défense en profondeur est préférable à une seule mesure de protection.
* Les logs doivent être conservés afin de permettre une investigation post-incident.
* Les procédures de réponse à incident doivent être préparées avant qu'un incident ne survienne.

---

## 🏁 20. Conclusion

L'analyse du scénario a permis d'identifier une attaque TCP SYN Flood ayant progressivement entraîné une perte de disponibilité du serveur web.

Les principales mesures recommandées sont :

* SYN Cookies
* Rate limiting
* Filtrage par pare-feu
* IDS/IPS
* Monitoring réseau
* Protection DDoS
* Architecture redondante
* Procédure de réponse à incident

La combinaison de ces mécanismes permet de réduire la probabilité qu'une attaque similaire provoque une interruption importante du service.

Cette investigation démontre également l'importance d'une approche structurée de la cybersécurité :

```text
Détecter
   ↓
Analyser
   ↓
Contenir
   ↓
Restaurer
   ↓
Améliorer
```

L'objectif final est de maintenir la confidentialité, l'intégrité et surtout la disponibilité des services critiques de l'organisation.

---

## 📚 Données utilisées

Cette analyse s'appuie sur le journal TCP/HTTP fourni dans le scénario de formation.

Le fichier de preuve utilisé dans le projet est :

```text
evidence/Wireshark-TCP-HTTP-Log.xlsx
```

Les analyses précédentes peuvent être consultées dans :

```text
rapport-incident.md
analyse-wireshark.md
chronologie-attaque.md
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
