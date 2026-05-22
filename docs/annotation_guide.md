# Guide d'annotation — Crible

**Version** : 1.1 *(dernière révision commitée de ce fichier)*  
**Aligné avec** : `eval_protocol.md`, `README.md`

**Politique de version** : le numéro n’est incrémenté qu’au commit qui enregistre les changements dans le dépôt. Les modifications locales non commitées ne font pas monter ce numéro — consulter l’historique avec `git log docs/annotation_guide.md`. En cas de doute, la version affichée ici est celle du dernier commit, pas du brouillon en cours.

Ce document fixe les règles pour annoter les messages du corpus pilote (puis le corpus complet). Il doit être lu *avant* toute annotation.

---

## 1. Tâche principale

Pour chaque message, répondre à :

> Ce message mérite-t-il une notification à l'utilisateurice, si iel a mis le canal en sourdine ?

| Label | Valeur | Signification |
|-------|--------|---------------|
| `CRITIQUE` | `1` | Oui — l'utilisateurice risque de rater quelque chose d'important |
| `NON_CRITIQUE` | `0` | Non — bruit, social léger, hors sujet |

En cas de doute, demander l'avis de l'autre annotateurice. Ne pas trancher seule sur les cas ambigus.

La criticité se juge **toujours par rapport au canal** que l'utilisateurice a mis en sourdine (voir § 2).

---

## 2. Généralisation et automatisation (approche TAL)

Ce guide n'a **pas** vocation à lister tous les types de chats ni tous les messages possibles. En bons TAListes, on vise une définition **générale**, puis une modélisation automatique.

### 2.1. Principe de base

Un même texte peut être `0` dans un canal et `1` dans un autre. Annoter avec la question :
*« Si ce canal est en sourdine, rater ce message pose-t-il un problème concret ? »*

### 2.2. Ce que l'on veut automatiser

Le système doit apprendre des **signaux abstraits** qui se transfèrent entre contextes :

- urgence temporelle ;
- obligation / action attendue ;
- impact si non-lu ;
- dynamique conversationnelle (cascade, replies) ;
- importance relative de l'émetteurice ;
- engagement social informatif.

L'annotation sert à fournir ces signaux au modèle, pas à fabriquer un catalogue exhaustif de cas.

### 2.3. Positionnement méthodologique

- Crible est d'abord un projet d'application IA/TAL.
- L'annotation est une étape de supervision nécessaire, mais limitée.
- On privilégie des règles robustes + apprentissage, plutôt que des listes figées de thèmes.

---

## 3. Sous-étiquettes (si `CRITIQUE = 1`)

Une ou plusieurs valeurs, séparées par `|` dans le CSV :

