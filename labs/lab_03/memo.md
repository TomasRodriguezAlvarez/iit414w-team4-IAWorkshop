# Technical Memo — Predicting Formula 1 Championship Points

## Overview

The objective of this analysis was to estimate how many championship points a Formula 1 driver is likely to score in a given race. This type of prediction is useful for strategic decision-making, such as evaluating the potential impact of race positions on the Constructors’ Championship.

To address this, we built and compared several predictive models using historical race data from 2019 to 2024. The models were trained on past seasons (2019–2023) and tested on the 2024 season to simulate a realistic forecasting scenario.

---

## Key Results

The best-performing model was **Linear Regression**, with an average prediction error of approximately **5.55 points per race**.

However, this result was only slightly better than a very simple baseline model that always predicts the average number of points (error ≈ 5.55). This indicates that even basic approaches perform nearly as well as more advanced models in this setting.

More complex models, such as Random Forest and XGBoost, did not improve performance. In fact, they showed worse results on new data, suggesting that they were overfitting to past races rather than learning general patterns.

---

## What This Means

The results suggest that predicting exact championship points is a **challenging task with the available data**.

One key reason is that more than half of the race entries result in zero points, which makes outcomes difficult to distinguish. Additionally, important factors that influence race results—such as weather conditions, mechanical failures, safety cars, and race strategy—are not included in the dataset.

While starting position (grid) and team/driver identity do provide some predictive signal, they are not sufficient to produce highly accurate forecasts on their own.

---

## Recommendation

Given the results, the recommended approach is to use a **simple and stable model such as Linear Regression** for estimating expected points.

Although its accuracy is limited, it performs consistently and avoids the risk of overfitting seen in more complex models. This makes it a more reliable tool for high-level planning and scenario estimation.

---

## Limitations and Risks

* The model does not account for unpredictable race events (e.g., accidents, weather, strategy decisions).
* Predictions have a relatively high error (~5.5 points), which should be considered when making decisions.
* The dataset lacks detailed performance indicators that could improve accuracy.

---

## Conclusion

This analysis shows that while it is possible to estimate expected championship points, predictions remain uncertain due to the complexity of Formula 1 races.

Simple models provide the most reliable results given the available data, but additional information would be required to significantly improve prediction accuracy.
