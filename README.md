# Radioactivity_Analysis

Measuring and analyzing radioactivity in air via beta decay curves. Further scientific background can be found in the Jupyter Notebook `Radioactivity_In_Air` (see repository structure below).

## Repository Structure

Primary data analysis is done using Jupyter Notebooks in the `notebooks` folder. Raw data can be found in the `data` folder and cached in the `notebooks/cache` folder.

To run the analysis yourself, instantiate your python environement of choice and run `pip install -r requirements.txt` to get all dependencies.

## Dependencies

1. `numpy`
2. `pandas`
3. `scikit-learn`
4. `jupyterlab`
5. `matplotlib`
6. `sklearn`
7. `scipy`

# Academic Background

## Introduction

The goal of this experiment is to measure and identify radioactive substances in the air with high accuracy and specificity. Isotopes of interest include:

1. Uranium-238 => Lead-206
2. Thorium-232 => Lead-208
3. Uranian-235 => Lead-207

### Equipment

1. Geiger-Mueller Tube: Radiation measurement.
    * Tube has ionized gas, any ionizing radiation causes a negative charge accumulation.
2. Picker-Scaler: Power supply.
3. DAQ/Computer Setup.

## Data Collection

### Session I: Cesium-137 Calibration Data for Voltage & Background Radiation Measurement

- Collecting data for Cesium-137 source.
- Plot data in python, taking plateau curve for data ⇒ determine operating voltage.
- Collecting data on background radiation.

### Session 2:

- Air sampler used to retrieve air samples ⇒ analysis of the model.
- "Radioactivity in Air" program is used to collect data from Geiger-Mueller tubes (counts number of beta(?) particle emissions in a given time span).
- Note that the filter on the GM tube is thin enough to let in **beta particles** and thick enough to **stop alpha particles**.
    - Therefore, we are counting **only beta particles**.
    
# The Model

## Approach to Data Analysis

- Multiple levels of sophistication:
- **EASY METHOD:**
    - Constant background subtraction of your data
    - Use a semi-logarithmic plot, find rough **half-life** and **total counting rate** by the end of the sampling period.
- **Full-Blown Analysis:**
    - Observing gamma-rays from filter paper.

## Parts of the Model

Model for **counts registered** in the detector:

### Part I: **Constant background** from no sample:

1. Result of cosmic rays and gamma-rays in the room.
2. Average number of counts per time period is easily calculated from the background data collection.
3. The goal is `<= 1 count per minute` of error.
4. Determine the background under **identical conditions** to the experiment.
5. **If you observe `n` counts, the standard error is `\sqrt{n}`**

### Part II: **Decay of long-lived isotopes accumulated on filter paper:**

1. Dominated by decay of "4n" series made from Thorium-232
    1. Results in Rn-220 (0.145s) → Po-216 (10.64h) → Bi-212 (1.00h) → Po-212 (0.3 us)
2. Half life of 10.64 hours for Po-216 will dominate activity.
3. Longer-lived components can be identified with `semi-logarithmic` plot of **detector counts with background subtracted.**
    1. Only for times after the half-hour activity from Rn-222 have decayed.
    2. Only the **first half hour** of data is relevant to the short-lived products of **Radon-222.**
4. `In Python`: Make a **linear fit** of data **after the half-hour** from the air sample.
    1. If good linear fit: subtract this component from the total data ⇒ corrected data ⇒ third part of the model.

### Part III: **Decay of products of Radon-222**:

**Secular Equilibrium:**

- Radon-222 → Po-218. Po-218 is highly *ionized* ⇒ becomes attached to dust.
    - The next beta decay produce Pb-214 and B-214 (also highly ionized), with half lives of ~0.5 hours.
    - These products also attach themselves to dust particles.
- Number of **isotopic breakdowns** per second for each is proportional to the amount of that isotope that is present.
    - More Po-218 ⇒ more decay per second ⇒ build up an equilibrium depending on lifetime of Po-218 and the amount being formed from it's **predecessor Radon.**
    - **At equilibrium**: Number of Po-218 created = number of Po-218 destroyed per second.
        - This type of equilibrium continues down the chain...
    - This is called **`SECULAR EQUILIBRIUM` ⇒** holds whenever **parent** has long half life compared to next generation(s).
