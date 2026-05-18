# Crible

Outil d'extraction d'informations critiques dans un flux de messagerie mis en sourdine.

## 1) Vision du projet

Construire un assistant qui repère automatiquement les messages vraiment importants (urgence, deadline, action demandée, information critique) dans des canaux bruyants (Discord, WhatsApp, etc.) et notifie uniquement ces messages.

Objectif principal : **réduire le bruit sans rater les informations critiques**.

## 2) Cas d'usage cibles

- Message urgent d'un enseignant dans un groupe muet.
- Changement de date/heure d'examen, rendu, réunion.
- Mention directe d'une action demandée (« peux-tu envoyer... », « il faut valider... »).
- Message à impact élevé (problème technique bloquant, incident, annulation).
- Message suscitant beaucoup de réactions (likes, emojis, réponses) : signal social de pertinence, exploitable en premier temps comme heuristique simple (*quid* des plaisantiers (blagues, memes) virales ?).

## 3) Définition « message critique »

Un message est « critique » s'il satisfait au moins un des critères :

- **Urgence temporelle** : action requise rapidement.
- **Impact élevé** : conséquences importantes si non lu.
- **Action explicite** : demande claire adressée au destinataire/groupe.
- **Contexte prioritaire** : émetteur ou canal à forte priorité.
- **Engagement social** (v1) : nombre élevé de réactions ou de réponses par rapport au fil habituel du canal.

La définition exacte sera formalisée dans un guide d'annotation.

## 4) Stack technique : OpenClaw

