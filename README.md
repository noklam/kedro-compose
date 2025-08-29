# Kedro Compose Patterns

## 1. Simple Branching Pattern

One parent pipeline branches into multiple child pipelines with automatic connections.

```mermaid
graph TD
    A[extract] --> B[features]
    B --> C[Model1.linear_predictions]
    B --> D[Model2.ensemble_predictions]
    B --> E[Model3.linear_evaluation]

    style A fill:#ffc107,color:#000000
    style B fill:#28a745,color:#ffffff
    style C fill:#17a2b8,color:#ffffff
    style D fill:#17a2b8,color:#ffffff
    style E fill:#17a2b8,color:#ffffff
```

## 2. Multi-Level Tree Pattern

Hierarchical branching structure where each level automatically connects to its parent.

```mermaid
graph TD
    A[extract_root] --> B[raw_data]

    B --> C[FE.preprocess]
    B --> D[ML.prepare_ml]

    C --> E[FE.preprocessed_data]
    D --> F[ML.ml_ready_data]

    E --> G[FE.Basic.basic_features]
    E --> H[FE.Advanced.advanced_features]

    F --> I[ML.Linear.linear_model]
    F --> J[ML.Ensemble.ensemble_model]

    style A fill:#ffc107,color:#000000
    style B fill:#28a745,color:#ffffff
    style C fill:#6f42c1,color:#ffffff
    style D fill:#6f42c1,color:#ffffff
    style E fill:#28a745,color:#ffffff
    style F fill:#28a745,color:#ffffff
    style G fill:#17a2b8,color:#ffffff
    style H fill:#17a2b8,color:#ffffff
    style I fill:#17a2b8,color:#ffffff
    style J fill:#17a2b8,color:#ffffff
```

## 3. Merge Pattern

Multiple separate pipelines that can be combined, with optional merge functionality.

```mermaid
graph TD
    A[extract_data] --> D[extracted_data]
    B[process_features] --> E[processed_features]
    C[train_final_model] --> F[advanced_features]

    D --> G[evaluate_final_model]
    E --> G
    F --> G

    G --> H[model_metrics]

    style A fill:#ffc107,color:#000000
    style B fill:#ffc107,color:#000000
    style C fill:#ffc107,color:#000000
    style D fill:#28a745,color:#ffffff
    style E fill:#28a745,color:#ffffff
    style F fill:#28a745,color:#ffffff
    style G fill:#dc3545,color:#ffffff
    style H fill:#17a2b8,color:#ffffff
```

## 4. Grid Pattern (Cartesian Product)

Create all combinations of feature engineering approaches × modeling approaches.

```mermaid
graph TD
    A[extract_data] --> B[raw_data]

    B --> C[FE1.basic_fe]
    B --> D[FE2.advanced_fe]

    C --> E[FE1.fe1_features]
    D --> F[FE2.fe2_features]

    E --> G[FE1.Linear.fe1_linear_model]
    E --> H[FE1.Ensemble.fe1_ensemble_model]
    F --> I[FE2.Linear.fe2_linear_model]
    F --> J[FE2.Ensemble.fe2_ensemble_model]

    G --> K[FE1.Linear.fe1_linear_pred]
    H --> L[FE1.Ensemble.fe1_ensemble_pred]
    I --> M[FE2.Linear.fe2_linear_pred]
    J --> N[FE2.Ensemble.fe2_ensemble_pred]

    style A fill:#ffc107,color:#000000
    style B fill:#28a745,color:#ffffff
    style C fill:#6f42c1,color:#ffffff
    style D fill:#6f42c1,color:#ffffff
    style E fill:#28a745,color:#ffffff
    style F fill:#28a745,color:#ffffff
    style G fill:#dc3545,color:#ffffff
    style H fill:#dc3545,color:#ffffff
    style I fill:#dc3545,color:#ffffff
    style J fill:#dc3545,color:#ffffff
    style K fill:#17a2b8,color:#ffffff
    style L fill:#17a2b8,color:#ffffff
    style M fill:#17a2b8,color:#ffffff
    style N fill:#17a2b8,color:#ffffff
```

---

## Pattern Reference Table

| Pattern              | Description                                                         | Code Example                                                                                                                                                                                                                           | Diagram                                        |
| -------------------- | ------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| **Simple Branching** | One parent branches to multiple children with automatic connections | `parent = ComposablePipeline(data_prep_pipeline)`<br>`parent.add_child("Model1", model1_pipeline)`<br>`parent.add_child("Model2", model2_pipeline)`<br>`parent.add_child("Model3", model3_pipeline)`                                   | [Pattern 1](#1-simple-branching-pattern)       |
| **Multi-Level Tree** | Hierarchical branching with automatic parent-child connections      | `root = ComposablePipeline(root_pipeline)`<br>`fe_branch = root.add_child("FE", fe_pipeline)`<br>`fe_branch.add_child("Basic", basic_fe_pipeline)`<br>`fe_branch.add_child("Advanced", advanced_fe_pipeline)`                          | [Pattern 2](#2-multi-level-tree-pattern)       |
| **Merge Pattern**    | Separate pipelines combined with optional merge functionality       | `branch1 = ComposablePipeline(extraction_pipeline)`<br>`branch2 = ComposablePipeline(feature_pipeline)`<br>`# Use .merge() for automatic fan-in`<br>`merged = branch1.merge(combine_func, "final_output")`                             | [Pattern 3](#3-merge-pattern)                  |
| **Grid Composition** | Cartesian product: FE approaches × Model approaches                 | `for fe_name, fe_pipeline in fe_approaches.items():`<br>`    fe_branch = root.add_child(fe_name, fe_pipeline)`<br>`    for model_name, model_pipeline in models.items():`<br>`        fe_branch.add_child(model_name, model_pipeline)` | [Pattern 4](#4-grid-pattern-cartesian-product) |

### Generated Namespaces Examples

| Pattern          | Generated Namespaces                                                                                                                                                                                                    |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Simple Branching | Root: `extract` → `features`<br>Children: `Model1.linear_predictions`, `Model2.ensemble_predictions`, `Model3.linear_evaluation`                                                                                        |
| Multi-Level Tree | Root: `extract_root` → `raw_data`<br>Level 1: `FE.preprocessed_data`, `ML.ml_ready_data`<br>Level 2: `FE.Basic.basic_features`, `FE.Advanced.advanced_features`, `ML.Linear.linear_model`, `ML.Ensemble.ensemble_model` |
| Merge Pattern    | Individual: `extracted_data`, `processed_features`, `advanced_features`<br>Merged: `model_metrics` (fan-in from all inputs)                                                                                             |
| Grid Composition | Grid: `FE1.Linear.fe1_linear_pred`, `FE1.Ensemble.fe1_ensemble_pred`<br>`FE2.Linear.fe2_linear_pred`, `FE2.Ensemble.fe2_ensemble_pred` (2×2=4 combinations)                                                             |

---

## Key Features

- 🚀 **Automatic Connections**: Parent outputs automatically connect to matching child inputs
- 🏗️ **Namespace Management**: Hierarchical namespaces generated automatically
- 🔗 **Merge Functionality**: Built-in `.merge()` method for fan-in patterns
- 📊 **Visualization**: ASCII pipeline visualization with `draw_pipeline()`
- 🔄 **Recursive Building**: Handles multi-level nested structures seamlessly