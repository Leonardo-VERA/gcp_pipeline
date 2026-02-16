# GCP Pipeline - Ingestion Shopify vers Google Cloud Platform

Pipeline d'ingestion de donnees Shopify vers GCP utilisant Cloud Build pour l'orchestration CI/CD.

## Architecture

```
Shopify API  -->  Extraction Python  -->  data/raw/ (JSON)
                                              |
                                         Transformation
                                              |
                                         data/processed/ (CSV)
                                              |
                                      Google Cloud Platform
                                       (BigQuery / GCS)
```

## Structure du projet

```
gcp_pipeline/
├── src/
│   └── gcp_pipeline/
│       ├── __init__.py
│       ├── main.py                    # Point d'entree principal
│       ├── shopify_simple_extract.py  # Extraction des donnees Shopify (produits, commandes, clients)
│       └── generate_test_data.py      # Generation de donnees de test via l'API Shopify
├── data/
│   ├── raw/                           # Donnees brutes JSON extraites de Shopify
│   └── processed/                     # Donnees transformees en CSV (products, variants, customers)
├── notebooks/
│   ├── shopify_analysis.ipynb         # Analyse exploratoire des donnees Shopify
│   └── analyze_csv.ipynb              # Analyse des fichiers CSV transformes
├── .env                               # Variables d'environnement (non versionne)
├── .gitignore
├── requirements.txt
└── README.md
```

## Donnees extraites

| Endpoint Shopify | Description            | Format sortie       |
|------------------|------------------------|---------------------|
| `/products.json` | Catalogue produits     | JSON brut + CSV     |
| `/orders.json`   | Commandes              | JSON brut           |
| `/customers.json`| Clients                | JSON brut + CSV     |

Les fichiers CSV generes dans `data/processed/` :
- **products.csv** - Informations produits (titre, vendeur, type, prix)
- **variants.csv** - Variantes produits (SKU, prix, inventaire)
- **customers.csv** - Donnees clients (anonymisees)

## Prerequis

- Python 3.10+
- Un compte Shopify avec une application Private/Custom App configuree
- Un projet GCP avec les APIs activees (BigQuery, Cloud Storage, Cloud Build)
- `gcloud` CLI installe et authentifie

## Installation

```bash
# Cloner le repository
git clone <URL_DU_REPO>
cd gcp_pipeline

# Creer un environnement virtuel
python -m venv .venv
source .venv/bin/activate

# Installer les dependances
pip install -r requirements.txt
```

## Configuration


> **Ne jamais commiter le fichier `.env`.** Il est deja present dans le `.gitignore`.

## Utilisation

### Extraction des donnees Shopify

```bash
# Lancer l'extraction complete (produits, commandes, clients)
python src/gcp_pipeline/shopify_simple_extract.py
```

Ce script :
1. Teste la connexion a l'API Shopify
2. Extrait les produits, commandes et clients (jusqu'a 250 par endpoint)
3. Affiche un apercu des donnees brutes
4. Sauvegarde les donnees en JSON horodate dans `data/raw/`

### Generation de donnees de test

```bash
# Creer 100 produits de test dans votre boutique Shopify
python src/gcp_pipeline/generate_test_data.py
```

### Analyse exploratoire

Les notebooks Jupyter dans `notebooks/` permettent d'explorer et analyser les donnees extraites.

```bash
jupyter notebook notebooks/
```

## Cloud Build (CI/CD)

Le deploiement est orchestre via Google Cloud Build. La configuration permet de :

- Valider le code a chaque push
- Executer le pipeline d'extraction
- Charger les donnees dans BigQuery / Cloud Storage

```yaml
# cloudbuild.yaml (a configurer)
steps:
  - name: 'python:3.10'
    entrypoint: 'pip'
    args: ['install', '-r', 'requirements.txt']
  - name: 'python:3.10'
    entrypoint: 'python'
    args: ['src/gcp_pipeline/shopify_simple_extract.py']
    secretEnv: ['SHOPIFY_STORE_NAME', 'SHOPIFY_ACCESS_TOKEN']
```

## Dependances

| Package        | Utilisation                          |
|----------------|--------------------------------------|
| pandas         | Manipulation et transformation des donnees |
| numpy          | Calculs numeriques                   |
| requests       | Appels HTTP vers l'API Shopify       |
| python-dotenv  | Chargement des variables `.env`      |
| matplotlib     | Visualisation des donnees            |
| ipython        | Notebooks interactifs                |

## Roadmap

- [x] Connexion et extraction depuis l'API Shopify
- [x] Sauvegarde des donnees brutes en JSON
- [x] Transformation en CSV (produits, variants, clients)
- [x] Notebooks d'analyse exploratoire
- [ ] Chargement vers BigQuery
- [ ] Stockage des fichiers bruts dans Cloud Storage (GCS)
- [ ] Configuration Cloud Build (`cloudbuild.yaml`)
- [ ] Scheduling automatique des extractions
- [ ] Gestion des secrets via GCP Secret Manager
