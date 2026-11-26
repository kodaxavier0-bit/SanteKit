# Scénarios de test — SantéKit (API et application)

Ce document recense les scénarios fonctionnels et techniques à valider avant la démonstration finale. Il est sous la responsabilité de **ONLA LOOGA Gertrude Rose Lynne** (tests et déploiement).

## 1. Scénarios fonctionnels (formulaire → API → résultat)

| # | Scenario | Entrées (symptômes) | Résultat attendu | Critère de succès |
|---|----------|---------------------|------------------|-------------------|
| TF-01 | Adulte, fièvre légère, 1 jour | age=34, fievre=true, temperature=38.6, toux=true, duree_jours=1, difficulte_respiratoire=false, douleur_poitrine=false, vomissements_diarrhee=false, maux_tete_intenses=false | niveau = **faible**, couleur = vert | Résultat affiché en < 3 s, avertissement médical présent |
| TF-02 | Enfant, forte fièvre, toux, 3 jours | age=7, fievre=true, temperature=39.2, toux=true, duree_jours=3, difficulte_respiratoire=false, douleur_poitrine=false, vomissements_diarrhee=false, maux_tete_intenses=false | niveau = **moyen**, couleur = orange | Résultat affiché en < 3 s, texte "Urgence moyenne" doublé de la couleur orange |
| TF-03 | Difficulté respiratoire + douleur poitrine | age=45, fievre=true, temperature=38.0, toux=false, duree_jours=2, difficulte_respiratoire=true, douleur_poitrine=true, vomissements_diarrhee=false, maux_tete_intenses=false | niveau = **élevé**, couleur = rouge | Résultat affiché en < 3 s, texte "Urgence élevée" doublé de la couleur rouge |
| TF-04 | Formulaire incomplet | Champ manquant (ex: age absent) | Message de validation, aucun envoi | Aucun appel API, message d'erreur en langage simple |
| TF-05 | Âge négatif | age=-5 | Message de validation, aucun envoi | Code 400 côté API, aucun résultat inventé |
| TF-06 | API éteinte / réseau coupé | Toutes valides | Message d'erreur clair, aucun résultat inventé | L'application affiche une erreur, pas de niveau d'urgence factice |
| TF-07 | Historique conservé | Après une évaluation, redémarrer l'app | L'évaluation apparaît dans l'historique | SQLite persiste après redémarrage |
| TF-08 | Centres de santé hors connexion | Aucun réseau requis | Liste des centres affichée | Données lues depuis SQLite local |

## 2. Scénarios API (Postman / curl)

| # | Scenario | Requête | Réponse attendue | Code HTTP |
|---|----------|---------|------------------|-----------|
| TA-01 | Vérification santé | `GET /health` | `{ "status": "ok" }` ou équivalent | 200 |
| TA-02 | Prédiction valide → faible | `POST /predict` (adulte, fièvre 1 jour) | `{ "niveau": "faible", ... }` | 200 |
| TA-03 | Prédiction valide → moyen | `POST /predict` (enfant, fièvre + toux, 3 jours) | `{ "niveau": "moyen", ... }` | 200 |
| TA-04 | Prédiction valide → élevé | `POST /predict` (respiration + poitrine) | `{ "niveau": "élevé", ... }` | 200 |
| TA-05 | Champ manquant | `POST /predict` sans `age` | `{ "erreur": "..." }` | 400 |
| TA-06 | Âge négatif | `POST /predict` avec `age: -5` | `{ "erreur": "..." }` | 400 |
| TA-07 | Type invalide | `POST /predict` avec `age: "trente"` | `{ "erreur": "..." }` | 400 |

## 3. Critères de succès généraux

- Le temps de réponse de l'API est inférieur à 3 secondes pour une requête valide.
- L'avertissement « Ceci n'est pas un avis médical » apparaît sur l'écran d'accueil et sur chaque écran de résultat.
- La couleur du résultat est toujours doublée d'un libellé textuel (jamais la couleur seule).
- En cas d'erreur ou d'API injoignable, aucun résultat n'est inventé.
- Le code secompile et l'application se lance sans plantage.
- L'historique et les centres de santé fonctionnent hors connexion.

## 4. Comment exécuter les tests

### Avec curl (local)

```bash
# Santé
curl http://localhost:5000/health

# Prédiction valide
curl -X POST http://localhost:5000/predict \
  -H "Content-Type: application/json" \
  -d '{"age":34,"fievre":true,"temperature":38.6,"toux":true,"duree_jours":3,"difficulte_respiratoire":false,"douleur_poitrine":false,"vomissements_diarrhee":false,"maux_tete_intenses":false}'

# Erreur 400 (âge négatif)
curl -X POST http://localhost:5000/predict \
  -H "Content-Type: application/json" \
  -d '{"age":-5,"fievre":true,"temperature":39.0,"toux":false,"duree_jours":2,"difficulte_respiratoire":false,"douleur_poitrine":false,"vomissements_diarrhee":false,"maux_tete_intenses":false}'
```

### Avec Postman

Importer la collection fournie dans `docs/tests/postman_collection.json`. Configurer l'environnement avec l'URL de l'API (localhost ou URL Render/PythonAnywhere).

### Sur l'application Android

1. Ouvrir l'écran Accueil → vérifier l'affichage de l'avertissement médical.
2. Remplir le formulaire selon les scénarios TF-01 à TF-04.
3. Vérifier l'écran Résultat (niveau, couleur, texte, avertissement).
4. Vérifier l'écran Historique après redémarrage.
5. Vérifier l'écran Centres de santé en mode avion (hors connexion).

## 5. Responsabilité

Ce document est sous la responsabilité de **ONLA LOOGA Gertrude Rose Lynne**. Toute mise à jour doit être proposée via une Pull Request et validée par le chef de groupe.