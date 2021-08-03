# Patient Electronic Medical Record Adoption in the US

## Background and Motivation

The aims of this study are to attempt to understand patient demographic and health-related factors associated with the availability and use of electronic medical records (EMRs) in the US.

Understanding which factors drive availability will help healthcare providers (HCPs) and practice managers determine which patients might need increased education about or assistance with accessing an EMR.

Understanding the factors associated with actual use of EMRs will also indicate whether patients who most need access to or easy transportability of their medical records (again, patients with multiple chronic conditions or poorer health, who may need to see multiple HCPs or may have more frequent hospital or physician visits) are actually using them.

For both of these outcomes, whether availability and adoption are changing over time and whether there was a discernible change after the onset of the COVID-19 pandemic will be assessed as part of the models.

The datasets for this analysis were publicly-available from the [Health Information National Trends Survey (HINTS)](https://hints.cancer.gov). This is an annual survey conducted by the [National Cancer Institute](https://www.cancer.gov) with the aim of:
> (collecting) nationally representative data about the American public’s knowledge of, attitudes toward, and use of cancer- and health-related information. HINTS data are used to monitor changes in the rapidly evolving fields of health communication and health information technology and to create more effective health communication strategies across different populations.

Data for Cycles 2-4 (2018-2020) were used. The 2020 data contain post-pandemic responses.  
Two outcome variables were derived from the HINTS fields:
1. EMR Availability: Whether a patient was offered EMR access by either their HCP or their insurer (coded as "Yes", "No", or "Don't Know")
2. EMR Use: How often in the past 12 months a patient accessed their EMR (divided into 5 categories ranging from "None" to "\>= 10")

## Files
### Background Information
Background files are in the folder `bkgrnd`.  
Detailed reports on the methodology used for each HINTS survey, including the specific data fields, collection, and cleaning process were downloaded along with the raw data. These files are:  
- `HINTS5-Cycle2-MethodologyReport.pdf`: Methodology for Cycle 2 (2018)
- `HINTS5-Cycle3-MethodologyReport.pdf`: Methodology for Cycle 3 (2019)
- `HINTS-WebPilotResultsReport.pdf`: Summary of pilot experiment performed during Cycle 3 allowing certain respondent to submit their surveys via the Web, instead of by mail. Points out key differences between Web and mail-in respondents.
- `HINTS5-Cycle4-MethodologyReport.pdf`: Methodology for Cycle 4 (2020)

Additionally, codebooks for each survey are available which describe each question, its possible responses, and the response counts. These files were used to preliminarily screen fields for relevance, completeness, and to assess whether they were present for all three cycles. These files are:
- `HINTS 5 Cycle 2 Public Codebook.pdf`: Codebook for Cycle 2 (2018)
- `HINTS 5 Cycle 3 Public Codebook.pdf`: Codebook for Cycle 3 (2019)
- `HINTS 5 Cycle 4 Public Codebook.pdf`: Codebook for Cycle 4 (2020)

### Data
All data files used for the analysis are in the `data` folder.

Raw data from HINTS in STATA format:
- `hints5_cycle2_public.dta`: Cycle 2 (2018)
- `hints5_cycle3_public.dta`: Cycle 3 (2019)
- `hints5_cycle3_public.dta`: Cycle 4 (2020)

The variables selected from each survey for preliminary analysis, along with a description of each, are in `HINTS-variables.ods`. The tab `final_set` lists the variables that were available for all three cycles and so were incorporated into the analysis. The other tabs compare variables between survey cycles.  

Lists of the variables to be retained from the raw dataset for each cycle (used to simplify reading in the data) are in the following CSV files:
- `C2_vars.csv`: Variables for Cycle 2 (2018)
- `C3_vars.csv`: Variables for Cycle 3 (2019)
- `C4_vars.csv`: Variables for Cycle 4 (2020)

The preliminary merged dataset, containing data for all three cycles with uniform field names and encodings, with missing-data codes retained, is `HINTS_merge_with_missing.csv.gz`.

These data were analyzed to determine each field's univariate relationship to EMR availability and use, and determinations were made about how to handle missing data. Then, the data were divided 70%/30% into training and test datasets for machine learning modeling. These final datasets are `HINTS_train.csv.gz` and `HINTS_test.csv.gz`

### Exploration and Analysis

Data selection, cleaning, and exploratory data analysis using graphical and univariate statistical methods were performed with Python. The code, description, and output (textual and graphical) of this process is contained in the Jupyter notebook `DSND_Final_Explore.ipynb`.  

Variables with significant univariate associations with EMR availability and use were placed into multivariate logistic regression machine learning models. The most parsimonious models which maximized precision and recall were determined, and the odds ratio of each final variable for predicting either EMR availability or use was calculated and examined. This process, including narrative, Python code, and textual description/analysis, is contained in `DSND_Final_Analysis.ipynb`.  

A technical blog post describing the data, methodology, results, and conclusions can be found [on Medium](https://ens59327.medium.com/patient-electronic-medical-record-adoption-in-the-us-f961c9b95a96).

## Dependencies
See `requirements.txt`.

## Summary of Results
Univariate EDA reduced the dataset from 59 variables to 38 with a significant relationship to EMR availability and 41 to EMR use. After handling missing and erroneous data codes, the initial set of 11942 records was reduced to 7818, which were split into 5490 (70%) for the training set and 2328 (30%) in the test set.  

Expanding the categorical variables with dummy encodings resulted in 106 features for predicting EMR access and 109 for predicting EMR use. These features were used to developed multivariate logistic regression models. Feature reduction was accomplished with recursive feature elimination (RFE).  Initially, automatic variable selection (scikit-learn's [RFECV](https://scikit-learn.org/stable/modules/generated/sklearn.feature_selection.RFECV.html#sklearn.feature_selection.RFECV)) was used, followed by RFE with manually set feature thresholds to further reduce the feature space. Model tuning with grid search was attempted as well. Overfitting was asssessed by looking for large discrepancies in precision, recall, and accuracy between the training and test set predictions. Adequacy of fit of the models was determined by precision and recall of predictions on the test set.

### EMR Availability
The original EMR access outcome variable (`offeredaccesseither`) was multi-class with three categories ("yes", "no", "don't know"). Fitting an RFECV automatically-pruned logistic regression model for this outcome resulted in 64 features, with relatively poor precision (0.622) and recall (0.555). Tuning the model parameters with a grid search resulted in a model with 81 features, and minimal improvements in precision (0.630) and recall (0.567).  

To potentially improve fit, the sparse "don't know" category was merged with "no", creating a binary outcome with better class balance. The binary model with default parameters and RFECV pruning had 52 features, and and improved precision (0.702) and recall (0.696). Grid-search tuning yielded a 93-feature model with essentially the same precision (0.706) and recall (0.700). Due to the increase in features and minimal improvement in fit, the latter model was discarded, and the former was taken as the starting point for manual feature reduction with RFE. From the initial 52-feature model, reduced models selected by RFE with between 5 and 50 parameters (in 5-parameter increments) were fit. Best precision (0.705) and recall (0.699) were obtained at 30 features, and this model was selected as the final one.

Demographic features associated with increased predicted likelihood of EMR access were female gender, higher education, and moderately older age (65-74). There was also an effect of year (survey cycle), and the COVID-19 pandemic. Demographics associated with decreased likelihood of access were being single, low income, non-Hispanic Asian race, linguistic isolation, and residing in the East South Central census division.

Health-related features associated with increased predicted EMR access were having a regular HCP whose care was more highly rated, seeing that HCP two or more times yearly, having health insurance, and a history of cancer. Features in this category associated with less likelihood of access were public insurance (Medicare and/or Medicaid) alone, very heavy drinking, and low confidence in one's ability to manage health affairs.

E-device and internet-related factors predicting increased likelihood of access were internet use, increased use of e-devices and the web for health-related purposes, visiting social-networking sites, and degree of use of public internet access. No variable in this category predicted reduced access.

To strength of each feature was examined via its odds ratio. The features most strongly associated with increased access were having health insurance and highly rating one's care. Those most strongly associated with decreased likelihood of access were low confidence in one's ability to manage health affairs, and non-Hispanic Asian race.

Finally, features with the largest difference in prevalence between those with predicted likelihoood of \>= 80% and \<= 20% of being offered EMR access were assessed. Of the top ten, lower income was more prevalent in the low-likelihood group, while female gender, higher education level, having a regular HCP, higher HCP rating, and several variables relating to increased internet/device use and use for health-related purposes were more prevalent in the high-likelihood group.

### EMR Use
Similar to the access variable, the EMR use variable (`accessonlinerecord`) was multiclass, except with five categories, relating to increased use frequency. The initial RFECV + logistic regression model for this outcome had 82 features and fair precision (0.607) with poor recall (0.471). Tuning the parameters with grid search gave a 99-parameter model with decreased precision (0.551) and slightly improved, but still poor, recall (0.517).

As with the other outcome, the culprit was believed to be the less frequent categories causing imbalanced predictions. A binary outcome variable comparing "none" to "any" use of the EMR was therefore created. The default RFECV + logistic regression model for this outcome showed greatly improved precision (0.741) and recall (0.724) using 62 features. Grid-search tuning decreased the feature space to 54 but with lower precision (0.709) and recall (0.689).  Based on this, the 62-feature model was used as the starting point for manual RFE tuning. Models containg from 5 to 60 features (again in 5-feature increments) were created and their scores compared. Optimal precision (0.742) and recall (0.724) where obtained with both 45 and 50 features; 45 was selected as more parsimonious.

Demographic features associated with increased predicted likelihood of EMR use were female gender, any education \>= high school,  and living in the Pacific (CA, OR, WA, AK, HI) census division. There was again an effect of year (survey cycle), and the COVID-19 pandemic. Demographics associated with decreased likelihood of use were being separated, lower income, Hispanic race, linguistic isolation, residing in the Middle Atlantic, East South Central, or Mountain census divisions, and residing in a more rural area.

Health-related features associated with increased predicted EMR use were having a regular HCP, see an HCP >= 3 times yearly, any care rating (aside from none/don't visit HCP), having health insurance, a history of diabetes or cancer, and being a current non-smoker. Features in this category associated with less likelihood of use were heavy drinking, severe psychological distress (highest PHQ-4 score) and low confidence in one's ability to manage health affairs.

E-device and internet-related factors predicting increased likelihood of EMR use were internet use, broadband access, having multiple e-devices, increased use of the devices and the web for health-related purposes, and degree of use of public internet access. As previously, no variable in this category predicted reduced use.

Examining odds ratios, an "excellent" HCP-care rating and >= college degree were most strongly associated with increased likelihood of EMR use.  Those most strongly associated with decreased likelihood of access were residing in the East South Central census division and in a linguistically-isolated area.  

Finally, prevalence differences between those with predicted likelihood of \>= 80% and \<= 20% of having used an EMR were assessed. The top ten were all more common in those predicted more likely to use an EMR. With the exception of education >= college degree and having a regular HCP, all were related to increased internet and e-device access/use for both general and health-related purposes.

#### History
Created August 2, 2021

#### License  
[Licensed](license.md) under the [MIT License](https://spdx.org/licenses/MIT.html). Yours to do with what you will.
