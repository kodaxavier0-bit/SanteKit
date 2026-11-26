# Déploiement de l'API Flask — SantéKit

Ce document décrit l'installation, la configuration et le déploiement de l'API Flask du projet SantéKit. Il est destiné à l'équipe et aux testeurs (Postman, curl).

## 1. Prérequis

- Python 3.9 ou supérieur
- `pip` et `venv`
- Accès au dépôt Git du groupe

## 2. Installation en local

```bash
# Cloner le dépôt
git clone https://github.com/kodaxavier0-bit/SanteKit.git
cd SanteKit

# Créer et activer un environnement virtuel
python -m venv venv
source venv/bin/activate   # Linux/macOS
# ou venv\Scripts\activate # Windows

# Installer les dépendances (versions figées dans requirements.txt)
pip install -r api/requirements.txt
```

## 3. Lancer l'API en local

```bash
cd api
python app.py
```

L'API est alors disponible sur `http://localhost:5000`.

Vérifier que l'API répond :
```bash
curl http://localhost:5000/health
```

## 4. Déploiement sur Render (recommandé pour la démo)

1. Créer un compte sur [Render](https://render.com).
2. Connecter le dépôt GitHub `https://github.com/kodaxavier0-bit/SanteKit.git`.
3. Créer un nouveau **Web Service**.
4. Choisir le répertoire racine du dépôt et configurer :
   - **Build command** : `pip install -r api/requirements.txt`
   - **Start command** : `gunicorn app:app` (ou `python app.py` selon la configuration)
   - **Runtime** : Python 3
5. Variables d'environnement (si nécessaires) : ajouter dans le tableau de bord Render.
6. Déployer. L'URL de l'API sera du type `https://santekit-api.onrender.com`.

> Render met le service en veille après une période d'inactivité. Le premier appel peut prendre quelques secondes supplémentaires.

## 5. Déploiement sur PythonAnywhere (alternative)

1. Créer un compte sur [PythonAnywhere](https://www.pythonanywhere.com).
2. Uploader les fichiers du dossier `api/` via l'interface ou Git.
3. Configurer une **Web App** Flask.
4. Définir le chemin du fichier WSGI vers `app.py`.
5. Installer les dépendances via une console bash :
   ```bash
   pip install --user -r requirements.txt
   ```
6. Activer l'application et noter l'URL.

## 6. Contrat d'API (source de vérité)

### Routes disponibles

| Méthode | Route | Description |
|---------|-------|-------------|
| GET | `/health` | Vérifie que l'API est en ligne |
| POST | `/predict` | Reçoit les symptômes, renvoie le niveau d'urgence |

### Exemple de requête valide (`POST /predict`)

```bash
curl -X POST http://localhost:5000/predict \
  -H "Content-Type: application/json" \
  -d '{
    "age": 34,
    "fievre": true,
    "temperature": 38.6,
    "toux": true,
    "duree_jours": 3,
    "difficulte_respiratoire": false,
    "douleur_poitrine": false,
    "vomissements_diarrhee": false,
    "maux_tete_intenses": false
  }'
```

### Exemple de réponse attendue (200)

```json
{
  "niveau": "moyen",
  "couleur": "orange",
  "conseil": "Consultez un médecin dans les 24 à 48 heures.",
  "confiance": 0.87,
  "avertissement": "Ceci n'est pas un avis médical."
}
```

### Erreur 400 (champ invalide)

```json
{
  "erreur": "age doit être un entier positif"
}
```

## 7. Variables d'environnement

Aucune clé secrète, mot de passe ou donnée personnelle ne doit être stockée dans le code ni dans le dépôt. Si des variables sont nécessaires (URL du modèle, etc.), elles doivent être configurées via l'interface du fournisseur d'hébergement, jamais commitées.

## 8. Dépannage

| Problème | Solution |
|----------|----------|
| `ModuleNotFoundError` | Vérifier que `pip install -r api/requirements.txt` a été exécuté dans l'environnement virtuel activé |
| L'API ne répond pas | Vérifier que `app.py` est lancé et que le port 5000 est libre |
| Erreur 400 inattendue | Vérifier que tous les champs JSON requis sont présents et correctement typés |
| Temps de réponse > 3 s | Vérifier que le modèle joblib est chargé correctement et que le serveur n'est pas en veille |

## 9. Responsabilité

Ce fichier est sous la responsabilité de **ONLA LOOGA Gertrude Rose Lynne** (déploiement et tests de l'API). Toute modification doit être proposée via une Pull Request et validée par le chef de groupe (KODA Xavier).