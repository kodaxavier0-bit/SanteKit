# SantéKit

Application mobile Android d'aide à la détection et à l'orientation face aux symptômes, associée à une API d'intelligence artificielle.

> ⚠️ **Avertissement : SantéKit ne remplace pas un médecin.** L'application ne fait aucun diagnostic, ne propose aucune prescription et ne cite aucun médicament. Elle fournit uniquement une orientation indicative. **Ceci n'est pas un avis médical.**

> **Données fictives :** le modèle est entraîné sur un dataset **synthétique**, sans aucune validité clinique. Les métriques mesurent la fidélité du modèle aux règles de génération, pas sa performance médicale réelle.

Projet de groupe (Groupe 3).

---

## Sommaire

1. [Présentation](#présentation)
2. [Fonctionnalités (MVP)](#fonctionnalités-mvp)
3. [Architecture](#architecture)
4. [Structure du dépôt](#structure-du-dépôt)
5. [Contrat d'API](#contrat-dapi)
6. [Installation et lancement](#installation-et-lancement)
7. [Tests](#tests)
8. [Travailler sur le projet](#travailler-sur-le-projet)
9. [Équipe](#équipe)
10. [Limites](#limites)
11. [Licences](#licences)

---

## Présentation

Beaucoup de personnes s'autoconsultent ou ignorent leurs symptômes, par manque de moyens ou parce qu'elles estiment que ce n'est pas grave. Cela retarde la consultation dans les cas sérieux et encombre inutilement les urgences pour les cas bénins.

SantéKit permet à l'utilisateur de saisir ses symptômes et d'obtenir immédiatement un **niveau d'urgence** (faible, moyen ou élevé) accompagné d'un conseil général, afin de décider s'il doit consulter rapidement ou s'auto-surveiller.

**Public visé :** le grand public équipé d'un smartphone Android (adultes, parents s'occupant d'enfants), les personnes éloignées d'un avis médical immédiat, les étudiants et familles sans connaissance médicale particulière.

---

## Fonctionnalités (MVP)

| Fonctionnalité | Description |
|---|---|
| Accueil et avertissement | Message « ce n'est pas un avis médical » affiché avant toute saisie |
| Formulaire de symptômes | Âge, fièvre (+ température), toux, durée en jours, difficulté à respirer, douleur à la poitrine, vomissements/diarrhée, maux de tête intenses, avec validation avant l'envoi |
| Résultat coloré | Niveau faible (vert), moyen (orange) ou élevé (rouge) + conseil adapté. La couleur est toujours accompagnée d'un texte |
| Chatbot d'orientation | Assistant conversationnel pour expliquer son état en langage naturel |
| Centres de santé | Liste préenregistrée (nom, quartier/ville, téléphone), consultable hors connexion |
| Carnet médical | Historique des évaluations conservé localement sur le téléphone (SQLite) |

*Hors MVP, si le temps le permet : recherche et filtre dans la liste des centres, suppression de l'historique, affichage de la confiance du modèle.*

---

## Architecture

```
┌──────────────────┐   POST /predict   ┌───────────────┐        ┌────────────────┐
│  Client Android  │ ────────────────► │   API Flask   │ ─────► │  Modèle joblib │
│ (Java, Retrofit) │ ◄──────────────── │               │ ◄───── │ (scikit-learn) │
└────────┬─────────┘    réponse JSON   └───────────────┘        └────────────────┘
         │
    ┌────▼─────┐
    │  SQLite  │  historique + centres de santé (local au téléphone)
    └──────────┘
```

Flux : formulaire → Retrofit → API → modèle → réponse JSON → écran Résultat → historique.

- **Modèle :** arbre de décision (`DecisionTreeClassifier`), comparé à une régression logistique. Le meilleur modèle est retenu, avec priorité au rappel de la classe « élevée ».
- **Stockage :** SQLite sur le téléphone uniquement. Le serveur ne contient que le modèle et le dataset, **aucune donnée de santé d'utilisateur**.
- **Aucun fournisseur d'IA externe :** le modèle est entraîné et hébergé par le groupe.

---

## Structure du dépôt

```
SanteKit/
├── android/      Application Android (Java/XML, Retrofit + Gson, SQLite)
├── api/          API Flask (app.py, requirements.txt, README de l'API)
├── ml/           Dataset synthétique, scripts de génération et d'entraînement, modèle .joblib, métriques
├── docs/         Rapport, contrat d'API, textes des conseils, présentation
├── AGENTS.md     Directives pour l'équipe et les assistants IA
└── README.md     Ce fichier
```

---

## Contrat d'API

Référence commune entre l'application et l'API.

**Routes :**
- `GET /health` : vérifie que l'API répond
- `POST /predict` : renvoie le niveau d'urgence pour des symptômes donnés

**Requête `POST /predict`** (`Content-Type: application/json`) :
```json
{
  "age": 34,
  "fievre": true,
  "temperature": 38.6,
  "toux": true,
  "duree_jours": 3,
  "difficulte_respiratoire": false,
  "douleur_poitrine": false,
  "vomissements_diarrhee": false,
  "maux_tete_intenses": false
}
```

**Réponse `200 OK` :**
```json
{
  "niveau": "moyen",
  "couleur": "orange",
  "conseil": "Consultez un médecin dans les 24 à 48 heures.",
  "confiance": 0.87,
  "avertissement": "Ceci n'est pas un avis médical."
}
```

**Réponse `400 Bad Request` :**
```json
{ "erreur": "age doit être un entier positif" }
```

En cas d'erreur ou d'API injoignable, l'application affiche un message clair et **n'invente jamais de résultat**.

---

## Installation et lancement

> Les commandes ci-dessous sont indicatives : adapter selon l'état final de `api/` et `ml/`.

### Application Android

**Prérequis :** Android Studio récent, JDK fourni avec Android Studio, un émulateur ou un téléphone Android.

1. Cloner le dépôt :
   ```bash
   git clone https://github.com/kodaxavier0-bit/SanteKit.git
   ```
2. Dans Android Studio : *File → Open* et choisir le dossier **`android/`**.
3. Laisser Gradle se synchroniser, puis lancer l'application (*Run*).

**Connexion à l'API en développement :**
- Émulateur Android : l'API tournant sur le PC est accessible via `http://10.0.2.2:5000/`.
- Téléphone réel en Wi-Fi : utiliser l'adresse IP locale du PC (ex. `http://192.168.x.x:5000/`).
- Tant que l'API n'est pas disponible, l'application peut fonctionner avec une source de données simulée (mock).

### API Flask

**Prérequis :** Python 3 et pip.

```bash
cd api
python -m venv venv
# Windows : venv\Scripts\activate    |    Linux/macOS : source venv/bin/activate
pip install -r requirements.txt
python app.py
```

L'API est alors disponible sur `http://localhost:5000`. Vérification rapide :
```bash
curl http://localhost:5000/health
```

Exemple de prédiction :
```bash
curl -X POST http://localhost:5000/predict \
  -H "Content-Type: application/json" \
  -d '{"age":34,"fievre":true,"temperature":38.6,"toux":true,"duree_jours":3,"difficulte_respiratoire":false,"douleur_poitrine":false,"vomissements_diarrhee":false,"maux_tete_intenses":false}'
```

### Données et modèle

```bash
cd ml
# 1. Générer le dataset synthétique (graine fixée, reproductible)
# 2. Entraîner et évaluer le modèle, produire le fichier .joblib
# Les noms exacts des scripts seront précisés ici par le responsable du modèle.
```

---

## Tests

**Modèle** (jeu de test 20 %, séparation stratifiée) :
- exactitude ≥ 90 %
- rappel de la classe « élevée » ≥ 95 % (sous-estimer une urgence est l'erreur la plus grave)
- F1 macro et matrice de confusion pour comparer l'arbre de décision et la régression logistique

**Scénarios fonctionnels :**
- Adulte avec fièvre légère depuis 1 jour → **faible**
- Enfant avec forte fièvre depuis 3 jours et toux → **moyen**
- Difficulté à respirer avec douleur à la poitrine → **élevé**
- Formulaire incomplet ou âge négatif → message de validation, aucun envoi
- API éteinte ou réseau coupé → message d'erreur clair
- Historique conservé après redémarrage de l'application
- Centres de santé affichés hors connexion

**API :** requêtes Postman ou curl (cas valides et invalides), temps de réponse visé inférieur à 3 secondes.

---

## Travailler sur le projet

Toutes les règles de collaboration et les directives pour les assistants IA sont dans [`AGENTS.md`](AGENTS.md). En résumé :

- Une **branche par membre et par tâche**, créée depuis `main` à jour (`feature/<prenom>-<sujet>`).
- **Jamais de push direct sur `main`** : on passe par une **Pull Request**, relue par le chef de groupe.
- Commits en français, courts et précis, faits par chacun avec son propre compte.
- Ne jamais commiter `local.properties`, `build/`, `.gradle/`, les fichiers de signature (`*.jks`, `*.keystore`) ni aucun secret.

---

## Équipe

| Membre | Responsabilité |
|---|---|
| KODA Xavier (chef de groupe) | Coordination, dépôt Git, architecture, API Flask, intégration et revue de code |
| HINMAKA TEHALE NYEYAMBE Obed | Application Android : écrans, formulaire, intégration Retrofit |
| NDIKUM FOKU Boris | Dataset synthétique, entraînement et évaluation du modèle scikit-learn |
| NJOUNDAM NOUBISSI Tatiana Claidane | Base SQLite, historique, collecte et saisie des centres de santé |
| ONLA LOOGA Gertrude Rose Lynne | Tests (scénarios, Postman), déploiement de l'API, configuration réseau |
| TANGATAI Veronique | Documentation, conseils et avertissements, rapport et présentation |

---

## Limites

- Dataset **synthétique**, clairement fictif, sans validité clinique.
- Seuls les symptômes du formulaire sont pris en compte.
- Aucune donnée de santé n'est stockée côté serveur.
- Hébergement gratuit de démonstration : l'API peut être en veille et répondre lentement au premier appel.

---

## Licences

| Composant | Licence |
|---|---|
| Flask, scikit-learn | BSD-3 |
| Retrofit, Android | Apache 2.0 |
| SQLite | Domaine public |

Coût du projet : nul (outils libres, hébergement gratuit).
