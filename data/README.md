# Données — Crible

## Structure

```
data/
├── README.md           # ce fichier
├── pilot_sample.csv    # échantillon pilote (template + exemples)
└── raw/                # exports bruts (NON versionnés — voir .gitignore)
```

## Format `pilot_sample.csv`

| Colonne | Type | Description |
|---------|------|-------------|
| `id` | entier | Identifiant unique du message |
| `texte` | texte | Contenu **anonymisé** du message |
| `canal` | texte | Ex. `discord#groupe-tal`, `whatsapp#equipe-foot` |
| `timestamp` | ISO 8601 | Date/heure du message (optionnel en pilote) |
| `nb_reactions` | entier | Nombre de réactions (likes, emojis) |
| `nb_replies` | entier | Nombre de réponses directes au message |
| `label` | 0 ou 1 | Vérité terrain après consensus (`CRITIQUE` / `NON_CRITIQUE`) |
| `sous_labels` | texte | Codes séparés par `\|` (vide si `label=0`) |
| `cascade_id` | texte | Identifiant de séquence (ex. `cascade_042`) si message en cascade — voir `docs/annotation_guide.md` § 4.4 |
| `reply_to_id` | entier | Id du message parent si ce message est une **réponse** dans le chat (cascade possible entre émetteurices différentes) |
| `emetteur_id` | texte | Pseudonyme stable anonymisé (pondération profil, § 9.2) |
| `label_morgane` | 0/1/vide | Annotation indépendante Morgane |
| `label_peng` | 0/1/vide | Annotation indépendante Peng |
| `notes` | texte | Commentaire libre (cas limites, cascade) |

Voir `docs/annotation_guide.md` pour les règles d'annotation.

## Objectif pilote

**300 messages** minimum pour mesurer κ ≥ 0,70 (`eval_protocol.md`).
