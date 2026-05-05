# Traversal Cost Prediction & Pathfinding

A machine learning and search algorithm pipeline for predicting terrain traversal costs and finding optimal paths across a grid. The notebook trains regression models on traversal cost data, applies the best model to estimate costs across a provided grid, then runs four classic search algorithms to find the cheapest path from corner to corner.

---

## Overview

The notebook addresses a two-part problem:

1. **Regression** — predict the traversal cost of a grid cell given terrain and environmental features.
2. **Pathfinding** — use the predicted cost grid to compare the performance of DFS, BFS, Dijkstra, and A* in finding a least-cost path from `(0,0)` to `(N-1, N-1)`.

---

## Dataset

**Training data:** `traversal_cost_data.csv`

**Pathfinding grid:** `provided_grid.csv`

**Key features:**
- `type_of_terrain` — categorical terrain type (one-hot encoded)
- `zone_classification` — categorical zone label (one-hot encoded)
- `time_of_day` — categorical time period (one-hot encoded)
- Various numeric features (scaled with `StandardScaler`)
- **Target:** `traversal_cost` (continuous)

---

## Notebook Structure

### 1. Data Preprocessing

- Loads `traversal_cost_data.csv` and inspects shape, types, missing values, and duplicates
- One-hot encodes the three categorical columns (`type_of_terrain`, `zone_classification`, `time_of_day`) with `drop_first=True`
- Splits into 80/20 train/test sets (`random_state=8`)
- Applies `StandardScaler` to all numeric features (fit on training set, transform applied to both)

---

### 2. Linear Regression

Trains a baseline `LinearRegression` model and evaluates with MAE, MSE, and RMSE. A scatter plot of predicted vs. actual values (with a perfect-prediction reference line) visualises fit quality.

---

### 3. Polynomial Regression (Degree 4)

Expands features to degree-4 polynomial terms, re-scales them, and trains another `LinearRegression`. Evaluated with:

- Train and test R²
- Train, test, and 5-fold cross-validated RMSE

A predicted vs. actual scatter plot is produced. **This model is selected for grid prediction** based on its performance.

---

### 4. Neural Network

Trains a Keras `Sequential` regression network:

| Layer | Units | Activation | Regularisation |
|---|---|---|---|
| Dense | 264 | ReLU | — |
| Dense | 128 | ReLU | L2 (0.001) |
| Dense | 64 | ReLU | — |
| Output | 1 | Linear | — |

- Optimiser: Adam (lr=0.001), loss: MSE
- Early stopping on validation loss (patience=10, restores best weights)
- Up to 200 epochs, batch size 64, 20% validation split
- Evaluated with MAE, MSE, RMSE and a scatter plot

---

### 5. Grid Prediction

Applies the polynomial regression model to `provided_grid.csv`:

- One-hot encodes and aligns columns to match the training feature set
- Scales numeric columns using the training scaler
- Applies the polynomial transformer and its scaler
- Saves predictions to `Estimated_grid.csv`
- Prints prediction range and summary statistics
- Renders a heatmap (`YlOrRd` colour scale) of predicted traversal costs across the grid

---

### 6. Pathfinding

Loads `Estimated_grid.csv` and reshapes the predicted costs into an N×N grid (requires a square number of rows). Implements four search algorithms using 4-directional moves (up, down, left, right):

| Algorithm | Strategy | Cost-Optimal |
|---|---|---|
| **DFS** | Depth-first (stack) | No |
| **BFS** | Breadth-first (queue) | No (uniform cost only) |
| **Dijkstra** | Priority queue on cumulative cost | Yes |
| **A\*** | Priority queue on cost + Manhattan heuristic | Yes |

Each algorithm returns the total path cost and the number of nodes visited. Results are printed for comparison.

---

## Dependencies

```
pandas
numpy
scikit-learn
matplotlib
seaborn
tensorflow / keras
heapq  (standard library)
collections  (standard library)
```

---

## Usage

1. Place `traversal_cost_data.csv` and `provided_grid.csv` in the same directory as the notebook.
2. Run cells sequentially — the pathfinding section depends on `Estimated_grid.csv` produced in the grid prediction step.
3. `Estimated_grid.csv` is written to the working directory automatically.

---

## File Structure

```
.
├── AI_assignment_final_code.ipynb   # Main notebook
├── traversal_cost_data.csv          # Training data (required)
├── provided_grid.csv                # Grid to predict costs for (required)
├── Estimated_grid.csv               # Output: grid with predicted costs
└── README.md                        # This file
```
