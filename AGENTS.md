# AGENTS.md — Directives pour SantéKit

Ce fichier s'adresse aux **assistants IA** (Claude, Copilot, Cursor, etc.) et aux **membres de l'équipe** qui les utilisent. Il se place à la racine du dépôt. Toute IA qui travaille sur ce projet doit le lire et le respecter avant de proposer du code.

---

## 1. Le projet en bref

**SantéKit** est une application Android d'aide à l'orientation face aux symptômes, associée à une API d'intelligence artificielle. L'utilisateur saisit ses symptômes et obtient un niveau d'urgence (faible, moyen, élevé) avec un conseil général.

- Projet de groupe (Groupe 3, 6 membres), chef de groupe : KODA Xavier.
- Dépôt : https://github.com/kodaxavier0-bit/SanteKit.git
- Projet académique : le code sera présenté et expliqué devant l'enseignant.

### Règle absolue : avertissement médical

SantéKit **ne remplace pas un médecin**. L'application ne fait **aucun diagnostic**, ne propose **aucune prescription** et ne cite **aucun médicament**. L'avertissement « Ceci n'est pas un avis médical » doit être :
- affiché avant toute saisie (écran d'accueil),
- visible sur **chaque** écran de résultat.

L'IA ne doit jamais écrire un conseil, un texte ou une réponse du chatbot qui ressemble à un diagnostic ou à une ordonnance.

---

## 2. Structure du dépôt

```
SantéKit/
├── android/      Application Android (Java/XML)
├── api/          API Flask (app.py, requirements.txt, README)
├── ml/           Dataset synthétique, scripts d'entraînement, modèle .joblib, métriques
├── docs/         Rapport, contrat d'API, textes des conseils, présentation
├── README.md     Présentation et instructions générales
├── AGENTS.md     Ce fichier
└── .gitignore
```

Chaque membre travaille **dans son dossier** (voir section 3). Ne jamais déplacer ni renommer les dossiers racine.

---

## 3. Répartition et propriété des fichiers

| Membre | Responsabilité | Zone principale |
|---|---|---|
| KODA Xavier (chef) | Coordination, Git, architecture, API Flask, revue de code | `api/`, fichiers racine |
| HINMAKA TEHALE NYEYAMBE Obed | Écrans Android, formulaire, intégration Retrofit | `android/` : paquets `ui`, `network`, `model` |
| NDIKUM FOKU Boris | Dataset synthétique, entraînement et évaluation du modèle | `ml/` |
| NJOUNDAM NOUBISSI Tatiana Claidane | SQLite, historique, centres de santé | `android/` : paquet `db` |
| ONLA LOOGA Gertrude Rose Lynne | Tests, Postman, déploiement de l'API | `api/` (déploiement), `docs/tests` |
| TANGATAI Veronique | Documentation, textes des conseils, rapport, présentation | `docs/` |

**Règle pour l'IA :** ne modifie que les fichiers de la zone de la personne qui te sollicite. Si une modification touche la zone d'un autre membre, **signale-le** au lieu de la faire.

### Fichiers partagés (à ne pas modifier sans accord)

`AndroidManifest.xml`, `build.gradle.kts` (dépendances), `settings.gradle.kts`, `gradle.properties`, `strings.xml`, `requirements.txt`, `.gitignore`, `README.md`, `AGENTS.md`.

Si une modification est nécessaire, la proposer explicitement (quelle ligne, pourquoi) pour que le membre la valide avec le chef.

---

## 4. Contrat d'API (source de vérité)

Le contrat suivant est la référence commune entre l'app Android et l'API Flask. **L'IA ne doit jamais l'inventer ni le modifier de sa propre initiative.**

**Routes :**
- `GET /health` : vérifie que l'API répond.
- `POST /predict` : reçoit les symptômes, renvoie le niveau d'urgence.

**Requête `POST /predict`** (JSON) :
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

**Réponse 200 :**
```json
{
  "niveau": "moyen",
  "couleur": "orange",
  "conseil": "Consultez un médecin dans les 24 à 48 heures.",
  "confiance": 0.87,
  "avertissement": "Ceci n'est pas un avis médical."
}
```

**Erreur 400 :** `{ "erreur": "age doit être un entier positif" }`

Règles :
- Les noms de champs JSON sont en `snake_case`, exactement comme ci-dessus.
- Niveaux : faible (vert), moyen (orange), élevé (rouge).
- En cas d'erreur ou d'API injoignable, l'application affiche un **message d'erreur clair** et **n'invente jamais un résultat**.
- Toute évolution du contrat se décide avec Xavier et est mise à jour dans `docs/api-contract.md` avant d'être codée.

### Points à confirmer (à trancher avec Xavier)

- Valeur exacte de `niveau` pour le niveau élevé : `eleve` ou `élevé` ?
- `temperature` est-elle obligatoire quand `fievre` est faux ?
- Plages de validation (âge maximum, durée maximum en jours).

---

## 5. Stack technique

| Bloc | Technologies |
|---|---|
| Client Android | Java, XML, Retrofit + Gson, SQLite |
| API | Python, Flask, joblib |
| Modèle | scikit-learn (DecisionTreeClassifier, comparé à une régression logistique) |
| Hébergement démo | Render ou PythonAnywhere |

Règles :
- **Pas de nouvelle dépendance** (bibliothèque, plugin Gradle, paquet pip) sans accord du chef. Les licences doivent rester libres et le coût nul.
- Pas de fournisseur d'IA externe : le modèle est entraîné et hébergé par le groupe.
- Aucune donnée de santé stockée côté serveur ; l'historique reste sur le téléphone (SQLite).
- Le projet Android est **en Java** (pas de Kotlin pour le code applicatif), avec des fichiers Gradle en Kotlin DSL (`.kts`).

---

## 6. Android : conventions

**Package racine :** `com.santekit.app`

```
com.santekit.app
├── ui        Activities et adaptateurs (Accueil, Formulaire, Résultat, Historique, Centres)
├── network   Interface Retrofit, client, sources de prédiction
├── model     Classes de données (SymptomRequest, PredictionResponse...)
└── db        SQLite : historique (evaluations) et centres (centres_sante)
```

**Règles de code :**
- Classes en `PascalCase`, méthodes et variables en `camelCase`, constantes en `MAJUSCULES_SNAKE`.
- Les champs JSON sont liés avec `@SerializedName` pour respecter le `snake_case` du contrat.
- Un écran = une Activity + un layout XML (`activity_formulaire.xml`, etc.).
- **Aucun texte en dur** dans les layouts ou le code : tout passe par `strings.xml` (l'avertissement médical inclus).
- **Aucun accès réseau sur le thread principal** : Retrofit en asynchrone (`enqueue`).
- Les écrans n'appellent jamais Retrofit directement : ils passent par l'interface `PredictionSource` (voir ci-dessous).
- Le code réseau, la base de données et l'affichage restent dans leurs paquets respectifs.

### Travailler sans l'API : le mock

Tant que l'API n'est pas prête, on développe avec une fausse source de données **au même format que la vraie** :
- `PredictionSource` : interface commune.
- `MockPredictionSource` : renvoie une réponse simulée selon les symptômes.
- `RetrofitPredictionSource` : appelle la vraie API.
- Un seul endroit du code choisit laquelle utiliser (par exemple une constante `USE_MOCK`). Les écrans ne changent pas quand on bascule.

### Accessibilité et interface

- La couleur du résultat est **toujours doublée d'un texte** (« Urgence élevée »), jamais la couleur seule.
- Validation des champs **avant** tout envoi : champ vide, âge négatif ou incohérent, température absente si fièvre cochée.
- Les erreurs sont affichées en langage simple, sans message technique brut.
- Les centres de santé et l'historique doivent fonctionner **hors connexion**.
- Résultat attendu en moins de 3 secondes quand l'API est disponible.

---

## 7. API Flask et modèle

- `GET /health` et `POST /predict` uniquement (sauf décision contraire).
- **Validation stricte** des entrées avec erreur 400 et message explicite (même format que le contrat).
- `requirements.txt` avec **versions figées**.
- Le modèle est chargé depuis le fichier joblib ; le chemin ne doit pas dépendre de la machine d'un membre.
- Le dataset est **synthétique** et généré par un script **reproductible** (graine fixée). La mention « données fictives, sans validité clinique » doit apparaître dans le dataset, le rapport et le README.
- Métriques visées (jeu de test 20 %, séparation stratifiée) : exactitude ≥ 90 %, rappel de la classe « élevée » ≥ 95 %. Sous-estimer une urgence est l'erreur la plus grave.
- Pas de secret, clé ou mot de passe dans le code.

---

## 8. Git et collaboration

**Branches :** une branche par membre et par tâche, créée depuis `main` à jour.
```
feature/<prenom>-<sujet>      ex. feature/obed-formulaire
fix/<prenom>-<sujet>          ex. fix/tatiana-historique
```

**Cycle de travail :**
1. `pull` de `main` avant de commencer.
2. Travailler sur sa branche, avec de petits commits fréquents.
3. Avant la PR : fusionner `origin/main` dans sa branche et régler les conflits localement.
4. Ouvrir une **Pull Request** vers `main` quand une partie fonctionne.
5. Le chef relit et fusionne.
6. Tout le monde fait un `pull` de `main`, puis recrée une branche pour la tâche suivante.

**Règles :**
- **Jamais de push direct sur `main`.**
- Les commits sont faits **par chacun avec son propre compte** (l'historique doit prouver la contribution des six membres).
- Messages de commit en français, à l'impératif, courts et précis : `Ajoute la validation du formulaire`, `Corrige l'appel Retrofit`.
- Une PR = un sujet. Elle doit compiler et se lancer avant d'être ouverte.
- Titre de PR clair, avec une courte description de ce qui a été fait et comment le tester.

**À ne jamais commiter :**
`local.properties`, `build/`, `.gradle/`, `.idea/`, fichiers `*.jks` / `*.keystore` (signature de l'APK), fichiers `.env`, clés et mots de passe, fichiers temporaires.

---

## 9. Règles pour les assistants IA

### Avant d'écrire du code
1. Lire ce fichier et le contrat d'API.
2. Identifier la zone concernée et s'assurer qu'elle appartient à la personne qui demande.
3. Si une information manque (nom de champ, comportement, version), **demander** plutôt que supposer.

### En écrivant du code
- Produire des changements **petits et ciblés** : une fonctionnalité à la fois.
- Respecter les conventions de ce fichier (langage, paquets, noms, `strings.xml`).
- Ne pas ajouter de dépendances, de fichiers partagés modifiés ou de refactorisations non demandées.
- Ne pas inventer de routes, de champs JSON ou de valeurs qui ne sont pas dans le contrat.
- Ne jamais coder un résultat factice dans le chemin de production : un mock reste isolé derrière `PredictionSource`.
- Ne jamais écrire de texte médical (diagnostic, médicament, dosage). Les textes des conseils viennent de `docs/` (Veronique).
- Ne jamais inclure de secret ou de donnée personnelle réelle dans le code, les exemples ou les tests.
- Commenter le code de façon utile, en français, sans verbosité.

### Après avoir écrit du code
- Indiquer **quels fichiers** sont créés ou modifiés et **pourquoi**.
- Indiquer **comment tester** (écran à ouvrir, requête curl, scénario).
- Signaler tout fichier partagé touché ou toute hypothèse faite.

### Responsabilité humaine
Le projet sera défendu devant l'enseignant. Chaque membre doit **comprendre le code qu'il commit**. L'IA doit expliquer ses choix quand on le lui demande, et un membre ne commite jamais du code qu'il ne saurait pas expliquer.

---

## 10. Tests et critères de réussite

**Scénarios fonctionnels à vérifier :**
- Adulte avec fièvre légère depuis 1 jour → niveau **faible**.
- Enfant avec forte fièvre depuis 3 jours et toux → niveau **moyen**.
- Difficulté à respirer avec douleur à la poitrine → niveau **élevé**.
- Formulaire incomplet ou âge négatif → message de validation, **aucun envoi**.
- API éteinte ou réseau coupé → message d'erreur clair, **aucun résultat inventé**.
- Historique conservé après redémarrage de l'application.
- Centres de santé affichés hors connexion.

**API :** requêtes Postman ou curl (cas valides et invalides), temps de réponse inférieur à 3 secondes.

**Définition de « terminé » pour une tâche :**
- le code compile et l'application se lance sans plantage,
- le comportement est testé sur au moins un scénario,
- les textes passent par `strings.xml`,
- l'avertissement médical est présent là où il doit l'être,
- la PR est ouverte et décrite.

---

## 11. Livrables du projet (rappel)

1. APK Android signé, publié en release.
2. API Flask avec README (routes, installation, exemples curl).
3. Dataset, scripts, modèle et rapport de métriques.
4. Dépôt Git (branches par membre, PR, release v1.0).
5. Rapport, textes définitifs des conseils, présentation et scénario de démonstration.

---

## 12. Communication d'équipe

- Un changement de contrat d'API, de structure ou de dépendances est **annoncé au groupe avant d'être fait**.
- En cas de blocage ou de conflit Git, prévenir le chef plutôt que de forcer (`push --force` est interdit sur les branches partagées).
- Ce fichier est mis à jour par PR, avec l'accord du chef, quand une règle évolue.
