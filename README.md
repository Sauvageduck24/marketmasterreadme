# Market Master: Algorithmic Trading Framework

\<div align="center"\>
\<img src="market%20master%20logo.jpg" alt="Market Master Logo" width="250"/\>

<br>

\</div\>

## 🎯 Project Overview

**Market Master** is an algorithmic trading framework in Python designed with one main focus: **methodological rigor and robust validation**. Unlike many platforms, the goal of this project is not the number of strategies, but rather the creation of a system that closes the gap between backtesting performance and real-world execution results.

The fundamental pillar of the system is combating lookahead bias and simulating realistic market conditions at every stage of development.

-----

## 🏗️ Workflow and Architecture

The system is structured in a logical pipeline that runs from idea to live execution.

### 1. Model Definition (`MODELS/utils/base_strategy.py`)

All strategies inherit from a base class. This standardizes signal generation (`entry`, `exit`) and ensures that the processing logic is consistent and reusable.

### 2. Rigorous Training and Validation (`MODELS/train.py`)

This is where the core of the system's rigor lies. Training is **not** a simple train/test split.

* **Walk-Forward Validation***: Models are trained and validated using walk-forward splits (progressive cross-validation), which better simulates how a model would operate over time, with periodic retraining.
* **Lookahead Bias Prevention***: This is the most critical point. Signals generated on the $t$ candle are **shifted** to the $t+1$ candle. This simulates real-world latency: the system receives the information from the closed candle, processes it, and executes the order on the *next* candle.
* **Signal Cleanup**: Stateful logic is applied to ensure that only one trade is processed at a time. A new entry signal cannot be generated if an open position already exists, and exit signals are ignored if there is no previous entry.

### 3. Ranking and Selection (MODELS/rank.py)

After walk-forward training, this module generates a ranking of all individual models based on key metrics (e.g., Sharpe, Calmar, Profit Factor). This allows filtering and selecting only the most robust strategies.

### 4. Model Ensemble (Ensemble)

Instead of relying on a single model, the system creates an ensemble meta-model using the top $N$ models in the ranking (e.g., the top 100). The signals from these models are combined to generate a final consensus signal.

### 5. Advanced Risk Management (Ensemble Level)

The ensemble does not operate blindly. A superior risk management layer is applied that **blocks trades** under unfavorable conditions:

* Blocking during high-impact news events.
* Blocking if a **daily loss limit** (drawdown) is reached.
* Blocking during **anomalous events** detected in data preprocessing (e.g., extreme volatility clusters).

### 6. Report (.html and .json)

The final results of the assembly's backtest are exported to an interactive HTML report and JSON for further analysis.

-----

## 🛡️ The Central Challenge: Backtest vs. Reality

The greatest achievement of this project is the solution to the problem of the discrepancy between the backtest and actual execution.

This is achieved in two ways:

1. **Vectorized Backtesting (in `train.py`): Fast and efficient for optimization, but inherently "perfect." This is mitigated by signal shifting.
2. **Real Execution Simulation (in `REAL/replay/`): This is the ultimate test. This script simulates a real-life execution, candle by candle, (event-driven) on historical data. Unlike the vectorized backtest, the script iterates through each candle one by one, without knowledge of future data, exactly replicating how the bot would operate live.

Finally, `REAL/main.py` contains the bot's main executable for operating in real time, consuming the trained models and applying the same logic validated in the replay.

-----

## 🧠 Methodological Focus and Capabilities

This project demonstrates a deep understanding of key issues in quantitative trading:

* **Lookahead Bias Management***: Implementation of explicit techniques (signal shifting) to prevent pre-observation bias.
* **Robust Validation**: Use of *Walk-Forward Optimization* instead of simple validation, which prevents overfitting.
  ) over time.
* **Data Analysis (`DATA/preprocessing`): Data mining capabilities to identify market regimes, clusters, and anomalies used for risk management.
* **Production Architecture**: A clear separation between research (*backtesting*) and production (*live trading*), with a replay module serving as a final validation bridge.

-----

## 💻 Technology Stack

* **Core**: Python
* **Data Analysis**: Pandas, NumPy, Polars, VectorBT
* **Machine Learning**: CatBoost, Optuna, Scikit-learn
* **Execution**: MetaTrader 5 API
* **Visualization/Web**: Flask, Bootstrap (for the monitoring dashboard)

-----

## ⚠️ Risk Disclosure

**IMPORTANT**: This software has been developed for research and educational purposes. Financial trading involves substantial risk of loss and is not suitable for all investors. Past performance does not guarantee future results. Always test any strategy thoroughly in a demo environment before trading live.
