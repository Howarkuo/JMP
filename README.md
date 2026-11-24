# JMP Practice
## Chemical Mechanical Planarization / Polishing (CMP) Experiments

This project uses a 16-run, two-level fractional factorial experiment to study six CMP factors and two measured responses. The process objective is to achieve the required material removal rate while keeping removal uniform across the wafer. For this practice exercise, use **Maximize** for polish rate and **Minimize** for standard deviation. A production process may instead specify a target polish rate.

![CMP process and practice overview](https://hackmd.io/_uploads/B1y1-flRGl.png)

![CMP experiment data table](https://hackmd.io/_uploads/ByP7IyeCzl.png)

## 1. Physical meaning of the columns

The carrier holds the wafer face-down against a polishing pad mounted on a rotating table, also called a platen. Slurry supplies the chemistry and abrasive particles at the wafer–pad interface.

| Column | Levels in this experiment | Physical meaning |
|---|---|---|
| Down Force | 180, 250 | Applied downward load that affects wafer–pad contact pressure. |
| Back Force | 50, 100 | Typically loading from behind the wafer inside the carrier. It changes contact-pressure distribution across the wafer. The exact actuator requires the equipment definition. |
| Oscillations | 0, 1 | Two coded motion settings, commonly oscillation off/on. Sweeping changes the wafer path across the pad. Confirm the coding in the original experiment. |
| Pad | 0, 1 | Two pad types. Stiffness, texture, and slurry transport can affect removal and uniformity. |
| Carrier Velocity | 10, 100 | Carrier/wafer rotation-speed setting. |
| Table Velocity | 10, 100 | Platen/pad rotation-speed setting. |
| Y-Polish Rate A/min | Measured | Average thickness removed per minute. Here A means ångström, Å. For example, 869 Å/min = 86.9 nm/min. |
| Y-Standard deviation A/min | Measured | Reported variability associated with polishing, used in this exercise as a wafer-uniformity response. Lower values indicate more uniform removal under the same measurement protocol. |

The table omits force and speed units. Treat their values as equipment settings until the source defines the units. The measurement locations, sample count, and precise standard-deviation calculation also require the source protocol. Some versions of the source label variability in Å, while this JMP table labels it Å/min. Preserve the supplied data and verify the unit before making physical conversions of that response.

A common empirical approximation for local material removal is Preston's equation.

$$
R \approx K_P p v_{\mathrm{rel}}
$$

Here, $R$ is removal rate, $p$ is contact pressure, $v_{\mathrm{rel}}$ is local wafer–pad sliding speed, and $K_P$ is a process-dependent coefficient. Pressure distribution, pad properties, and relative motion can affect both the average removal rate and its spatial variation. Carrier and table rotation jointly determine local relative velocity.

## 2. Experimental design and modeling types

There are six factors at two levels each. A full factorial would have $2^6=64$ combinations. This table contains 16 selected combinations, a quarter fraction of the full design.

| Columns | JMP modeling type |
|---|---|
| Down Force, Back Force, Carrier Velocity, Table Velocity | Continuous |
| Oscillations, Pad | Nominal |
| Both Y response columns | Continuous |

The blue triangles and green bars beside column names indicate continuous and nominal modeling types, respectively. Each row is one experimental condition. The supplied table contains one reported value for each response per condition.

## 3. Least Squares Fit, responses, and Prediction Profiler

![Two response reports and combined Profiler](https://hackmd.io/_uploads/By_pKyl0Ge.png)

| Section in the JMP window | What it does | Why open it |
|---|---|---|
| Least Squares Fit | Parent report for the fitted models. JMP estimates a separate equation for each response using the specified terms. | Inspect the fitted equations and their diagnostics. |
| Response Y–Polish Rate | Reports the model for average removal rate. | Identify associations between settings and throughput. |
| Response Y–Standard Deviation | Reports a separate model for the measured variability response. | Identify associations between settings and uniformity. |
| Prediction Profiler | Evaluates both fitted equations at the displayed settings. Each panel varies one factor while holding the others at their current values. | Explore settings, interactions, and the trade-off between removal rate and variability. |

For response $k$ and run $i$, write the model as

$$
y_{k,i}=f_k(\mathbf{x}_i;\boldsymbol{\beta}_k)+\varepsilon_{k,i}.
$$

The vector $\mathbf{x}_i$ contains the six settings. Each response has its own coefficients $\boldsymbol{\beta}_k$ and residuals $\varepsilon_{k,i}$. Ordinary least squares estimates the coefficients by minimizing

$$
\operatorname{SSE}_k=\sum_{i=1}^{16}\left[y_{k,i}-\widehat{y}_{k,i}\right]^2.
$$

A residual is the measured response minus its fitted value. The models describe the relationship within the tested operating range.

## 4. Fit the model in JMP

### Run the saved script

Click the green triangle beside **Fit Model** in the data table's Scripts panel. In this project, the saved script produced both response reports and a combined Prediction Profiler.

### Rebuild the displayed model manually

1. Open **Analyze → Fit Model**.
2. Select both response columns and click **Y**.
3. Set **Personality → Standard Least Squares**.
4. Select all six process factors and click **Add**.
5. Clear other selections before creating each interaction. Select only **Down Force** and **Carrier Velocity**, then click **Cross**.
6. Repeat for **Pad × Carrier Velocity** and **Carrier Velocity × Table Velocity**.
7. Confirm six main effects and three interactions in **Construct Model Effects**. Keep the intercept included.
8. Click **Run**.

The displayed model has nine effect parameters plus an intercept. With 16 complete observations and full column rank, this leaves six residual degrees of freedom for each response.

### Mathematical form

For clarity, let $d=\mathrm{DownForce}-215$, $b=\mathrm{BackForce}-75$, $c=\mathrm{CarrierVelocity}-55$, and $t=\mathrm{TableVelocity}-55$. Let $o$ and $q$ be 0/1 indicators for oscillation and pad level 1. One equivalent representation is

$$
\widehat y_k=\beta_{0,k}+\beta_{d,k}d+\beta_{b,k}b+\beta_{o,k}o+\beta_{q,k}q+\beta_{c,k}c+\beta_{t,k}t+\beta_{dc,k}dc+\beta_{qc,k}qc+\beta_{ct,k}ct.
$$

JMP can use different nominal contrasts and centering conventions. Coefficients must be interpreted with their actual coding. To retrieve the equation corresponding to the report, use the response red triangle and select **Show Prediction Expression**, or save **Prediction Formula**.

The down-force slope in this representation is

$$
\frac{\partial\widehat y_k}{\partial d}=\beta_{d,k}+\beta_{dc,k}c.
$$

Thus, an interaction allows the effect of one setting to depend on another setting.

## 5. Fitting method and response-surface options

![JMP model specification and response-surface attribute](https://hackmd.io/_uploads/S1b4G9BAGg.png)

| JMP option | Meaning in this project |
|---|---|
| Standard Least Squares | Fits the terms explicitly entered in Construct Model Effects. Use this to reproduce the displayed nine-term model. |
| Stepwise | Selects among supplied candidate terms using a chosen selection rule. A different selection rule or candidate list can produce a different model. |
| Add | Adds selected factors as main effects. |
| Cross | Creates interaction or polynomial terms from selected variables/effects. |
| Macros → Factorial to Degree, Degree 2 | Generates main effects and two-factor interactions. Six factors generate 21 effect terms, exceeding what 16 runs can estimate jointly. |
| Attributes → Response Surface Effect | Marks selected model effects with the RS attribute for response-surface analysis. |
| Macros → Response Surface | Generates main effects, two-way interactions, and quadratic terms for continuous factors, and applies RS attributes. |

A quadratic model for two continuous factors can include

$$
\widehat y=\beta_0+\beta_1x_1+\beta_2x_2+\beta_{12}x_1x_2+\beta_{11}x_1^2+\beta_{22}x_2^2.
$$

The interaction term changes a factor's slope as another factor changes. The squared terms represent curvature with respect to individual factors.

This experiment supports linear changes between the tested factor levels and selected interaction contrasts. For Down Force, the centered values are $-35$ and $+35$, so $d^2=1225$ for every run. That constant is confounded with the intercept. Estimating a separate quadratic effect requires additional levels and an appropriately augmented design. A shared center point can test aggregate curvature, while estimating individual quadratic effects requires enough independent design points.

## 6. Open Effect Tests

1. Click the red triangle immediately left of **Response Y-Polish Rate A/min**.
2. Choose **Regression Reports → Effect Tests**.
3. Scroll within that response section to find the table.
4. Repeat for **Response Y-Standard deviation A/min**.

![Effect Tests and Sorted Parameter Estimates](https://hackmd.io/_uploads/rJ2-ukeRfg.png)

| Report column | Interpretation |
|---|---|
| Source | Main effect or interaction being tested. |
| Nparm / DF | Number of effect parameters and test degrees of freedom. Each displayed term has one degree of freedom. |
| Sum of Squares | Adjusted contribution associated with testing that effect conditional on the other terms in the model. |
| F Ratio | Effect mean square divided by residual mean square. |
| Prob > F | p-value for the effect test under the fitted model and its error assumptions. |
| Estimate in Sorted Parameter Estimates | Coefficient or contrast estimate under the displayed coding. Interpret it alongside interactions and units. |

At a 0.05 threshold, a small p-value provides evidence of an effect conditional on this model. Compare physical changes over the tested range to judge practical size. For these one-degree-of-freedom effects, $F=t^2$, so the F-test and corresponding two-sided coefficient t-test have matching p-values.

### Polish-rate results from the displayed fit

| Term | p-value | Interpretation within the fitted model |
|---|---|---|
| Down Force | <0.0001 | Strongest statistical evidence. Higher force raises predicted polish rate at both tested carrier speeds. |
| Back Force | 0.8202 | Weak evidence over the tested range. |
| Oscillations | 0.0998 | Evidence falls short of the 0.05 threshold. |
| Pad | 0.0586 | Close to the threshold, interpreted at the carrier reference setting. |
| Carrier Velocity | 0.0003 | Associated with polish rate, with a slope dependent on force, pad, and table speed. |
| Table Velocity | 0.0001 | Associated with polish rate, with a slope dependent on carrier speed. |
| Down Force × Carrier Velocity | 0.0007 | Fitted down-force benefit increases with carrier speed. |
| Pad × Carrier Velocity | 0.1304 | Limited evidence for this assigned interaction. |
| Carrier Velocity × Table Velocity | 0.0026 | Fitted table-speed benefit decreases with carrier speed. |

The reported down-force coefficient is 5.3392857 and the centered force–carrier coefficient is 0.0336508. The fitted increase from force 180 to 250 is approximately 268 Å/min at carrier speed 10 and 480 Å/min at carrier speed 100.

### Standard-deviation results from the displayed fit

| Term | p-value |
|---|---|
| Carrier Velocity | 0.0011 |
| Table Velocity | 0.0065 |
| Pad | 0.0446 |
| Pad × Carrier Velocity | 0.0538 |
| Oscillations | 0.0774 |
| Back Force | 0.1058 |
| Down Force | 0.2749 |
| Down Force × Carrier Velocity | 0.2921 |
| Carrier Velocity × Table Velocity | 0.4277 |

Carrier and table speeds have the strongest statistical evidence for the variability response. Their displayed main-effect coefficients are positive at the model's reference settings. Use the Profiler to evaluate their net effects at a particular recipe, including interactions.

## 7. Aliasing in the 16-run design

With low/high coding of −1/+1, the actual run pattern makes these pairs of interaction columns identical.

| Interaction in the saved model | Aliased interaction |
|---|---|
| Down Force × Carrier Velocity | Back Force × Oscillations |
| Pad × Carrier Velocity | Down Force × Table Velocity |
| Carrier Velocity × Table Velocity | Down Force × Pad |

Each fitted coefficient represents a shared contrast. Assigning that contrast to the displayed interaction assumes the omitted aliased interaction is negligible. Stepwise selection uses the same experimental information and inherits this ambiguity. Additional runs designed to break these aliases would support separate estimates.

## 8. Check model fit and residuals

| Response | Reported R² | Reported RMSE |
|---|---|---|
| Polish rate | 0.99241 | 33.684 Å/min |
| Standard deviation | About 0.93 | 75.549 in the supplied response units |

R² describes the proportion of observed response variation explained by the fitted model. RMSE estimates the residual error scale after accounting for the fitted parameters.

**The standard-deviation response and RMSE measure different quantities.** The standard-deviation column is an experimental outcome modeled as a function of machine settings. RMSE measures discrepancies between observed outcomes and their fitted values for a particular regression.

In the polish-rate screenshot, externally studentized residuals for runs 10 and 16 are near the outer simultaneous limits. Review those runs and the residual pattern when assessing the model. With one reported observation per recipe, the table provides no independent pure-error estimate from replicated recipes. Its residual error can contain measurement variation and omitted effects. A high training R² alone does not establish prediction accuracy at new recipes.

## 9. Use the Prediction Profiler

Read each row as one predicted response and each factor column as a slice through its fitted equation.

| Profiler feature | Meaning |
|---|---|
| Red values below factor columns | Currently selected settings. |
| Vertical red dashed line | Current setting for that factor. |
| Black curve or line | Predicted response as that factor varies, holding other settings fixed. |
| Red response number at left | Predicted response at the current complete recipe. |
| Blue interval/shading | Uncertainty interval under the selected Profiler display options. Check the menu to determine whether it shows a mean confidence interval or an individual prediction interval. |
| Rightmost desirability curves | How acceptable each predicted response is under the specified goals and limits. |
| Bottom desirability row | Combined desirability as the settings change. |

A steep line means the predicted response is sensitive to that factor at the current settings. With an interaction, changing another factor can change the slope. A nominal factor switches between its available categories.

### Example from the earlier Profiler

| Factor | Setting |
|---|---|
| Down Force | 250 |
| Back Force | 100 |
| Oscillations | 0 |
| Pad | 0 |
| Carrier Velocity | 10 |
| Table Velocity | 100 |

| Response | Model prediction | Measured value in run 4 |
|---|---|---|
| Polish rate | 1336.125 Å/min | 1327 Å/min |
| Standard deviation | 179.875 | 213 |

Predictions differ from measurements because the equation fits all 16 runs simultaneously. Changing Carrier Velocity from 10 to 100 at this recipe increases both predicted responses in the displayed models.

### Set optimization goals

1. Click the red triangle beside **Prediction Profiler**.
2. Choose **Optimization and Desirability → Set Desirabilities**.
3. Set polish rate to **Maximize**, or **Match Target** when a required rate is specified.
4. Set standard deviation to **Minimize**.
5. Define meaningful limits and importance weights. The screenshots do not provide production acceptance limits.
6. Choose **Optimization and Desirability → Maximize Desirability**.
7. Record the settings, predicted responses, and uncertainty intervals. Keep categorical settings at their available levels and continuous settings within the tested range.
8. Confirm a selected recipe with additional experiments before treating it as an established operating condition.

In the earlier screenshot, the standard-deviation desirability curve increased with standard deviation. Change that goal to Minimize for a uniformity objective. The displayed desirability value of 0.308809 therefore reflected the goals active at that time.

If a combined Profiler is needed, save **Prediction Formula** for each response, open **Graph → Profiler**, and assign both formula columns to **Y, Prediction Formula**. Add standard-error formula columns when you need the associated uncertainty display.

## 10. References

- [applying-statistics-in-quality-projects/a-doe-in-a-manufacturing-environment-part-1](http://blog.minitab.com/blog/applying-statistics-in-quality-projects/a-doe-in-a-manufacturing-environment-part-1) 
- [JMP — Response Options, including Effect Tests and Profiler](https://www.jmp.com/support/help/en/19.2/jmp/response-options.shtml)
- [JMP — Construct Model Effects, macros, and attributes](https://www.jmp.com/support/help/en/19.2/jmp/construct-model-effects.shtml)
- [JMP — Response Surface Design](https://www.jmp.com/support/help/en/19.2/jmp/response-surface-design.shtml)
- [JMP — Stepwise Regression Control Panel](https://www.jmp.com/support/help/en/19.2/jmp/stepwise-regression-control-panel.shtml)
- [Minitab — CMP manufacturing DOE context](https://blog.minitab.com/en/blog/applying-statistics-in-quality-projects/a-doe-in-a-manufacturing-environment-part-1)
- [CMP review — Preston equation and wafer-scale modeling](https://academic.oup.com/jom/article/doi/10.1093/jom/ufae055/7915492)

Numerical results above are transcribed from the JMP tables and Profiler screenshots discussed in this project. Force/speed units and the variability measurement protocol remain to be verified against the original experimental documentation. The image links are preserved from the supplied project notes.