| Code | Quand l'utiliser |
|------|------------------|
| `urgence` | Action ou réponse attendue très vite (aujourd'hui, maintenant, « urgent ») |
| `deadline` | Date/heure limite explicite (rendu, réunion, événement, etc.) |
| `action` | Demande claire à quelqu'un (« envoie », « valide », « réponds », @mention + consigne) |
| `impact` | Conséquence importante si non lu (annulation, incident, panne bloquante, etc.) |
| `social_engagement` | Pertinence surtout signalée par beaucoup de réactions/réponses → *à confirmer par les annotateurices* |
| `cascade` | Ce message fait partie d'une séquence qui, prise ensemble, forme une info critique (voir § 4.4) |

Un message peut cumuler plusieurs sous-étiquettes (ex. `deadline|action` ou `deadline|cascade`).

---

## 4. Critères détaillés

### 4.1. `CRITIQUE = 1` — OUI notifier

*(Formulations ci-dessous valables pour tout contexte de chat.)*

- Changement de date/heure d'un événement qui engage le groupe — y compris si l'heure n'est pas précisée : l'information reste très importante.
- Consigne ou obligation avec échéance (rendu, inscription, présence obligatoire, convocation).
- Demande directe à la personne ou au groupe (« il faut que vous… », « merci de confirmer avant… »).
- Incident bloquant ou annulation.
- Message d'un émetteurice prioritaire dans **ce** canal avec info factuelle nouvelle.
- Fort engagement social et contenu informatif (pas seulement un meme).

### 4.2. `CRITIQUE = 0` — NON notifier

- Salutations, remerciements, blagues sans conséquence.
- Mèmes / images virales avec réactions mais *sans* info nouvelle pour le groupe.
- « ok », « +1 », emojis seuls.
- Discussions hors sujet sans deadline ni action.
- Messages déjà couverts par un message critique plus récent dans les 10 min (doublon) — sauf s'ils appartiennent à une cascade en cours (§ 4.4).

### 4.3. Rareté des messages « complets »

Un message explicite, complet et pertinent en une seule fois est très rare dans les flux réels.

**Exemple (cas rare)** : « Le partiel INFO est reporté au 22/06 à 14h en amphi B. » — date, heure, lieu et objet sont réunis dans un seul envoi.

En pratique, la plupart des informations importantes arrivent fragmentées : formulation vague, détails manquants, reformulations, réponses qui précisent la date ou le lieu. Ne pas exiger qu'un message isolé ressemble à l'exemple A pour le labeliser `1` : un fragment peut être `0` seul tout en participant à une info critique une fois la séquence reconstituée (§ 4.4).

### 4.4. Information en cascade (message après message)

Souvent, l'information critique n'est pas dans un message unique mais se construit *en cascade*. Deux formes fréquentes :

1. Enchaînement : plusieurs messages successifs du même émetteurice dans le fil.
2. Réponse dans le fil : un autre émetteurice répond au message initial (fonction « répondre » / reply du chat : message 2 en réponse directe au message 1).

| Ordre | Message (exemple) | Émetteurice | Seul ? | Dans la cascade ? |
|-------|-------------------|-------------|--------|-------------------|
| 1 | « Le partiel est décalé. » | [user_A] | Souvent `0` ou `1` ambigu | Oui |
| 2 | « C'est le 22 juin. » | [user_A] | `0` (incomplet) | Oui (enchaînement) |
| 3 | « À 14h, amphi B. » | [user_A] | `0` (incomplet) | Oui (enchaînement) |
| 1 | « Le partiel est décalé. » | [user_A] | Souvent `0` ou `1` ambigu | Oui |
| 2 | « Oui, le 22/06 à 14h, amphi B. » | [user_B] *(réponse au msg 1)* | `0` ou `1` seul | Oui (réponse) |
| Ensemble | Séquence liée (enchaînement et/ou réponses) | — | — | `1` critique (équivalent à l'exemple A) |

**Règles d'annotation (pilote)** :

- Annoter chaque message avec `label` (0/1) comme d'habitude.
- Si le message participe à une cascade : remplir `cascade_id` (même identifiant pour toute la séquence, ex. `cascade_042`) et ajouter la sous-étiquette `cascade`.
- Si la séquence vue dans son ensemble est critique, au moins un message de la cascade doit être `1` (en pratique : le dernier qui rend l'info exploitable, ou le premier si l'annonce initiale suffit à alerter).
- **Fenêtre indicative** à discuter (à calibrer sur le pilote) :
  - **Enchaînement** : messages du même émetteurice dans le fil, à moins de 15 minutes d'intervalle ;
  - **Réponse** : message qui répond à un autre (reply du chat : message 2 en réponse directe au message 1), même si l'émetteurice est différente — la précision peut venir d'une autre personne. Renseigner `reply_to_id` (id du message parent) dans le CSV.
- Une cascade peut mélanger les deux : annonce initiale + réponses d'autres personnes + compléments du même émetteurice ; même `cascade_id` pour tout le groupe.

**Fonctionnalité Crible (cible produit, pas seulement annotation)** : le système doit repérer et agréger ce type d'informations successives (enchaînements et fils de réponses) avant de notifier — voir § 9.1.

### 4.5. Cas limites (à trancher)

| Exemple | Recommandation v1 | Discussion |
|---------|-------------------|------------|
| « RDV demain ? » sans heure ni enjeu | `0` | Pas assez d'impact seul |
| Blague avec 80 réactions | `0` | `social_engagement` seul ≠ critique |
| « Quelqu'un a le poly ? » | `0` sauf urgence imminente | Action faible |
| Rappel « n'oubliez pas le devoir » (sans date) | `1` + `action` | Impact pédagogique |
| « Examen décalé au 15 » sans heure | `1` + `deadline` (+ `impact`) | À conserver : info très importante ; pas de `0` par absence d'heure |
| « Le partiel est décalé » puis précisions en 2–3 messages | `1` sur la séquence (`cascade`) | Voir § 4.4 ; messages isolés souvent incomplets |
| Annonce vague + réponse d'une autre personne qui précise date/lieu | `1` sur la séquence (`cascade`) | Reply au message initial ; émetteurices différentes, même `cascade_id` |

---

## 5. Engagement social (`social_engagement`) → *à discuter*

Utiliser cette sous-étiquette quand :

- le nombre de réactions ou de réponses est nettement au-dessus de la moyenne du canal (repère : top 10 % du pilote) ;
- et le texte contient une information, pas seulement de l'humour. (? *à discuter*)

Ne pas étiqueter `CRITIQUE` uniquement à cause des réactions si le contenu est une blague/meme.

**Champs CSV utiles** : `nb_reactions`, `nb_replies` (remplis à la collecte).

---

## 6. Colonnes CSV recommandées (pilote et corpus)

En plus des colonnes de base (`data/README.md`), prévoir pour le pilote :

| Colonne | Description |
|---------|-------------|
| `cascade_id` | Identifiant commun à une séquence (vide si message isolé) |
| `reply_to_id` | Id du message auquel celui-ci répond (vide si pas une reply) — permet les cascades entre émetteurices différentes |
| `emetteur_id` | Pseudonyme stable après anonymisation (ex. `[user_ref_1]`) — pour la pondération par profil (§ 9.2) |
| `notes` | Cas limites, cascade, désaccords |

---

## 7. Procédure d'annotation (pilote 300 messages)

1. Chaque annotateurice annote indépendamment les 300 messages (fichier `data/pilot_sample.csv`, colonnes `label_morgane` / `label_peng` ou équivalent).
2. Calculer κ de Cohen (script ou tableur) — seuil ≥ 0,70 (`eval_protocol.md`).
3. Lister les désaccords ; réunion de consensus sur les cas ambigus.
4. Réviser ce guide si κ < 0,70.
5. Fusionner en colonne `label` (vérité terrain) pour l'entraînement / l'éval.
6. Réunion annotateurices : trancher les fonctionnalités § 9 (cascade, pondération profils) et mettre à jour ce guide en v1.2 si besoin.
7. Vérifier la **diversité des contextes** dans le pilote (pas uniquement un seul type de canal), sans chercher une taxonomie exhaustive.

---

## 8. Exemples commentés

### Exemple A — CRITIQUE (message complet — rare)

> Texte : « Le partiel INFO est reporté au 22/06 à 14h en amphi B. »  
> Label : `1`  
> Sous-étiquettes : `deadline|impact`  
> Motif : changement officiel avec date, heure et lieu dans un seul message. Ce format est exceptionnel dans les flux réels ; ne pas en faire la norme pour exiger un `1`. La plupart des cas ressemblent plutôt à l'exemple G (cascade).

### Exemple G — CRITIQUE (cascade, même émetteurice — cas fréquent)

> Séquence (même `cascade_id`, même émetteurice, < 15 min) :  
> 1. « Le partiel INFO est décalé. » → `label` : `1`, `cascade`  
> 2. « Le 22 juin. » → `label` : `0` ou `1`, `cascade` (fragment ; critique en séquence)  
> 3. « 14h, amphi B. » → `label` : `1`, `cascade|deadline`  
> Motif : aucun message n'est aussi complet que l'exemple A, mais l'ensemble équivaut à une notification critique. Crible doit reconstruire cette info (§ 9.1).

### Exemple H — CRITIQUE (cascade par réponse d'une autre émetteurice)

> Séquence (même `cascade_id`, `reply_to_id` sur le message 2) :  
> 1. [user_A] « Le partiel est décalé, je vous envoie les détails. » → `label` : `1`, `cascade`  
> 2. [user_B] *(réponse au msg 1)* « C'est le 22/06 à 14h, amphi B. » → `label` : `1`, `cascade|deadline`, `reply_to_id` = id du msg 1  
> Motif : la précision vient d'une autre personne via une reply ; l'info critique est dans la paire, pas dans le message 1 seul. À agréger comme une cascade (§ 9.1).

### Exemple B — NON_CRITIQUE

> Texte : « 😂😂😂 » (40 réactions)  
> Label : `0`  
> Motif : engagement social sans information nouvelle.

### Exemple C — CRITIQUE (action)

> Texte : « @tous Merci d'envoyer votre choix de sujet avant vendredi 18h. »  
> Label : `1`  
> Sous-étiquettes : `deadline|action`

### Exemple D — CRITIQUE (engagement + contenu)

> Texte : « La séance de demain est annulée » + 25 réponses « merci » / questions  
> Label : `1`  
> Sous-étiquettes : `impact|social_engagement`  
> Motif : l'annulation est l'info critique ; les réactions confirment l'importance perçue.

### Exemple E — NON_CRITIQUE (faux positif engagement)

> Texte : « Qui a vu le dernier épisode ? » + 60 réactions  
> Label : `0`  
> Motif : viral mais sans impact sur les obligations du groupe.

### Exemple F — CRITIQUE (date sans heure — cas à conserver)

> Texte : « Examen décalé au 15 » (sans heure ni lieu précisés)  
> Label : `1`  
> Sous-étiquettes : `deadline|impact`  
> Motif : changement d'examen = information très importante pour l'utilisateurice ; l'absence d'heure ne diminue pas la criticité. Ce cas doit rester annoté `1` dans le corpus pilote et en production.

---

## 9. Fonctionnalités à concevoir — *à discuter*

Ces points ne sont pas figés : les deux annotateurices doivent en discuter (réunion de cadrage, ~30–45 min), noter les décisions ci-dessous, puis les reporter dans le README / le skill OpenClaw / le code.

### 9.1. Agrégation des informations en cascade

**Besoin** : détecter qu'une info critique se répartit sur plusieurs messages consécutifs (même fil, même émetteurice ou fil de réponses) et notifier une fois l'info reconstituée, plutôt que d'ignorer chaque fragment ou de spammer une alerte par message.

**Pistes techniques (à valider en équipe)** :

- Fenêtre glissante de N messages ou T minutes sur un canal ;
- Regroupement par `cascade_id` heuristique : proximité temporelle + mots-clés liés et/ou chaîne de réponses (`reply_to_id` / fil de discussion), y compris si l'émetteurice change entre le message parent et la reply ;
- Décision finale par le LLM (OpenClaw) sur le bloc agrégé, pas seulement sur le dernier message ;
- Une seule notification synthétique : « Partiel INFO reporté au 22/06 à 14h, amphi B (reconstitué à partir de 3 messages). »

**Questions pour la discussion** :

- Fenêtre temporelle : 5, 15 ou 30 minutes ?
- Notifier au 1er message suspect ou attendre la fin de la cascade ?
- Comment annoter les cascades dans le pilote pour entraîner / évaluer cette fonction ?

| Décision (à remplir) | Choix retenu | Date |
|----------------------|--------------|------|
| Fenêtre cascade | | |
| Stratégie de notification | | |
| Priorité implémentation (v0 / v1 / v2) | | |

### 9.2. Pondération par profil émetteurice (apprentissage automatique) → *à discuter*

**Besoin** : certains profils envoient souvent des messages pertinents (enseignant, référent admin), d'autres plutôt du bruit social. Crible pondère le score de criticité selon l'historique de l'émetteurice sur le canal.

**Orientation TAL** : la pondération est apprise automatiquement à partir du corpus annoté (pilote puis corpus complet) — pas de liste manuelle de contacts « prioritaires ». C'est cohérent avec les baselines et le protocole d'évaluation (`eval_protocol.md`) : features + modèle (ex. score de pertinence par `emetteur_id` appris par régression ou statistiques agrégées sur les labels).

**Comportement visé** :

- Émetteurice à forte pertinence historique (apprise) → seuil d'alerte légèrement abaissé (favoriser le rappel) ;
- Émetteurice surtout « bruit » (apprise) → seuil relevé sauf signal fort (deadline explicite, engagement exceptionnel + contenu) ;
- Nouvelle émetteurice ou peu de messages : score neutre (pas de pénalité ni de bonus avant observation suffisante).

**Pistes techniques** :

- Features par (`emetteur_id`, `canal`) : taux de messages labelisés `1`, précision historique, volume, récence ;
- Apprentissage sur le train set ; évaluation sur val/test (pas de fuite) ;
- Intégration : facteur multiplicatif ou offset sur le score heuristique / LLM ; ou feature supplémentaire dans un classifieur (CamemBERT + métadonnées émetteurice).

**Précautions** :

- Ne pas créer de biais systémique : priorité au score neutre tant que l'historique est insuffisant (seuil minimal de messages observés) ;
- Transparence pour l'utilisateurice (option « réinitialiser la confiance » par contact/canal) ;
- RGPD : stocker un score agrégé appris, pas l'historique brut ré-identifiable (voir § 10.6).

**Paramètres à discuter** :

- Périmètre des features : par canal, par émetteurice globale, ou les deux ?
- Seuil minimal d'observations avant d'appliquer une pondération non neutre ?
- Métrique cible pour l'apprentissage : taux de vrais positifs historiques, F1 par émetteurice, autre ?

| Décision (à remplir) | Choix retenu | Date |
|----------------------|--------------|------|
| Mode | Automatique (apprentissage sur pilote / train) — *non négociable pour le volet TAL* |
| Périmètre des features (`canal` / global / les deux) | | |
| Seuil minimal de messages par `emetteur_id` | | |
| Métrique de pertinence profil apprise | | |
| Priorité implémentation (v0 / v1 / v2) | | |

---

## 10. Précautions RGPD

Ce projet traite des données personnelles (contenus de messages, pseudos, métadonnées). Les annotateurices et le développement doivent appliquer les principes du RGPD dès la collecte — pas seulement au moment du commit git.

### 10.1. Principes à respecter

| Principe | Application concrète pour Crible |
|----------|----------------------------------|
| Finalité | Données utilisées uniquement pour la recherche / le projet master (annotation, entraînement, évaluation) — pas pour un autre usage (marketing, profilage commercial, etc.). |
| Minimisation | Ne collecter que les champs utiles au CSV (texte, canal, réactions, ids techniques). Pas d’export « au cas où » de profils complets ou de pièces jointes non nécessaires. |
| Limitation de conservation | Supprimer les exports bruts dès l’anonymisation validée ; ne pas garder de copies personnelles hors du cadre du projet. |
| Intégrité et confidentialité | Accès restreint au dépôt et aux dossiers `data/` (équipe projet + encadrant·e si applicable). Mots de passe, dépôt privé, pas de partage public des CSV. |
| Responsabilité | Documenter qui collecte, qui annote, où sont stockées les données ; en cas de doute, demander à l’encadrant·e du master avant d’élargir le périmètre. |

### 10.2. Base légale et consentement

- Privilégier des canaux où les participantes et participants savent que les messages peuvent être utilisés à des fins pédagogiques (groupe de promo, serveur de cours), ou obtenir un accord explicite de l’administrateurice du canal / de la promotion.
- Éviter d’exporter des conversations privées (DM) ou des groupes sans lien avec le projet sans base légale claire.
- Respecter les conditions d’utilisation des plateformes (Discord, WhatsApp, etc.) : certains exports automatisés ou bots peuvent être interdits — vérifier avant branchement OpenClaw.

### 10.3. Anonymisation (obligatoire avant commit et avant partage)

- Remplacer noms, prénoms, pseudos identifiables par `[user]` ou pseudonymes non réversibles (`[user_enseignant]`, pas « Martin »).
- Masquer numéros de téléphone, e-mails, liens personnels, photos, identifiants de compte réels.
- Vérifier le champ `notes` des annotateurices : aucune info permettant de ré-identifier une personne.
- Ne **jamais** committer d’exports bruts non anonymisés (`data/raw/` est ignoré par git ; garder les bruts hors dépôt si indispensable, puis les détruire après anonymisation).

**Script fourni** : `scripts/run_anonymize.py` (complément manuel obligatoire : la regex ne couvre pas tout).

### 10.4. Annotation et jeu de données

- `emetteur_id` : identifiant pseudonyme stable pour l’étude, pas le pseudo affiché tel quel s’il est identifiable.
- Ne pas publier de captures d’écran avec noms ou avatars visibles dans le mémoire sans floutage.
- Partage du pilote entre les deux annotateurices : canal chiffré ou dépôt privé ; pas d’envoi par messagerie personnelle non sécurisée si évitable.

### 10.5. Traitement par LLM (OpenClaw / APIs)

- Si les messages passent par une API cloud (OpenAI, Anthropic, etc.) : les données quittent votre infrastructure — minimiser le volume envoyé, lire les politiques du fournisseur, privilégier l’auto-hébergement OpenClaw + modèle local lorsque possible.
- Ne pas coller dans des outils publics (ChatGPT web, etc.) des extraits non anonymisés du corpus.
- Logs gateway / OpenClaw : désactiver ou limiter la conservation des contenus si la configuration le permet.

### 10.6. Fonctionnalités du projet (cascade, pondération profil)

- Pondération par émetteurice (§ 9.2) : ne stocker qu’un score agrégé (pertinence historique), pas l’historique brut ré-identifiable des messages.
- Cascade : les blocs agrégés envoyés à la notification ne doivent pas recopier de données personnelles inutiles.

### 10.7. En cas d’incident ou de demande

- Fuite ou commit accidentel de données brutes : retirer le commit du dépôt distant, prévenir l’encadrant·e, documenter l’incident.
- Demande d’effacement : si une personne identifiable apparaît malgré l’anonymisation, supprimer ou corriger les lignes concernées dans le corpus et les dérivés (modèles ré-entraînés si nécessaire).
- Tenir une liste à jour des personnes ayant accès aux données du pilote.

---

## 11. Révisions

Une ligne n’est ajoutée ici qu’après commit (même règle que le numéro en tête de document).

| Version | Commit / date | Changements |
|---------|---------------|-------------|
| 1.0 | 2026-05-22 | Première version : critères CRITIQUE/NON_CRITIQUE, sous-étiquettes, messages complets rares, cascades (enchaînement + replies), colonnes CSV, exemples A–H, § 9 (agrégation + pondération auto), § 10 RGPD |
| 1.1 | 2026-05-22 | Recentrage TAL/IA : généralisation et automatisation (pas de taxonomie exhaustive) ; signaux transférables ; renumérotation sections ; alignement `data/README.md` |
