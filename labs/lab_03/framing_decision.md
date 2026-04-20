Business Question

Is it possible to predict how many championship points a driver will contribute in a specific Grand Prix based on their starting grid position and the team's historical performance?
This prediction is vital for the Strategy Director to calculate the return on investment for car upgrades and prioritize defending critical race positions to ensure FIA ​​prize money.

Target Variable
The target variable will be points (points obtained at the end of the race). It is defined as a continuous numerical variable ranging from 0 to 25 (or 26 if the fastest lap is included), representing the driver's tangible result for the Constructors' Championship.


Metric
We will use MAE (Mean Absolute Error). This metric is the most appropriate for this business problem because it measures error in the same units as the target (points). For a team leader, it is much more intuitive to understand that the model "is off by an average of 2.5 points" than to interpret quadratic metrics like RMSE, which would disproportionately penalize large errors.


Rejected Alternative
We considered using a Binary Classification approach (Top-10 Finish: Yes/No), but rejected it as insufficient for strategic decision-making. In Formula 1, the financial and competitive difference between a P1 (25 pts) and a P10 (1 pt) is enormous. A binary model would treat both outcomes as equally "successful," obscuring the risk of losing 24 critical points. By choosing regression, we maintain the granularity needed to distinguish between a podium finish and simply placing in the points.
