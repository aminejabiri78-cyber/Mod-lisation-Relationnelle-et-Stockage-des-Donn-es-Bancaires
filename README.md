# 🏦 FinanceCore — Pipeline de Données Bancaires avec PostgreSQL & SQLAlchemy

> Pipeline ETL complet pour l'ingestion, la modélisation et l'analyse de transactions financières dans une base PostgreSQL relationnelle.

---

## 📋 Table des Matières

- [Aperçu du Projet](#aperçu-du-projet)
- [Architecture de la Base de Données](#architecture-de-la-base-de-données)
- [Prérequis](#prérequis)
- [Installation](#installation)
- [Configuration](#configuration)
- [Utilisation](#utilisation)
- [Structure du Schéma](#structure-du-schéma)
- [Vues KPI](#vues-kpi)
- [Contrôle d'Intégrité](#contrôle-dintégrité)

---

## 📌 Aperçu du Projet

Ce projet met en place un pipeline de données bancaires de bout en bout :

1. **Connexion** à une base PostgreSQL via SQLAlchemy
2. **Création** des tables normalisées (schéma relationnel en étoile)
3. **Chargement** des données depuis un fichier CSV (`financecore_clean.csv`)
4. **Indexation** pour l'optimisation des requêtes
5. **Vues analytiques** pour les KPIs métiers
6. **Vérification** de l'intégrité référentielle

---

## 🗄️ Architecture de la Base de Données

```
segment ──────────┐
                  ▼
agence ──── transactions ──── client
                  ▲                ▲
produit ──────────┘           segment
```

Le schéma suit un modèle en **étoile** avec `transactions` comme table de faits centrale.

---

## ⚙️ Prérequis

- Python 3.8+
- PostgreSQL 13+
- Packages Python :

```bash
pip install pandas sqlalchemy psycopg2-binary python-dotenv
```

---

## 🚀 Installation

```bash
# 1. Cloner le dépôt
git clone https://github.com/votre-utilisateur/financecore.git
cd financecore

# 2. Installer les dépendances
pip install -r requirements.txt

# 3. Configurer les variables d'environnement
cp .env.example .env
# Remplir les valeurs dans .env

# 4. Lancer le pipeline
python pipeline.py
```

---

## 🔐 Configuration

Créer un fichier `.env` à la racine du projet :

```env
DB_USER=votre_utilisateur
DB_PASS=votre_mot_de_passe
DB_HOST=localhost
DB_PORT=5432
DB_NAME=financecore
```

> ⚠️ Ne jamais committer le fichier `.env`. Il est ignoré via `.gitignore`.

---

## 📂 Utilisation

Le script principal effectue les étapes suivantes dans l'ordre :

```python
# Étape 1 — Connexion
engine = create_engine(db_url)

# Étape 2 — Création des tables
# segment, client, agence, produit, transactions

# Étape 3 — Chargement depuis CSV
df = pd.read_csv("financecore_clean.csv")

# Étape 4 — Insertion dans l'ordre des dépendances
# segment → client → agence + produit → transactions

# Étape 5 — Création des index
# idx_client_id, idx_date_transactions, idx_agence

# Étape 6 — Création des vues analytiques
```

---

## 🗂️ Structure du Schéma

### `segment`
| Colonne | Type | Description |
|---|---|---|
| `segment_id` | SERIAL PK | Identifiant auto |
| `segment_client` | VARCHAR(60) UNIQUE | Libellé du segment |

### `client`
| Colonne | Type | Description |
|---|---|---|
| `client_id` | VARCHAR(60) PK | Identifiant client |
| `score_credit_client` | DECIMAL(10,3) | Score de crédit |
| `segment_id` | INT FK → segment | Segment d'appartenance |

### `agence`
| Colonne | Type | Description |
|---|---|---|
| `agence_id` | SERIAL PK | Identifiant auto |
| `agence` | VARCHAR(60) UNIQUE | Nom de l'agence |

### `produit`
| Colonne | Type | Description |
|---|---|---|
| `produit_id` | SERIAL PK | Identifiant auto |
| `produit` | VARCHAR(60) | Nom du produit |
| `categorie` | VARCHAR(60) | Catégorie du produit |

### `transactions`
| Colonne | Type | Description |
|---|---|---|
| `transaction_id` | VARCHAR(60) PK | Identifiant transaction |
| `client_id` | VARCHAR(60) FK | Référence client |
| `produit_id` | INT FK | Référence produit |
| `agence_id` | INT FK | Référence agence |
| `date_transaction` | VARCHAR(60) | Date de la transaction |
| `montant` | DECIMAL(12,2) | Montant |
| `devise` | VARCHAR(10) | Code devise |
| `taux_change_eur` | DECIMAL(10,4) | Taux de change EUR |
| `type_operation` | VARCHAR(60) | Type d'opération |
| `statut` | VARCHAR(30) | Statut de la transaction |
| `solde_avant` | DECIMAL(12,2) | Solde avant transaction |
| `is_anomalie` | VARCHAR(60) | Flag anomalie |

---

## 📊 Vues KPI

| Vue | Description |
|---|---|
| `jointure_table` | Vue dénormalisée complète (toutes les tables jointes) |
| `kpi_total_transactions` | Nombre, somme et moyenne des transactions |
| `kpi_risk_segment` | Score moyen et taux de défaut par segment |
| `kpi_transactions_mensuelles` | Volume mensuel des transactions |

### Exemple d'utilisation

```python
with engine.connect() as con:
    df_kpi = pd.read_sql(text("SELECT * FROM kpi_transactions_mensuelles"), con)
print(df_kpi)
```

---

## ✅ Contrôle d'Intégrité

Le pipeline vérifie l'absence d'**enregistrements orphelins** après chaque chargement :

```sql
-- Clients sans transactions
SELECT count(*) FROM client cl
LEFT JOIN transactions t ON t.client_id = cl.client_id
WHERE t.client_id IS NULL;

-- Segments sans clients
SELECT count(*) FROM segment s
LEFT JOIN client cl ON s.segment_id = cl.segment_id
WHERE cl.segment_id IS NULL;
```

Un résultat de `0` pour chaque requête confirme l'intégrité référentielle complète.

---

## 📁 Structure du Projet

```
MODÉLISATION_RELATIONNELLE/
│
├── assets/
│   └── EDR.png                              # Diagramme Entité-Relation
│
├── data/
│   └── financecore_clean.csv                # Dataset source
│
├── src/
│   └── financiere_bank_modelesitation.ipynb # Notebook principal (ETL + SQL)
│
├── .venv/                                   # Environnement virtuel Python
│   ├── Include/
│   ├── Lib/
│   ├── Scripts/
│   ├── share/
│   ├── .gitignore
│   └── pyvenv.cfg
│
├── .env                                     # Variables d'environnement (non versionné)
├── .gitignore
├── README.md
└── requirements.txt
```

---

## 🤝 Contribution

Les contributions sont les bienvenues. Merci d'ouvrir une *issue* avant de soumettre une *pull request*.

---

## 📝 Licence

Ce projet est sous licence MIT.