- Therefore, **MEASUREMENT OF DISINTEGRATION RATE OF ANY MEMBER OF THE SERIES ⇒ ACTIVITY OF RADON PRESENT.**
- All beta activity in air can be accounted for given decay of Pb-214 and Bi-214 (after alpha particles were eliminated by that aluminum filter).
- *If model fits data*: If we determine the amount of **`Pb-214`,** we can get the **CONCENTRATION OF RADON IN THE ATMOSPHERE.**

## Mathematical Model of Daughters of Radon-222

- After filtering is stopped, the few alpha-emitting Po-218 atoms don't affect much, especially after the first 10-15 minutes where it increases the amount of Pb-214.
- Subsequent counting rate = only because of **Pb-214 and Bi-214**.
- Rest of the model will neglect any build-up of Pb-214 from Po-218 on the filter paper.

...

- Variation of counting rate over time = f(decay constant_Pb-214, decay constant_Bi-214).
    - A_B(t) = *actual activity of Pb-214 (RaB) at time t.*
    - A_C(t) = *actual activity of Bi-214 (RaC) at time t.*
- The **ratio of isotope B (Pb-214)** and **isotope C (Bi-214)** at time zero is:

$$R = \frac{A_c(0)}{A_B(0)}$$

- `Observed counting rate` = `real activity` x `efficiency`  (efficiency = \varepsilon)
    - efficiency is not 1 because low-energy beta particles tend to be stopped by the aluminum foil.
- Beta particle energy spectra ranges from `0 -> E_max`
    - `E_max` is higher for Bi-214 than for Pb-214
    - \varepsilon is therefore larger for Bi-214
    - For 27 um thick foil and window of density 1.5mg/cm^2, we have:

$$\varepsilon_C = 0.95, \varepsilon_B = 0.80 ⇒ \varepsilon_C/\varepsilon_B = 0.1$$

- If we say

$$A^{obs}_B(t) = \varepsilon_B A_B(t); \,\,\, A^{obs}_C(t) = \varepsilon_CA_C(t)$$

- Where *obs* denotes that it is what we actually count in the experiment and non-*obs* is what is really happening/being emitted, then

$$\frac{A^{obs}_C(0)}{A^{obs}_B(0)} = \frac{\varepsilon_C}{\varepsilon_B} \frac{A_C(0)}{A_B(0)} = \frac{\varepsilon_C}{\varepsilon_B}R$$

- Recall that Po-218 is neglected because of its short lifetime...
- The following relatonships are for

$$\lambda = \frac{\ln 2}{\text{half-life}}$$

$$A_B^{obs}(t) = A_B^{obs}(0)e^{-\lambda_B t}$$

$$A_C^{obs}(t) = \frac{\varepsilon_C}{\varepsilon_B} A_B^{obs}(0)\lbrack \frac{\lambda_C}{\lambda_C - \lambda_B} e^{-\lambda_Bt} + ( R - \frac{\lambda_C}{\lambda_C - \lambda_B} ) e^{-\lambda_C t} \rbrack$$

$$A_B^{obs}(t)+A_C^{obs}(t) = A_B^{obs}(0) \times \lbrack(1 + \frac{\varepsilon_C}{\varepsilon_B}\frac{\lambda_C}{\lambda_C-\lambda_B})e^{-\lambda_B t} + \frac{\varepsilon_C}{\varepsilon_B} ( R - \frac{\lambda_C}{\lambda_C - \lambda_B} ) e^{-\lambda_C t} \rbrack$$

- the total observed counting rate is (A_b, obs(t) + A_c,obs(t)

## Comparison of Model to Data

- Part III ⇒ predicted decay curves given `A_B,obs` and `R`
- Goal of the advanced model is to get a good fit (after applying steps 1-2)
- First approximation of A_B,obs is from extrapolating curve back to t=0, noting that

$$A_B^{obs}(0) + A_C^{obs}(0) = A_B^{obs}(0)[1+\frac{\varepsilon_C}{\varepsilon_B}]$$

- First approximation of R based on graph from *Whyte and Taylor* that states that R is a function of sampling time:

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dac2fecc-08c0-4a1d-86da-91757c13ed82/Screen_Shot_2020-03-16_at_1.33.21_AM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dac2fecc-08c0-4a1d-86da-91757c13ed82/Screen_Shot_2020-03-16_at_1.33.21_AM.png)