**OpenClaw** ([openclaw.ai](https://openclaw.ai/)) est le socle retenu pour Crible : assistant IA personnel open source, auto-hébergé, avec intégrations natives aux messageries (Discord, WhatsApp, Telegram, Slack, Signal, iMessage).

### Pourquoi OpenClaw pour ce projet

- **Connexion aux flux** : ingestion et gestion des messages déjà prises en charge (sessions, debounce, historique de groupe).
- **Raisonnement contextuel** : l'agent exploite le fil de conversation, l'émetteur et le canal pour juger la pertinence d'un message.
- **Skills extensibles** : possibilité d'ajouter un skill « Crible » (règles, outils, prompts) dédié à la détection de criticité.
- **Notifications ciblées** : alertes via le canal de l'utilisateur (Telegram, Discord, etc.) sans réveiller tout le groupe muet.
- **Mémoire et feedback** : persistance du contexte et boucle « utile / pas utile » pour affiner le comportement.

### Rôle de Crible dans OpenClaw

Crible n'est pas un second bot parallèle : c'est la **couche de filtrage intelligent** branchée sur OpenClaw :

1. Messages entrants sur canaux/groups en sourdine → OpenClaw les reçoit.
2. Le skill Crible évalue la criticité (prompt + règles, puis affinage).
3. Si critique → notification dédiée ; sinon → silence (`NO_REPLY` / pas d'alerte).

Documentation utile : [Messages](https://documentation.openclaw.ai/concepts/messages), [Agent](https://documentation.openclaw.ai/cli/agent), [Configuration](https://documentation.openclaw.ai/gateway/configuration).

### Compléments NLP (évaluation académique)

Pour le mémoire et la comparaison scientifique, conserver des **baselines classiques** en parallèle (sans remplacer OpenClaw) :

- heuristiques + mots-clés + seuil d'engagement (réactions/réponses) ;
- TF-IDF + régression logistique ;
- CamemBERT/FlauBERT fine-tuné sur corpus annoté.

Ces modèles servent à mesurer précision/rappel/F1 sur jeu de test ; OpenClaw reste le prototype opérationnel.

### Évolution prévue

- v0 : skill OpenClaw + définition de criticité dans le prompt système.
- v1 : skill dédié avec seuils, liste d'émetteurs/canaux prioritaires, types (`urgence`, `deadline`, `action`).
- v2 : pré-filtre léger (classifieur) + décision finale OpenClaw sur les candidats ; active learning via feedback.

## 5) Architecture fonctionnelle

1. **Ingestion** via OpenClaw (canaux Discord, WhatsApp, etc.).
2. **Contexte** : session, historique de groupe, métadonnées émetteur/canal.
3. **Skill Crible** : évaluation criticité (LLM + règles configurables).
4. **Filtrage** : seuil adaptatif, priorités par canal/émetteur.
5. **Notification** : alerte utilisateur sur canal choisi (sans spammer le groupe muet).
6. **Journalisation + feedback** pour amélioration continue et évaluation.

## 6) Proposition de roadmap (12 semaines, équipe de 2)

### Phase 0 : cadrage (S1)

- Formaliser problèmes, objectifs, contraintes (RGPD, ToS plateformes).
- Définir la taxonomie « critique ».
- Installer et configurer OpenClaw (canal pilote : Discord ou WhatsApp).
- Écrire le protocole d'évaluation.

**Livrables**

- Cahier des charges court.
- Guide d'annotation v1.
- Métriques cibles (précision, rappel, F1, taux d'alertes utiles).

### Phase 1 : données (S2–S3)

- Constituer un jeu de données (historique anonymisé + données synthétiques).
- Annoter un premier corpus (minimum viable : 2k–5k messages).
- Mesurer accord inter-annotateurs (vous 2).

**Livrables**

- Dataset versionné (`train/val/test`).
- Rapport qualité annotations (kappa ou accord simple).

### Phase 2 : skill Crible sur OpenClaw (S4–S5)

- Premier skill : prompt système + critères de criticité.
- Tests sur flux réel ou export simulé (groupes muets).
- Réglage silence vs notification (`NO_REPLY`, canal d'alerte dédié).

**Livrables**

- Skill Crible v0 fonctionnel.
- Jeu d'exemples annotés pour calibrer le comportement.

### Phase 3 : baselines NLP + prototype bout-en-bout (S6–S7)

- Baselines pour le mémoire : heuristiques, TF-IDF, CamemBERT/FlauBERT.
- Pipeline OpenClaw : ingestion → skill Crible → notification.
- Tableau de bord simple (logs, faux positifs/faux négatifs).

**Livrables**

- MVP exécutable.
- Démo sur flux réel/simulé.

### Phase 4 : améliorations (S8–S9)

- Skill Crible v1 : types de criticité, priorités canal/émetteur.
- Option pré-filtre classique + décision OpenClaw sur candidats.
- Active learning via feedback « utile / pas utile ».
- Comparaison OpenClaw vs baselines sur corpus annoté.

**Livrables**

- Rapport d'ablation (prompt seul vs skill structuré vs pré-filtre).
- Métriques OpenClaw + baselines sur jeu de test.

### Phase 5 : fiabilité et évaluation finale (S10–S11)

- Stress test sur gros volumes.
- Étude erreurs (types de messages ratés).
- Ajustement précision/rappel selon scénarios.

**Livrables**

- Rapport expérimental final.
- Recommandations de déploiement.

### Phase 6 : rendu (S12)

- Nettoyage code + documentation.
- Slides et démo finale.
- Roadmap post-master (industrialisation).

**Livrables**

- Dépôt propre, reproductible.
- Mémoire/rapport + support de présentation.

## 7) Métriques à suivre

- **Rappel critique** (priorité #1) : ne pas rater les messages critiques.
- **Précision** : limiter les fausses alertes.
- **F1 critique**.
- **Latence d'inférence** (temps avant notification).
- **Alertes utiles/jour** (métrique orientée usage réel).

## 8) Risques et parades

- **Données sensibles** → anonymisation stricte + minimisation des logs.
- **Label noise** → guide annotation + revues croisées.
- **Dérive de domaine** (Discord ≠ WhatsApp) → évaluation multi-domaines.
- **Trop de faux positifs** → seuil adaptatif + feedback utilisateur.

## 9) Prochaines actions immédiates

1. Installer OpenClaw et connecter un canal pilote (Discord ou WhatsApp).
2. Valider la taxonomie de criticité (30–45 min).
3. Rédiger le prompt / skill Crible v0 et tester sur 20–30 messages réels.
4. Lancer l'annotation pilote (200 messages) pour les baselines du mémoire.
5. Fixer un objectif MVP : rappel ≥ 0,90 sur messages critiques (évalués manuellement).
