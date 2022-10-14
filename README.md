# Predictive Analytics in R
<script
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"
  type="text/javascript">
</script>

<div align="center">
<!-- <h1 align="center" style="display: block; font-size: 2.5em; font-weight: bold; margin-block-start: 1em; margin-block-end: 1em;">
 <br><strong>Predictive Analytics in R</strong></br>
</h1>  
 -->
<img src="https://cdn3.iconfinder.com/data/icons/product-management-color/64/metric-analytic-analysis-statistic-data-presentation-512.png" alt="Hatch logo" width="250" role="img">


</div>

<div align="center">

| | |
| --- | --- |
| CI/CD | [![CI - Test](https://github.com/pypa/hatch/actions/workflows/test.yml/badge.svg)](https://github.com/pypa/hatch/actions/workflows/test.yml) [![CD - Build Hatch](https://github.com/pypa/hatch/actions/workflows/build-hatch.yml/badge.svg)](https://github.com/pypa/hatch/actions/workflows/build-hatch.yml) [![CD - Build Hatchling](https://github.com/pypa/hatch/actions/workflows/build-hatchling.yml/badge.svg)](https://github.com/pypa/hatch/actions/workflows/build-hatchling.yml) |
| Docs | [![Docs - Release](https://github.com/pypa/hatch/actions/workflows/docs-release.yml/badge.svg)](https://github.com/pypa/hatch/actions/workflows/docs-release.yml) [![Docs - Dev](https://github.com/pypa/hatch/actions/workflows/docs-dev.yml/badge.svg)](https://github.com/pypa/hatch/actions/workflows/docs-dev.yml) |
| Package | [![PyPI - Version](https://img.shields.io/pypi/v/hatch.svg?logo=pypi&label=PyPI&logoColor=gold)](https://pypi.org/project/hatch/) [![PyPI - Downloads](https://img.shields.io/pypi/dm/hatchling.svg?color=blue&label=Downloads&logo=pypi&logoColor=gold)](https://pypi.org/project/hatch/) [![PyPI - Python Version](https://img.shields.io/pypi/pyversions/hatch.svg?logo=python&label=Python&logoColor=gold)](https://pypi.org/project/hatch/) |
| Meta | [![Hatch project](https://img.shields.io/badge/%F0%9F%A5%9A-Hatch-4051b5.svg)](https://github.com/pypa/hatch) [![code style - black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black) [![types - Mypy](https://img.shields.io/badge/types-Mypy-blue.svg)](https://github.com/python/mypy) [![imports - isort](https://img.shields.io/badge/imports-isort-ef8336.svg)](https://github.com/pycqa/isort) [![License - MIT](https://img.shields.io/badge/license-MIT-9400d3.svg)](https://spdx.org/licenses/) [![GitHub Sponsors](https://img.shields.io/github/sponsors/ofek?logo=GitHub%20Sponsors&style=social)](https://github.com/sponsors/ofek) |

 </div>
 
-----

## Introduction[![]()](#introduction) ⚡️ ⚙️ 📖 💡 

**SOA PA** is a guide to predictive analytics in R. 

---

## Table of contents[![](./docs/img/pin.svg)](#table-of-contents)
1. [Generalized Linear Models](#generalized-linear-models)  
  1.1 [Ordinary Least Squares](#ordinary-least-squares)  
2. [Decision Trees](#decision-trees)  
  2.1 [Regression Trees](#regression-trees)  
3. [Cluster and Principal Component Analyses](#cluster-analysis)  

---

### Project Definition Framework
> Appendix A demonstrates concepts and methodologies for effective problem definition and project management.

#### Define
After reading the sample project, formulate a concise project statement that will guide the analyses you will eventually perform.

Project Statement: Determine factors that will relate consumer decisions to buy term life insurance and for those who do, the amount purchased. Selected factors should be able to be used by the marketing department to better target their sales efforts.

Note: This statement implies that the models selected will be used to provide marketing insights, not predictions. That is, the use won’t be “input characteristics of a potential customer to see if we want to contact that person”, but rather, “target advertising to these groups of people because they are more likely to purchase term insurance”.

> Example: “Using internal data available on agents and policyholders, we can identify particular agents or attributes of policyholders that lead to them being more likely to lapse within the first two years of the policy. This will allow us to pinpoint at the time of quote which customers will be more expensive and can drive better underwriting decisions leading to an overall decrease in the costs associated with early lapse. We aim to reduce the duration 1 lapse rate from 22% to 14% and increase the pricing IRR from 10% to 12%.”

#### Evaluate and Prioritize
Examine the feasibility of the proposed solutions – is it possible and what are the constraints?  
*	Implementation constraints (data, resources, experience vs. timelines) 
*	Regulatory acceptance 
*	Consumer acceptance 
*	Potential for gaming
*	Cost/effort

**Implementation Constraints (Data, Resources, Experience vs. Timelines)**  
The following are potential concerns regarding the data available for us:  
1.	Partial Data: If the data that you have has been used before for other modeling tasks and you have been in contact with the team responsible, you can be relatively assured of the quality of the data. If the data has not been used for this purpose before, you cannot be certain of the quality of the data and may be forced to rely upon the most recent period or two.
2.	Old Data: If data exists across multiple legacy databases, any exercises that required modeling all datasets would require a significant data matching exercise and coordination between different parts of the business.
3.	Restricted Data: Personal data may be restricted if information may not be used in analysis
4.	Unverifiable Data:  Cannot be certain of quality of data and may be forced to rely upon data for most recent periods only.

**Other Considerations:**
*	 **Regulatory Acceptance:** Although not involved directly, the regulator is an important stakeholder in the analytics decision-making process. The business should consider how the regulator will react (either in-light of explicit guidance or from a principal’s perspective) to the predictive analytics solution being proposed.
*	 **Consumer Acceptance:** Customers may be directly involved or exposed to changes brought about by the predictive analytics project, either through direct communication or a change in the way they interact with existing products or services.
*	 **Potential for Gaming:** In particular, does the solution treat individuals with certain behaviors or characteristics more favorably than others? Is it possible for someone to change their behavior to achieve a better rate or outcome?
* 	**Required Cost and Effort:** If the cost and effort are too high given the expected return, both in the context of implementation, maintenance, and initial project itself, then it isn’t worth doing. Effort and cost determine whether you will undertake a project and will constrain the way you implement. This is one reason it is important to understand the effort and cost up front.

#### Prepare
Assess key ingredients missing from current analytical setup which need to be readied prior to project execution:  
*	 Data: Determine when and how can data (signed-off during the prioritize stage) be accessed, additional technology needed. 
*	 Internal Communication: Perform project setup tasks, engage staff, and form a dynamic business + project team.
*	 Stakeholder Interviews: Determine what information the stakeholders will need regarding progress of the project and when.
* 	SMEs: Determine whether external SMEs are needed to guide aspects of the analytics.



---

## 1. Generalized Linear Models[![](./docs/img/pin.svg)](#generalized-linear-models)

> 💡 In current version, the AREG engine handles multithreading (_Local_) and multiprocessing (_Public_) communication. 

### 1.1 Orindary Least Squares[![]()](#ordinary-least-squares)

**Model Definition**
> The standard (nongeneralized) linear model with p predictors is written as: 
> *	Y = b0 + b1x1 + … + bPxP + e


The method of estimating parameters β0 through βp that we will use for linear models is called ordinary least squares. The goal is to determine the parameter values that minimize the sum of squared errors (SSE), all called the sum of squared residuals (SSR):

> **SSE = SSR = ∑ (ŷi – yi)2, β0  + β1x1 + … +  βpxp** 

With an additional assumption that ε has a normal distribution, this is also the maximum likelihood estimator

**Data Preparation**
```{r}
AutoClaim <- read.csv("AutoClaim.csv")

# We will only use GENDER, AGE, BLUEBOOK, and CLM_AMT for demonstration.
AutoClaim_sub <- subset(AutoClaim, select = c("GENDER", "AGE", "BLUEBOOK", "CLM_AMT"))

# Create age bands and then remove age.
AutoClaim_sub$AGE_BAND <- cut(x = AutoClaim_sub$AGE, breaks = c(0, 25, 35, 45, 55, 65, 85))
AutoClaim_sub$AGE <- NULL

# Select only cases where CLM_AMT is positive.
AutoClaim_sub <- AutoClaim_sub[AutoClaim$CLM_AMT > 0, ]
head(AutoClaim_sub)
```

**Testing Assumptions for OLS**
1. Mean of Residuals is Zero
2. Constant Variance (Residual Plots)
3. Residuals have no autocorrelation
4. Residuals and Predictor are Uncorrelated (Hypothesis Test that Correlation is Zero)
5. Residuals have normal distribution (Q-Q Plot or Histogram)

```{r}
model.ols <- lm(CLM_AMT ~ GENDER + AGE_BAND + BLUEBOOK, data = AutoClaim_sub)

# 1. Check the mean of the residuals.
mean(model.ols$residuals)

# 2. Check the residuals for constant variance. Not all four plots relate to this assumption.
par(mfrow = c(2, 2))
plot(model.ols)

# 4. Check that the residuals and the predictor variables are uncorrelated.
cor.test(AutoClaim_sub$BLUEBOOK, model$residuals) # Run this for each your predictor variables

# 5. Check that the residuals have a normal distribution.
# One check is a Q-Q plot, which appears in the upper right corner of the plots made when checking for constant variance. Another option is to make a histogram of the residuals.

resid <- data.frame(model.ols$residuals)
ggplot(resid, aes(x = model.residuals)) +
  geom_histogram(position = "identity", col = "black")

```

**Problems with OLS Model**  

1. Not appropriate when:  
  -  The range of target variable values is positive. The normal distribution allows for negative values and hence the model may predict negative outcomes.
  -  The variance of the target depends on the mean. This violates the constant variance assumption.
  -  The target variable is binary. The restriction to 0 and 1 responses does not fit normal distribution.
2. Other Limitations:  
  -  Sensitive to outliers  
  -  By definition, they do not perform well with non-linear relations.  


### 1.2 Generalized Linear Models Specifications and Assumptions 
The generalized assumptions are:
* Given the predictor variable values, the target variables are independent
* Given the predictor variable values, the target variable's distribution is a member of the exponential family.
* Given the predictor variable values, the expected value of the target variables is:

  > $$\mu = {g^-1(\nu)}, {\nu = X\beta}$$ 
  > 
  > Where g is called the **link function**

---

## 2. Decision Trees[![](./docs/img/pin.svg)](#decision-trees)

> 💡 In current version, the AREG engine handles multithreading (_Local_) and multiprocessing (_Public_) communication. 



---

## Documentation

The [documentation]() is made with [Material for MkDocs](https://github.com/squidfunk/mkdocs-material) and is hosted by [GitHub Pages](https://docs.github.com/en/pages).

## License

SOA-PA is distributed under the terms of the [MIT](https://spdx.org/licenses/MIT.html) license.
