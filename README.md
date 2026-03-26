# Previsione del Prezzo degli Immobili Residenziali

Progetto di Machine Learning per la stima automatizzata del valore di mercato di abitazioni residenziali, sviluppato su **Dataiku DSS**.

---

## Panoramica

Un'agenzia immobiliare o un portale online vuole prevedere il prezzo di vendita degli immobili residenziali. Il modello supporta tre scenari principali:

- **Valutazione automatizzata (AVM):** stime rapide del prezzo di mercato
- **Supporto agli agenti:** definizione del prezzo di listino ottimale
- **Analisi investimenti:** identificazione di immobili sotto/sopravvalutati

---

## Dataset

- **Fonte:** [House Prices – Advanced Regression Techniques (Kaggle)](https://www.kaggle.com/datasets/shree1992/housedata)
- **Dimensioni:** ~80 variabili su immobili residenziali
- **Target:** `SalePrice` – prezzo di vendita finale dell'immobile

| Categoria | Variabili | Descrizione |
|---|---|---|
| Dimensioni | `GrLivArea`, `TotalBsmtSF`, `LotArea` | Metratura zona giorno, seminterrato, lotto |
| Qualità | `OverallQual`, `OverallCond`, `YearBuilt` | Qualità, condizione, anno di costruzione |
| Caratteristiche | `BedroomAbvGr`, `FullBath`, `GarageCars` | Camere, bagni, posti auto |
| Location | `Neighborhood` | Quartiere dell'immobile |
| **Target** | **`SalePrice`** | **Prezzo di vendita finale** |

---

## Flusso di Lavoro in Dataiku

Il progetto è composto da due rami paralleli che confluiscono nella recipe di scoring:

```
house_raw --> house_clean --> house_features --> [Predict LogSalePrice] --> house_scored --> house_scored_prepared
                                                          |
test_raw  --> test_clean  --> test_clean_prepared --------+--> house_predictions --> house_predictions_prepared
```

### Preparazione dei Dati

- **Valori mancanti:** le variabili categoriche come `GarageType` e `BsmtQual` vengono imputate con la categoria `"Nessuno"`, poiché il valore mancante indica l'assenza della caratteristica
- **Feature engineering:**
  - `HouseAge` = `YrSold` - `YearBuilt`
  - `TotalSF` = `TotalBsmtSF` + `1stFlrSF` + `2ndFlrSF`
  - Trasformazione logaritmica (`log(x+1)`) su `SalePrice` e feature numeriche asimmetriche
- **Encoding:** dummy encoding per le variabili categoriche

---

## Modelli e Risultati

Sono stati addestrati due modelli su 1185 righe (train) e valutati su 275 righe (test):

| Modello | R² |
|---|---|
| Gradient Boosted Trees | 0.876 |
| Ridge (L2) Regression | 0.836 |

Il modello selezionato è il **Gradient Boosted Trees** (300 alberi, learning rate 0.1, max depth 3).

### Metriche di Valutazione (Gradient Boosted Trees)

| Metrica | Valore |
|---|---|
| R² Score | 0.8755 |
| RMSE | 0.1338 |
| RMSLE | 0.01034 |
| MAE | 0.09385 |
| MAPE | 0.79% |

### Feature Importance

Le variabili con maggiore impatto predittivo sul prezzo:

| Feature | Importanza |
|---|---|
| `LogTotalSF` | 25% |
| `OverallQual` | 17% |
| `HouseAge` | 12% |
| `OverallCond` | 9% |
| `LogLotArea` | 7% |
| `LogGrLivArea` | 5% |
| `GarageCars` | 5% |

Dal SHAP plot si osserva che valori alti di `LogTotalSF` e `OverallQual` aumentano il prezzo previsto, mentre un'età elevata (`HouseAge`) tende a ridurlo.

---

## Struttura della Repository

```
house-price-prediction/
├── data/
│   ├── raw/              # Dataset originale (house_raw) - non tracciato da git
│   ├── processed/        # Dataset intermedi prodotti in Dataiku
│   └── predictions/      # Output della Score Recipe (house_predictions)
├── docs/
│   └── presentazione     # Presentazione del progetto (Gamma)
├── notebooks/
│   └── exploratory_analysis.ipynb
├── .gitignore
└── README.md
```

---

## Presentazione

La presentazione del progetto è stata realizzata su **Gamma** (link : https://gamma.app/docs/Quanto-vale-casa-tua-3or725m2inyj29u?mode=doc) e illustra le scelte di modellazione e le principali determinanti del prezzo emerse dall'analisi.

---

## Come Replicare il Progetto

1. Scaricare il dataset da [Kaggle](https://www.kaggle.com/datasets/shree1992/housedata) e caricarlo in Dataiku come `house_raw`
2. Eseguire le recipe nell'ordine: Prepare → Feature Engineering → Split → Train → Score
3. Consultare `house_predictions` per le stime sui nuovi immobili

---
