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

## 3) Définition « message critique »

Un message est « critique » s'il satisfait au moins un des critères :

- **Urgence temporelle** : action requise rapidement.
- **Impact élevé** : conséquences importantes si non lu.
- **Action explicite** : demande claire adressée au destinataire/groupe.
- **Contexte prioritaire** : émetteur ou canal à forte priorité.

La définition exacte sera formalisée dans un guide d'annotation.

## 4) Stack IA recommandée (MVP → avancé)

### MVP (robuste et rapide)

- **Classification binaire** (`critique` vs `non critique`) avec un modèle type CamemBERT/FlauBERT.
- Features de contexte : auteur, canal, heure, présence de mots d'urgence, mentions.
- Calibration de seuil pour favoriser le rappel (ne pas rater les vrais critiques).

### Évolution

- Classification multi-classes (`urgence`, `deadline`, `action`, `incident`, etc.).
- Reranking par modèle plus lourd (cross-encoder/LLM) sur les top candidats.
- Boucle de feedback utilisateur (corriger « utile/pas utile ») pour adaptation continue.

### À propos d'OpenClaw

OpenClaw (https://openclaw.ai/) peut être exploré comme piste expérimentale, mais il faut d'abord vérifier :

- maturité écosystème,
- coût d'inférence,
- facilité d'intégration locale/cloud,
- performance comparée à des baselines simples.

La recommandation est de **benchmarker OpenClaw contre 2 baselines solides** avant décision finale.

## 5) Architecture fonctionnelle

1. **Ingestion** des messages (API/exports/connecteurs).
2. **Pré-traitement** (nettoyage, langue, segmentation, déduplication).
3. **Inférence** du score de criticité.
4. **Filtrage + seuil adaptatif** selon profil utilisateur.
5. **Notification** (push, email, webhook, bot Discord).
6. **Journalisation + feedback** pour amélioration continue.

## 6) Roadmap (12 semaines, équipe de 2)

### Phase 0 - Cadrage (S1)

- Formaliser problèmes, objectifs, contraintes (RGPD, ToS plateformes).
- Définir la taxonomie « critique ».
- Écrire le protocole d'évaluation.

**Livrables**

- Cahier des charges court.
- Guide d'annotation v1.
- Métriques cibles (précision, rappel, F1, taux d'alertes utiles).

### Phase 1 - Données (S2–S3)

- Constituer un jeu de données (historique anonymisé + données synthétiques).
- Annoter un premier corpus (minimum viable : 2k–5k messages).
- Mesurer accord inter-annotateurs (vous 2).

**Livrables**

- Dataset versionné (`train/val/test`).
- Rapport qualité annotations (kappa ou accord simple).

### Phase 2 - Baselines ML/NLP (S4–S5)

- Baseline 1 : règles + mots-clés + score heuristique.
- Baseline 2 : classifieur léger (TF-IDF + Logistic Regression).
- Baseline 3 : transformeur fine-tuné (CamemBERT/FlauBERT).

**Livrables**

- Tableau comparatif métriques + latence.
- Choix du modèle MVP justifié.

### Phase 3 - Prototype bout-en-bout (S6–S7)

- Pipeline ingestion → inférence → notification.
- Tableau de bord simple (logs, faux positifs/faux négatifs).
- Réglage du seuil utilisateur.

**Livrables**

- MVP exécutable.
- Démo sur flux réel/simulé.

### Phase 4 - Améliorations IA (S8–S9)

- Test OpenClaw (ou autre modèle avancé) en reranker.
- Ajout de contexte conversationnel (fenêtre de messages précédents).
- Début active learning via feedback utilisateur.

**Livrables**

- Rapport d'ablation (avec/sans contexte, avec/sans reranker).
- Décision sur stack finale.

### Phase 5 - Fiabilité et évaluation finale (S10–S11)

- Stress test sur gros volumes.
- Étude erreurs (types de messages ratés).
- Ajustement précision/rappel selon scénarios.

**Livrables**

- Rapport expérimental final.
- Recommandations de déploiement.

### Phase 6 - Rendu (S12)

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

1. Valider la taxonomie de criticité (30–45 min).
2. Définir le format de dataset et commencer l'annotation pilote (200 messages).
3. Entraîner baseline TF-IDF + baseline CamemBERT.
4. Fixer un objectif MVP : rappel ≥ 0,90 sur classe critique.
