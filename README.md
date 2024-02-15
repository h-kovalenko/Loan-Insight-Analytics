# Loan-Insight-Analytics

## Introduction
The LoanInsight 2021 project focuses on analyzing the **Loan Level Dataset** for the year 2021, a comprehensive collection of mortgage and lending industry data. This dataset offers detailed information on individual loans, including borrower demographics, loan terms, and financial situations. The goal of the analysis is to uncover meaningful insights into lending practices, with a particular emphasis on disparities in approval rates and denials, including racial and ethnic considerations.

* [Data Analysis Question & Answers](./questions_and_answers.md)

## Data
The Loan Level Dataset for 2021 comprises 2.6 million rows and 89 columns, offering a holistic view of factors influencing loan approval decisions. Key variables include applicant demographics, loan terms, property details, and financial information. Notable numeric statistics include an average loan amount of $304,753.97 and an average interest rate of 3.08%.

The dataset can be retrieved from the [link](https://ffiec.cfpb.gov/data-publication/dynamic-national-loan-level-dataset/2021)
## Analysis and Results
The analysis involved data cleaning using Google Cloud Tools like **DataPrep** and exploration through **BigQuery** and **Looker**. Key insights include:

First Bank and Mortgage Research Center display a significantly high loan approval rate for applicants with Debt-to-Income Ratios >60%, surpassing the 16% baseline.

Black individuals within the same income group experience a higher denial rate for home purchases.

Certain banks exhibit higher denial rates for Black or African American applicants compared to white applicants.

The mortgage denial rate for Black or African Americans in red states is 1.5% higher than in blue states, while denial rates for Asians and Whites remain stable.

## Conclusion
**Loan Insight Analytics** provides valuable insights into mortgage lending practices, utilizing a rich dataset sourced from the Home Mortgage Disclosure Act (HMDA). The findings shed light on various aspects of the mortgage lending landscape, emphasizing disparities in lending practices and the need for further examination and potential policy interventions.

