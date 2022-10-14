# Generalized Linear Models[![](./docs/img/pin.svg)](#generalized-linear-models)

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

If the conditional distribution of the target variable is normal (member of exponential) and the link function is simply $g(\mu) = \mu$, we have the ordinary regression model.
