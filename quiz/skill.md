# Format JSON des questions — guide de référence

Ce document décrit le format JSON attendu pour générer des questions de quiz. Donne ce document à une IA pour qu'elle génère un fichier `questions.json` valide.

## Structure générale

```json
{
  "questions": [
    { "question": "...", "type": "...", "data": { ... } }
  ]
}
```

Chaque question a 3 champs : `question` (texte affiché), `type` (un des 5 types ci-dessous) et `data` (structure spécifique au type).

---

## 1. `qcm_simple` — choix multiple, une seule bonne réponse

```json
{
  "question": "Quelle est la capitale de la France ?",
  "type": "qcm_simple",
  "data": {
    "choix": ["Paris", "Lyon", "Marseille"],
    "reponse": 0
  }
}
```
- `choix` : liste des options (texte).
- `reponse` : index (0-based) de la bonne option. **Un seul index.**
- Règle : au moins 2 choix, exactement 1 bonne réponse.

---

## 2. `qcm_multiple` — choix multiple, plusieurs bonnes réponses

```json
{
  "question": "Lesquels sont des langages front-end ?",
  "type": "qcm_multiple",
  "data": {
    "choix": ["HTML", "Python", "CSS", "C++"],
    "reponses": [0, 2]
  }
}
```
- `choix` : liste des options.
- `reponses` : liste des index (0-based) corrects.
- Règle : au moins 2 choix, au moins 1 bonne réponse (peut aller jusqu'à toutes les options).

---

## 3. `texte_libre` — réponse tapée librement

```json
{
  "question": "Formule chimique de l'eau ?",
  "type": "texte_libre",
  "data": {
    "reponses": ["H2O", "h2o"],
    "caseSensitive": false
  }
}
```
- `reponses` : liste des formulations acceptées (synonymes inclus).
- `caseSensitive` : `true` si la casse doit être respectée, sinon `false`.
- Règle : au moins 1 réponse acceptée.

---

## 4. `texte_a_trous` — texte à compléter

Le texte contient des marqueurs `{{1}}`, `{{2}}`, etc. Chaque marqueur correspond à une entrée du tableau `trous` (matching par `id`, pas par ordre d'apparition).

Deux modes possibles via le champ `mode` :

### Mode `"libre"` — l'utilisateur tape la réponse dans un champ

```json
{
  "question": "Complétez la phrase.",
  "type": "texte_a_trous",
  "data": {
    "mode": "libre",
    "texte": "Le {{1}} est la capitale de la {{2}}.",
    "trous": [
      { "id": 1, "reponses": ["Caire", "Le Caire"], "caseSensitive": false },
      { "id": 2, "reponses": ["Égypte"], "caseSensitive": false }
    ]
  }
}
```
- Chaque trou a : `id`, `reponses` (liste de formulations acceptées), `caseSensitive`.

### Mode `"menu_deroulant"` — l'utilisateur choisit dans une liste déroulante

```json
{
  "question": "Complétez la phrase.",
  "type": "texte_a_trous",
  "data": {
    "mode": "menu_deroulant",
    "texte": "La {{1}} de la Terre orbite autour du {{2}}.",
    "trous": [
      { "id": 1, "options": ["Lune", "Étoile", "Comète"], "reponse": "Lune" },
      { "id": 2, "options": ["Soleil", "Mars", "Jupiter"], "reponse": "Soleil" }
    ]
  }
}
```
- Chaque trou a : `id`, `options` (liste affichée dans le menu déroulant, doit inclure la bonne réponse), `reponse` (la valeur correcte parmi `options`).

Règle commune : chaque `id` utilisé dans `texte` doit exister dans `trous`, et inversement. Au moins 1 trou.

---

## 5. `ordre` — remettre des éléments dans l'ordre

```json
{
  "question": "Remettez les couches du modèle OSI dans l'ordre, de la couche 1 à la couche 7.",
  "type": "ordre",
  "data": {
    "elements": [
      "Physique", "Liaison", "Réseau", "Transport",
      "Session", "Présentation", "Application"
    ]
  }
}
```
- `elements` : liste ordonnée — **l'ordre dans le JSON est l'ordre correct**.
- À l'affichage, les éléments sont mélangés ; l'utilisateur les réordonne avec des boutons haut/bas.
- Règle : au moins 2 éléments.

---

## Checklist pour une IA qui génère des questions

- Le fichier racine est `{ "questions": [...] }`.
- Chaque question a `question`, `type`, `data`.
- `type` est exactement l'une de : `qcm_simple`, `qcm_multiple`, `texte_libre`, `texte_a_trous`, `ordre`.
- Pour `qcm_simple` : un seul `reponse` (entier).
- Pour `qcm_multiple` : `reponses` (tableau d'entiers, non vide).
- Pour `texte_libre` : `reponses` (tableau de chaînes, non vide) + `caseSensitive` (booléen).
- Pour `texte_a_trous` : `mode` (`"libre"` ou `"menu_deroulant"`), `texte` avec marqueurs `{{id}}`, et `trous` cohérents avec ces marqueurs.
- Pour `ordre` : `elements` (tableau de chaînes, ≥ 2), dans l'ordre correct.
