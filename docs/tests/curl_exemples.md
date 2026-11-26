# Exemples de requêtes curl — API SantéKit

Ces exemples permettent de tester l'API Flask en ligne de commande. Remplacer `http://localhost:5000` par l'URL de déploiement (Render ou PythonAnywhere) pour les tests à distance.

## 1. Vérifier que l'API est en ligne

```bash
curl -i http://localhost:5000/health
```

Réponse attendue :
```
HTTP/1.1 200 OK
Content-Type: application/json

{"status":"ok"}
```

## 2. Prédiction valide — Niveau faible (vert)

Adulte avec fièvre légère depuis 1 jour.

```bash
curl -X POST http://localhost:5000/predict \
  -H "Content-Type: application/json" \
  -d '{
    "age": 34,
    "fievre": true,
    "temperature": 38.6,
    "toux": true,
    "duree_jours": 1,
    "difficulte_respiratoire": false,
    "douleur_poitrine": false,
    "vomissements_diarrhee": false,
    "maux_tete_intenses": false
  }'
```

Réponse attendue (200) :
```json
{
  "niveau": "faible",
  "couleur": "vert",
  "conseil": "...",
  "confiance": 0.XX,
  "avertissement": "Ceci n'est pas un avis médical."
}
```

## 3. Prédiction valide — Niveau moyen (orange)

Enfant avec forte fièvre, toux, depuis 3 jours.

```bash
curl -X POST http://localhost:5000/predict \
  -H "Content-Type: application/json" \
  -d '{
    "age": 7,
    "fievre": true,
    "temperature": 39.2,
    "toux": true,
    "duree_jours": 3,
    "difficulte_respiratoire": false,
    "douleur_poitrine": false,
    "vomissements_diarrhee": false,
    "maux_tete_intenses": false
  }'
```

## 4. Prédiction valide — Niveau élevé (rouge)

Difficulté à respirer avec douleur à la poitrine.

```bash
curl -X POST http://localhost:5000/predict \
  -H "Content-Type: application/json" \
  -d '{
    "age": 45,
    "fievre": true,
    "temperature": 38.0,
    "toux": false,
    "duree_jours": 2,
    "difficulte_respiratoire": true,
    "douleur_poitrine": true,
    "vomissements_diarrhee": false,
    "maux_tete_intenses": false
  }'
```

## 5. Erreur 400 — Âge négatif

```bash
curl -X POST http://localhost:5000/predict \
  -H "Content-Type: application/json" \
  -d '{
    "age": -5,
    "fievre": true,
    "temperature": 39.0,
    "toux": false,
    "duree_jours": 2,
    "difficulte_respiratoire": false,
    "douleur_poitrine": false,
    "vomissements_diarrhee": false,
    "maux_tete_intenses": false
  }'
```

Réponse attendue (400) :
```json
{
  "erreur": "age doit être un entier positif"
}
```

## 6. Erreur 400 — Champ manquant

```bash
curl -X POST http://localhost:5000/predict \
  -H "Content-Type: application/json" \
  -d '{
    "fievre": true,
    "temperature": 38.6
  }'
```

Réponse attendue (400) :
```json
{
  "erreur": "..."
}
```

## 7. Mesurer le temps de réponse

```bash
curl -o /dev/null -s -w "%{time_total}\n" -X POST http://localhost:5000/predict \
  -H "Content-Type: application/json" \
  -d '{"age":34,"fievre":true,"temperature":38.6,"toux":true,"duree_jours":3,"difficulte_respiratoire":false,"douleur_poitrine":false,"vomissements_diarrhee":false,"maux_tete_intenses":false}'
```

Le temps doit être inférieur à 3 secondes.

## Note importante

Aucun secret, clé ou mot de passe ne doit apparaître dans ces commandes ni dans aucun fichier commité.