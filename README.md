# Building a Limit Order Book and Market Maker Simulator in Python

## Introduction
This project is a simulation platform developed in Python, focusing on **Market Microstructure**. It features a robust **Limit Order Book (LOB)** and implements a **Market Maker (MM)** agent. The core idea is to create a controlled setting to observe how MM operational decisions influence the LOB, manage **inventory risk**, and determine final profitability.

---

## Simulation Core: Limit Order Book Implementation

The `OrderBook` class is engineered for performance and strict adherence to price-time priority using a hybrid data structure:

* **HashMaps and Deques:** `self.bids` and `self.asks` are dictionaries mapping prices to `collections.deque`. This enables **O(1)** additions and cancellations at specific price levels, preserving **time priority**.
* **Bisect for Price Ordering:** The built-in `bisect` module is used to maintain sorted price lists (`bid_prices` descending, `ask_prices` ascending) efficiently. This ensures **O(log N)** insertion for new price levels and fast access to the **Best Bid** and **Best Ask**.
* **OID Mapping:** Each order receives a unique OID tracked via an `oid_map` dictionary, allowing **O(1)** order lookups essential for fast cancellations by the Market Maker.



---

## Generation of Realistic Market Flows

To simulate the volatile and dynamic market environment, the `run_simulation_and_return` loop incorporates the following mechanisms:

* **Random Walk Price:** The "true price" of the underlying asset follows a **Random Walk** with defined volatility, mimicking basic price movement.
* **Noise Traders:** Order arrivals are governed by a **Poisson distribution**. These traders randomly place **Limit Orders** (adding liquidity) or aggressive **Market Orders** (taking liquidity), creating the continuous flow and "noise" the MM must navigate.



---

## Market Maker Logic: Inventory Skewing Strategy

The `MarketMaker` class incorporates a quantitative **Inventory Skewing** strategy to manage exposure and maximize spread capture:

1.  **Dynamic Updates:** The `quote(ts)` method cancels all previous active quotes and submits new Limit Orders centered around the adjusted Reservation Price.
2.  **Mark-to-Market Valuation (MtM):** Performance is tracked using **MtM Valuation** (`get_valuation`), which provides a complete financial view by valuing the remaining inventory at the current Midprice, instead of relying solely on realized PnL.

## Technical Choices and Analysis

| Feature | Motivation |
| :--- | :--- |
| **Bisect + HashMap** | Ensures **O(log N)** price level handling and **O(1)** order access for peak performance. |
| **Unique OID Tracking** | Essential for accurately distinguishing the Market Maker's orders from background noise and for precise PnL attribution. |
| **Mark-to-Market PnL** | Provides a complete and realistic view of profitability by accounting for unrealized gains/losses on inventory. |
| **Event-driven MM** | The Market Maker reacts dynamically to trades and time steps, adjusting quotes to optimize spread capture while managing risk. |

