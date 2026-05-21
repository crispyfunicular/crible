# Protocole d'Évaluation — Crible
## 1. Jeu de données et contrôle qualité
### 1.1. Constitution et découpage du corpus
Le corpus cible (3 500 à 6 000 messages) sera intégralement constitué d'historiques réels anonymisés (e.x. Discord, WhatsApp). La collecte pourra cibler en priorité des canaux à forte densité d'informations, afin d'assurer une présence suffisante de messages critiques dans l'échantillon.

Le découpage est figé ainsi : Train Set (70%) / Validation Set (15%) / Test Set (15%).

RGPD : Nettoyage automatique via Regex (remplacement des noms et téléphones par [user]).

### 1.2. Annotation et accord inter-annotateurs

La tâche principale est une classification binaire : **1 = CRITIQUE**, **0 = NON_CRITIQUE**. Chaque message critique reçoit également une ou plusieurs sous-étiquettes : `urgence`, `deadline`, `action`, `impact`, `social_engagement`.

Un échantillon pilote de 300 messages est annoté indépendamment par les deux membres de l’équipe. L’accord est mesuré par le **$\kappa$ de Cohen** :

$$\kappa = \frac{p_o - p_e}{1 - p_e}$$

Un seuil de **$\kappa \ge 0,70$** est requis avant de passer à l’annotation à grande échelle. En deçà, le guide d’annotation est révisé et les désaccords sont résolus par consensus.


## 2. Métriques d’évaluation (TAL & UX)

### 2.1. Métriques de classification 
Dans un flux de messagerie, les messages critiques sont par nature rares par rapport au bruit global. L'accuracy n'est donc pas un indicateur fiable. On se base plutôt sur la matrice de confusion classique (VP, FP, FN, VN) et trois métriques clés :

|  | Prédit CRITIQUE | Prédit NON_CRITIQUE |
| --- | --- | --- |
| Réel CRITIQUE | VP  | FN  |
| Réel NON_CRITIQUE | FP  | VN |

- Rappel (Recall) — Priorité absolue (Cible >= 90%) : Pour ne rater aucune info critique.

$$\text{Recall} = \frac{VP}{VP + FN}$$

- Précision (Cible >= 60%) : Pour limiter les fausses alertes et ne pas troller l'utilisateur.

$$\text{Precision} = \frac{VP}{VP + FP}$$

- F2-Score — On choisit le F2 plutôt que le F1 pour pénaliser les faux négatifs deux fois plus sévèrement que les faux positifs ("mieux vaut une alerte de trop qu'une info manquée").

$$F_2 = 5 \cdot \frac{\text{Precision} \cdot \text{Recall}}{4 \cdot \text{Precision} + \text{Recall}}$$

Remarque : Ces scores sont aussi calculés par sous-catégorie (urgence, deadline, etc.) pour repérer les angles morts des modèles.

### 2.2. Métriques d'ingénierie (hors-ligne ; évoluera si déploiement stream)
Pour que l'outil soit utilisable au quotidien, le modèle ne doit pas juste être précis, il doit aussi être rapide et efficace :

- Taux de réduction du bruit (Objectif >= 85%) : Pourcentage de messages inutiles bloqués par le système.

- - Formule simple : (Messages totaux - Messages notifiés) / Messages totaux

- Latence (Temps de réponse) : Temps écoulé entre l'arrivée d'un message et l'envoi de la notification. 

---

## 3. Évaluation Comparative et Étude d’Ablation

### 3.1. Phases d’évaluation

- Étape 1 : Évaluation du LLM pur (Prompt-only) sur les 300 messages pilotes dès que l'annotation est prête.

- Étape 2 : Test et benchmark des modèles NLP classiques (Regex, TF-IDF, CamemBERT...) pour voir s'ils peuvent rivaliser ou servir de pré-filtres.

- Étape 3 : Évaluation de l'approche hybride (Modèle NLP + LLM) pour optimiser la latence et les coûts avant validation finale.



### 3.2. Matrice comparative des modèles (Template)

*Les scores réels seront loggés au fil de l'eau dans le fichier `experiment_logs.md`.*

| Modèle / Architecture | Rappel | Précision | F2-Score | Latence | Coût |
| --- | --- | --- | --- | --- | --- |
| **1. LLM API (Prompt-only / v0)** |  |  |  |  |  |
| **2. Heuristiques / Regex** *(Baseline)* |  |  |  |  |  |
| **3. TF-IDF + Régression Logistique** *(Baseline)* |  |  |  |  |  |
| **4. Modèle local fine-tuné** *(ex: CamemBERT...)* |  |  |  |  |  |
| **5. Approche Hybride (Modèle local + LLM)** |  |  |  |  |  |


### 3.3. Étude d’ablation

Trois configurations seront comparées pour isoler la contribution de chaque composant :

* **Configuration A** : OpenClaw *Prompt-only* (Zéro-shot, sans règles métier).
* **Configuration B** : Configuration A + Structuration du skill (Seuils de priorité, métadonnées émetteurs/canaux).
* **Configuration C** : Configuration B + Pré-filtre léger classique (Modèle NLP isole les candidats, le LLM valide).

---
## 4. Analyse des erreurs (Qualitative en fin de projet)
Une analyse rapide sera menée sur le modèle final pour classer les erreurs restantes en deux catégories :
- Faux positifs (Surnotification) : Le système a notifié un message inutile.
- Faux négatifs (Messages manqués)** : Le système a bloqué un message important.
---

## 5. Objectifs de performance (Pour validation du MVP)
Pour que l'outil soit considéré comme validé et prêt pour la démonstration finale, le modèle retenu doit impérativement atteindre les seuils suivants :
- Rappel : $\ge 0,90$ (Sécurité : interdiction de rater une info critique).
- Précision : $\ge 0,60$ (Confort : pas plus de 40% de fausses alertes).
- F2-Score : $\ge 0,82$ (Équilibre global requis).
- Latence 

