# Software Engineer Salary Prediction

A machine-learning project that estimates annual software-engineer compensation in USD from country, education, and years of professional experience. A [Streamlit](https://streamlit.io/) app serves two views: a prediction form backed by a saved decision tree, and charts over the [2023 Stack Overflow Developer Survey](https://survey.stackoverflow.co/2023/).

## What the app does

**Predict.** Choose a country, an education level, and years of experience. The app encodes those inputs with the same label encoders used in training and returns an estimated yearly salary.

**Explore.** Summarizes full-time respondents who reported a salary:

- share of respondents by country
- mean salary by country
- mean salary by years of professional experience

The charts keep countries with at least 1,000 respondents and salaries from $10,000 to $250,000. Those display filters are separate from the training filters described below.

## Data

Source: 2023 Stack Overflow Developer Survey (`survey_results_public.csv`, stored with [Git LFS](https://git-lfs.com/), about 151 MB).

The target is `ConvertedCompYearly`, renamed to `Salary`: total yearly compensation converted to USD. The model uses three inputs.

| Feature | Survey column | What the model sees |
| --- | --- | --- |
| Country | `Country` | 17 countries with at least 400 full-time respondents |
| Education | `EdLevel` | Four buckets: Less than a Bachelors, Bachelor's degree, Master's degree, Post grad |
| Experience | `YearsCodePro` | Years as a number. "Less than 1 year" is 0.5. "More than 50 years" is 50 |

Post grad covers a professional degree or other doctorate (the survey's JD, MD, Ph.D., and Ed.D. responses). Everything short of a bachelor's degree is grouped into Less than a Bachelors.

Training rows are full-time employed respondents with a non-null salary between $10,000 and $300,000. Countries below the 400-respondent cutoff are dropped rather than predicted as "Other".

Countries the model can score:

Australia, Brazil, Canada, Denmark, France, Germany, India, Israel, Italy, Netherlands, Norway, Poland, Spain, Sweden, Switzerland, United Kingdom of Great Britain and Northern Ireland, United States of America.

## Model

Training and model comparison live in `SalaryPrediction.ipynb`. Country and education are label-encoded, then three regressors are fit in scikit-learn.

| Model | Training RMSE |
| --- | ---: |
| Linear regression | $50,346 |
| Decision tree, unrestricted depth | $37,759 |
| Random forest | $37,830 |
| Decision tree, `max_depth=10` (deployed) | $38,688 |

The deployed model is a `DecisionTreeRegressor` with `max_depth=10` and `random_state=0`. Depth was chosen with `GridSearchCV` over `{None, 2, 4, 6, 8, 10, 12}` using negative mean squared error. The tree and both label encoders are saved together in `saved_steps.pkl`.

The RMSE figures above are computed on the same rows the models were fit on. They are a useful comparison during development, and they are optimistic relative to error on new respondents. Limiting depth keeps the deployed tree smaller than an unrestricted tree fit on the same data.

A notebook check for the United States, a master's degree, and 15 years of experience returns about **$176,015**.

## Project layout

```
app.py                     Streamlit entry point (Predict / Explore)
predict_page.py            Prediction form
explore_page.py            Survey charts
SalaryPrediction.ipynb     Cleaning, model comparison, and model export
saved_steps.pkl            Trained tree and label encoders
survey_results_public.csv  2023 Stack Overflow survey (Git LFS)
requirements.txt
```

## Run it locally

Use Python 3.9. `requirements.txt` pins `scikit-learn==1.3.0`, which is the version that produced `saved_steps.pkl`, and `matplotlib==3.4.2`. The prediction page only needs the pickle. The explore page also needs the survey file.

```bash
git lfs install
git clone https://github.com/sakshamrai101/SWE_Salary_Prediction.git
cd SWE_Salary_Prediction
git lfs pull

python3.9 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

streamlit run app.py
```

On Windows, activate the environment with `.venv\Scripts\activate`.

Open the local URL Streamlit prints, leave **Predict** selected, choose inputs, and click **Calculate Salary**. Switch the sidebar to **Explore** for the charts.

## Limits

- Compensation here depends only on country, education, and experience. Role, company size, and city are not in the model.
- Figures are self-reported survey responses converted to USD, not verified offers.
- Salaries outside $10,000–$300,000 were removed, so the model is not meant for very low or very high compensation.
- Countries with fewer than 400 full-time respondents cannot be scored.
- Treat the result as a rough estimate from 2023 survey data.
