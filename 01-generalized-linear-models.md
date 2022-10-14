# Generalized Linear Models[![](./docs/img/pin.svg)](#generalized-linear-models)

## 1.1 Orindary Least Squares[![]()](#ordinary-least-squares)

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


## 1.2 Specifications and Assumptions 
The generalized assumptions are:
* Given the predictor variable values, the target variables are independent
* Given the predictor variable values, the target variable's distribution is a member of the exponential family.
* Given the predictor variable values, the expected value of the target variables is:

  > $$\mu = {g^-1(\nu)}, {\nu = X\beta}$$ 
  > 
  > Where g is called the **link function**

If the conditional distribution of the target variable is normal (member of exponential) and the link function is simply $g(\mu) = \mu$, we have the ordinary regression model.

## 1.3 Link Function $g$

**Commonly Used Link Functions**

1. **Identity:** $g(n) = n, g^-1(n) exp(=) n$
2. **Log:** $g(n) = log(n), g^-1(n) = exp(n)$
3. **Reciprocal:** $g(n) = 1/n, g^-1(n) = 1/m$
4. **Logit:** $g(n) = log(n/(1-n)), g^-1(n)=e^n/(1+e^n)$


<table class="wikitable" style="background:white;">
<caption>Common distributions with typical uses and canonical link functions
</caption>
<tbody><tr>
<th>Distribution</th>
<th>Support of distribution</th>
<th>Typical uses</th>
<th>Link name</th>
<th>Link function, <span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=g(\mu )\,\!}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold">X</mi>
        </mrow>
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold-italic">β<!-- β --></mi>
        </mrow>
        <mo>=</mo>
        <mi>g</mi>
        <mo stretchy="false">(</mo>
        <mi>μ<!-- μ --></mi>
        <mo stretchy="false">)</mo>
        <mspace width="thinmathspace"></mspace>
        <mspace width="negativethinmathspace"></mspace>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=g(\mu )\,\!}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/5e2ebd12256b9e1b8dcdfdd4bd625f37df639ded" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.838ex; margin-right: -0.387ex; width:11.366ex; height:2.843ex;" alt="{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=g(\mu )\,\!}"></span></th>
<th>Mean function
</th></tr>
<tr>
<td><a href="/wiki/Normal_distribution" title="Normal distribution">Normal</a>
</td>
<td>real: <span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle (-\infty ,+\infty )}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mo stretchy="false">(</mo>
        <mo>−<!-- − --></mo>
        <mi mathvariant="normal">∞<!-- ∞ --></mi>
        <mo>,</mo>
        <mo>+</mo>
        <mi mathvariant="normal">∞<!-- ∞ --></mi>
        <mo stretchy="false">)</mo>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle (-\infty ,+\infty )}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/e577bfa9ed1c0f83ed643206abae3cd2f234cf9c" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.838ex; width:11.107ex; height:2.843ex;" alt="(-\infty ,+\infty )"></span></td>
<td>Linear-response data</td>
<td>Identity
</td>
<td><span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=\mu \,\!}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold">X</mi>
        </mrow>
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold-italic">β<!-- β --></mi>
        </mrow>
        <mo>=</mo>
        <mi>μ<!-- μ --></mi>
        <mspace width="thinmathspace"></mspace>
        <mspace width="negativethinmathspace"></mspace>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=\mu \,\!}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/63238c06f9c1927aee60b40fec3adccd419cf32a" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.838ex; margin-right: -0.387ex; width:8.441ex; height:2.676ex;" alt="\mathbf {X} {\boldsymbol {\beta }}=\mu \,\!"></span></td>
<td><span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle \mu =\mathbf {X} {\boldsymbol {\beta }}\,\!}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mi>μ<!-- μ --></mi>
        <mo>=</mo>
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold">X</mi>
        </mrow>
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold-italic">β<!-- β --></mi>
        </mrow>
        <mspace width="thinmathspace"></mspace>
        <mspace width="negativethinmathspace"></mspace>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle \mu =\mathbf {X} {\boldsymbol {\beta }}\,\!}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/12c514082234f52d09595635789f474de0279b7d" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.838ex; margin-right: -0.387ex; width:8.441ex; height:2.676ex;" alt="\mu =\mathbf {X} {\boldsymbol {\beta }}\,\!"></span>
</td></tr>
<tr>
<td><a href="/wiki/Exponential_distribution" title="Exponential distribution">Exponential</a>
</td>
<td rowspan="2">real: <span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle (0,+\infty )}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mo stretchy="false">(</mo>
        <mn>0</mn>
        <mo>,</mo>
        <mo>+</mo>
        <mi mathvariant="normal">∞<!-- ∞ --></mi>
        <mo stretchy="false">)</mo>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle (0,+\infty )}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/de77e40eb7e2582eef8a5a1da1bc027b7d9a8d6e" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.838ex; width:8.138ex; height:2.843ex;" alt="(0,+\infty )"></span></td>
<td rowspan="2">Exponential-response data, scale parameters
</td>
<td rowspan="2"><a href="/wiki/Multiplicative_inverse" title="Multiplicative inverse">Negative inverse</a>
</td>
<td rowspan="2"><span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=-\mu ^{-1}\,\!}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold">X</mi>
        </mrow>
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold-italic">β<!-- β --></mi>
        </mrow>
        <mo>=</mo>
        <mo>−<!-- − --></mo>
        <msup>
          <mi>μ<!-- μ --></mi>
          <mrow class="MJX-TeXAtom-ORD">
            <mo>−<!-- − --></mo>
            <mn>1</mn>
          </mrow>
        </msup>
        <mspace width="thinmathspace"></mspace>
        <mspace width="negativethinmathspace"></mspace>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=-\mu ^{-1}\,\!}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/f6532ae0a7d9f63020f9a3e4175c391fb1130f99" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.838ex; margin-right: -0.387ex; width:12.582ex; height:3.176ex;" alt="\mathbf {X} {\boldsymbol {\beta }}=-\mu ^{-1}\,\!"></span>
</td>
<td rowspan="2"><span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle \mu =-(\mathbf {X} {\boldsymbol {\beta }})^{-1}\,\!}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mi>μ<!-- μ --></mi>
        <mo>=</mo>
        <mo>−<!-- − --></mo>
        <mo stretchy="false">(</mo>
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold">X</mi>
        </mrow>
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold-italic">β<!-- β --></mi>
        </mrow>
        <msup>
          <mo stretchy="false">)</mo>
          <mrow class="MJX-TeXAtom-ORD">
            <mo>−<!-- − --></mo>
            <mn>1</mn>
          </mrow>
        </msup>
        <mspace width="thinmathspace"></mspace>
        <mspace width="negativethinmathspace"></mspace>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle \mu =-(\mathbf {X} {\boldsymbol {\beta }})^{-1}\,\!}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/11209fa27eda9b964da5691b83fd3652d59ddcc0" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.838ex; margin-right: -0.387ex; width:14.391ex; height:3.176ex;" alt="\mu =-(\mathbf {X} {\boldsymbol {\beta }})^{-1}\,\!"></span>
</td></tr>
<tr>
<td><a href="/wiki/Gamma_distribution" title="Gamma distribution">Gamma</a>
</td></tr>
<tr>
<td><a href="/wiki/Inverse_Gaussian_distribution" title="Inverse Gaussian distribution">Inverse <br>Gaussian</a>
</td>
<td>real: <span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle (0,+\infty )}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mo stretchy="false">(</mo>
        <mn>0</mn>
        <mo>,</mo>
        <mo>+</mo>
        <mi mathvariant="normal">∞<!-- ∞ --></mi>
        <mo stretchy="false">)</mo>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle (0,+\infty )}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/de77e40eb7e2582eef8a5a1da1bc027b7d9a8d6e" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.838ex; width:8.138ex; height:2.843ex;" alt="(0,+\infty )"></span></td>
<td></td>
<td>Inverse <br>squared</td>
<td><span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=\mu ^{-2}\,\!}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold">X</mi>
        </mrow>
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold-italic">β<!-- β --></mi>
        </mrow>
        <mo>=</mo>
        <msup>
          <mi>μ<!-- μ --></mi>
          <mrow class="MJX-TeXAtom-ORD">
            <mo>−<!-- − --></mo>
            <mn>2</mn>
          </mrow>
        </msup>
        <mspace width="thinmathspace"></mspace>
        <mspace width="negativethinmathspace"></mspace>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=\mu ^{-2}\,\!}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/0a3b87590326202b24e85ce5762989fd34bff8c2" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.838ex; margin-right: -0.387ex; width:10.774ex; height:3.176ex;" alt="{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=\mu ^{-2}\,\!}"></span></td>
<td><span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle \mu =(\mathbf {X} {\boldsymbol {\beta }})^{-1/2}\,\!}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mi>μ<!-- μ --></mi>
        <mo>=</mo>
        <mo stretchy="false">(</mo>
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold">X</mi>
        </mrow>
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold-italic">β<!-- β --></mi>
        </mrow>
        <msup>
          <mo stretchy="false">)</mo>
          <mrow class="MJX-TeXAtom-ORD">
            <mo>−<!-- − --></mo>
            <mn>1</mn>
            <mrow class="MJX-TeXAtom-ORD">
              <mo>/</mo>
            </mrow>
            <mn>2</mn>
          </mrow>
        </msup>
        <mspace width="thinmathspace"></mspace>
        <mspace width="negativethinmathspace"></mspace>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle \mu =(\mathbf {X} {\boldsymbol {\beta }})^{-1/2}\,\!}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/9f2b2781a377e3d9ed78c1b1e026fda1e8895402" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.838ex; margin-right: -0.387ex; width:14.227ex; height:3.343ex;" alt="{\displaystyle \mu =(\mathbf {X} {\boldsymbol {\beta }})^{-1/2}\,\!}"></span>
</td></tr>
<tr>
<td><a href="/wiki/Poisson_distribution" title="Poisson distribution">Poisson</a>
</td>
<td>integer: <span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle 0,1,2,\ldots }">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mn>0</mn>
        <mo>,</mo>
        <mn>1</mn>
        <mo>,</mo>
        <mn>2</mn>
        <mo>,</mo>
        <mo>…<!-- … --></mo>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle 0,1,2,\ldots }</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/b1da8ed7e74b31b6314f23f122a1198c104fcaad" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.671ex; width:9.312ex; height:2.509ex;" alt="0,1,2,\ldots "></span></td>
<td>count of occurrences in fixed amount of time/space</td>
<td><a href="/wiki/Natural_logarithm" title="Natural logarithm">Log</a></td>
<td><span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=\ln(\mu )\,\!}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold">X</mi>
        </mrow>
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold-italic">β<!-- β --></mi>
        </mrow>
        <mo>=</mo>
        <mi>ln</mi>
        <mo>⁡<!-- ⁡ --></mo>
        <mo stretchy="false">(</mo>
        <mi>μ<!-- μ --></mi>
        <mo stretchy="false">)</mo>
        <mspace width="thinmathspace"></mspace>
        <mspace width="negativethinmathspace"></mspace>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=\ln(\mu )\,\!}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/245ed014e9dd7f9624171201d1a4daecb1c20997" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.838ex; margin-right: -0.387ex; width:12.189ex; height:2.843ex;" alt="{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=\ln(\mu )\,\!}"></span></td>
<td><span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle \mu =\exp(\mathbf {X} {\boldsymbol {\beta }})\,\!}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mi>μ<!-- μ --></mi>
        <mo>=</mo>
        <mi>exp</mi>
        <mo>⁡<!-- ⁡ --></mo>
        <mo stretchy="false">(</mo>
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold">X</mi>
        </mrow>
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold-italic">β<!-- β --></mi>
        </mrow>
        <mo stretchy="false">)</mo>
        <mspace width="thinmathspace"></mspace>
        <mspace width="negativethinmathspace"></mspace>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle \mu =\exp(\mathbf {X} {\boldsymbol {\beta }})\,\!}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/7fac36b3451b711d49417813988a6e8bb4db5719" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.838ex; margin-right: -0.387ex; width:13.803ex; height:2.843ex;" alt="{\displaystyle \mu =\exp(\mathbf {X} {\boldsymbol {\beta }})\,\!}"></span>
</td></tr>
<tr>
<td><a href="/wiki/Bernoulli_distribution" title="Bernoulli distribution">Bernoulli</a>
</td>
<td>integer: <span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle \{0,1\}}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mo fence="false" stretchy="false">{</mo>
        <mn>0</mn>
        <mo>,</mo>
        <mn>1</mn>
        <mo fence="false" stretchy="false">}</mo>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle \{0,1\}}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/28de5781698336d21c9c560fb1cbb3fb406923eb" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.838ex; width:5.684ex; height:2.843ex;" alt="\{0,1\}"></span></td>
<td>outcome of single yes/no occurrence
</td>
<td rowspan="5"><a href="/wiki/Logit" title="Logit">Logit</a>
</td>
<td><span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=\ln \left({\frac {\mu }{1-\mu }}\right)\,\!}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold">X</mi>
        </mrow>
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold-italic">β<!-- β --></mi>
        </mrow>
        <mo>=</mo>
        <mi>ln</mi>
        <mo>⁡<!-- ⁡ --></mo>
        <mrow>
          <mo>(</mo>
          <mrow class="MJX-TeXAtom-ORD">
            <mfrac>
              <mi>μ<!-- μ --></mi>
              <mrow>
                <mn>1</mn>
                <mo>−<!-- − --></mo>
                <mi>μ<!-- μ --></mi>
              </mrow>
            </mfrac>
          </mrow>
          <mo>)</mo>
        </mrow>
        <mspace width="thinmathspace"></mspace>
        <mspace width="negativethinmathspace"></mspace>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=\ln \left({\frac {\mu }{1-\mu }}\right)\,\!}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/8756b6c8f78882b05820c4058a861002462ef4b4" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -2.505ex; margin-right: -0.387ex; width:18.64ex; height:6.176ex;" alt="{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=\ln \left({\frac {\mu }{1-\mu }}\right)\,\!}"></span>
</td>
<td rowspan="5"><span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle \mu ={\frac {\exp(\mathbf {X} {\boldsymbol {\beta }})}{1+\exp(\mathbf {X} {\boldsymbol {\beta }})}}={\frac {1}{1+\exp(-\mathbf {X} {\boldsymbol {\beta }})}}\,\!}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mi>μ<!-- μ --></mi>
        <mo>=</mo>
        <mrow class="MJX-TeXAtom-ORD">
          <mfrac>
            <mrow>
              <mi>exp</mi>
              <mo>⁡<!-- ⁡ --></mo>
              <mo stretchy="false">(</mo>
              <mrow class="MJX-TeXAtom-ORD">
                <mi mathvariant="bold">X</mi>
              </mrow>
              <mrow class="MJX-TeXAtom-ORD">
                <mi mathvariant="bold-italic">β<!-- β --></mi>
              </mrow>
              <mo stretchy="false">)</mo>
            </mrow>
            <mrow>
              <mn>1</mn>
              <mo>+</mo>
              <mi>exp</mi>
              <mo>⁡<!-- ⁡ --></mo>
              <mo stretchy="false">(</mo>
              <mrow class="MJX-TeXAtom-ORD">
                <mi mathvariant="bold">X</mi>
              </mrow>
              <mrow class="MJX-TeXAtom-ORD">
                <mi mathvariant="bold-italic">β<!-- β --></mi>
              </mrow>
              <mo stretchy="false">)</mo>
            </mrow>
          </mfrac>
        </mrow>
        <mo>=</mo>
        <mrow class="MJX-TeXAtom-ORD">
          <mfrac>
            <mn>1</mn>
            <mrow>
              <mn>1</mn>
              <mo>+</mo>
              <mi>exp</mi>
              <mo>⁡<!-- ⁡ --></mo>
              <mo stretchy="false">(</mo>
              <mo>−<!-- − --></mo>
              <mrow class="MJX-TeXAtom-ORD">
                <mi mathvariant="bold">X</mi>
              </mrow>
              <mrow class="MJX-TeXAtom-ORD">
                <mi mathvariant="bold-italic">β<!-- β --></mi>
              </mrow>
              <mo stretchy="false">)</mo>
            </mrow>
          </mfrac>
        </mrow>
        <mspace width="thinmathspace"></mspace>
        <mspace width="negativethinmathspace"></mspace>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle \mu ={\frac {\exp(\mathbf {X} {\boldsymbol {\beta }})}{1+\exp(\mathbf {X} {\boldsymbol {\beta }})}}={\frac {1}{1+\exp(-\mathbf {X} {\boldsymbol {\beta }})}}\,\!}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/b739082e7ee418a2163685f976c75b4906910158" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -2.671ex; margin-right: -0.387ex; width:37.302ex; height:6.509ex;" alt="{\displaystyle \mu ={\frac {\exp(\mathbf {X} {\boldsymbol {\beta }})}{1+\exp(\mathbf {X} {\boldsymbol {\beta }})}}={\frac {1}{1+\exp(-\mathbf {X} {\boldsymbol {\beta }})}}\,\!}"></span>
</td></tr>
<tr>
<td><a href="/wiki/Binomial_distribution" title="Binomial distribution">Binomial</a>
</td>
<td>integer: <span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle 0,1,\ldots ,N}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mn>0</mn>
        <mo>,</mo>
        <mn>1</mn>
        <mo>,</mo>
        <mo>…<!-- … --></mo>
        <mo>,</mo>
        <mi>N</mi>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle 0,1,\ldots ,N}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/4f0dabd0eecff746a5377991354a67ea28a4e684" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.671ex; width:10.601ex; height:2.509ex;" alt="0,1,\ldots ,N"></span></td>
<td>count of # of "yes" occurrences out of N yes/no occurrences
</td>
<td><span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=\ln \left({\frac {\mu }{n-\mu }}\right)\,\!}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold">X</mi>
        </mrow>
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold-italic">β<!-- β --></mi>
        </mrow>
        <mo>=</mo>
        <mi>ln</mi>
        <mo>⁡<!-- ⁡ --></mo>
        <mrow>
          <mo>(</mo>
          <mrow class="MJX-TeXAtom-ORD">
            <mfrac>
              <mi>μ<!-- μ --></mi>
              <mrow>
                <mi>n</mi>
                <mo>−<!-- − --></mo>
                <mi>μ<!-- μ --></mi>
              </mrow>
            </mfrac>
          </mrow>
          <mo>)</mo>
        </mrow>
        <mspace width="thinmathspace"></mspace>
        <mspace width="negativethinmathspace"></mspace>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=\ln \left({\frac {\mu }{n-\mu }}\right)\,\!}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/ecbce4c90689853e5656461e1165f5473d276a44" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -2.505ex; margin-right: -0.387ex; width:18.873ex; height:6.176ex;" alt="{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=\ln \left({\frac {\mu }{n-\mu }}\right)\,\!}"></span>
</td></tr>
<tr>
<td rowspan="2"><a href="/wiki/Categorical_distribution" title="Categorical distribution">Categorical</a>
</td>
<td>integer: <span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle [0,K)}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mo stretchy="false">[</mo>
        <mn>0</mn>
        <mo>,</mo>
        <mi>K</mi>
        <mo stretchy="false">)</mo>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle [0,K)}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/aa074207d3bea2e879410172ce89ba2435d37d11" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.838ex; width:5.814ex; height:2.843ex;" alt="[0,K)"></span></td>
<td rowspan="2">outcome of single K-way occurrence
</td>
<td rowspan="3"><span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=\ln \left({\frac {\mu }{1-\mu }}\right)\,\!}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold">X</mi>
        </mrow>
        <mrow class="MJX-TeXAtom-ORD">
          <mi mathvariant="bold-italic">β<!-- β --></mi>
        </mrow>
        <mo>=</mo>
        <mi>ln</mi>
        <mo>⁡<!-- ⁡ --></mo>
        <mrow>
          <mo>(</mo>
          <mrow class="MJX-TeXAtom-ORD">
            <mfrac>
              <mi>μ<!-- μ --></mi>
              <mrow>
                <mn>1</mn>
                <mo>−<!-- − --></mo>
                <mi>μ<!-- μ --></mi>
              </mrow>
            </mfrac>
          </mrow>
          <mo>)</mo>
        </mrow>
        <mspace width="thinmathspace"></mspace>
        <mspace width="negativethinmathspace"></mspace>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=\ln \left({\frac {\mu }{1-\mu }}\right)\,\!}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/8756b6c8f78882b05820c4058a861002462ef4b4" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -2.505ex; margin-right: -0.387ex; width:18.64ex; height:6.176ex;" alt="{\displaystyle \mathbf {X} {\boldsymbol {\beta }}=\ln \left({\frac {\mu }{1-\mu }}\right)\,\!}"></span>
</td></tr>
<tr>
<td>K-vector of integer: <span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle [0,1]}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mo stretchy="false">[</mo>
        <mn>0</mn>
        <mo>,</mo>
        <mn>1</mn>
        <mo stretchy="false">]</mo>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle [0,1]}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/738f7d23bb2d9642bab520020873cccbef49768d" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.838ex; width:4.653ex; height:2.843ex;" alt="[0,1]"></span>, where exactly one element in the vector has the value 1
</td></tr>
<tr>
<td><a href="/wiki/Multinomial_distribution" title="Multinomial distribution">Multinomial</a>
</td>
<td><i>K</i>-vector of integer: <span class="mwe-math-element"><span class="mwe-math-mathml-inline mwe-math-mathml-a11y" style="display: none;"><math xmlns="http://www.w3.org/1998/Math/MathML" alttext="{\displaystyle [0,N]}">
  <semantics>
    <mrow class="MJX-TeXAtom-ORD">
      <mstyle displaystyle="true" scriptlevel="0">
        <mo stretchy="false">[</mo>
        <mn>0</mn>
        <mo>,</mo>
        <mi>N</mi>
        <mo stretchy="false">]</mo>
      </mstyle>
    </mrow>
    <annotation encoding="application/x-tex">{\displaystyle [0,N]}</annotation>
  </semantics>
</math></span><img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/703d57dca548a7f9d927247c2a27b67666aebdd5" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.838ex; width:5.554ex; height:2.843ex;" alt="[0,N]"></span></td>
<td>count of occurrences of different types (1 .. <i>K</i>) out of <i>N</i> total <i>K</i>-way occurrences
</td></tr></tbody></table>

### Binary Data
When Response Variable is Binary (0 or 1), as in classification, use logistic regression.

**Classification**
There are many possible classiﬁcation techniques, or classiﬁers, that one might use to predict a qualitative response

Three of the most widely-used classiﬁers: logistic regression, linear discriminant analysis, and K-nearest neighbors.

Rather than modeling this response Y directly, logistic regression models the probability that Y belongs to a particular category.

$p(X) = e^{β0+β1X}/(1 + e^{β0+β1X})$

```{r}
#install.packages("ISLR")/
library(ISLR)
train=(data.all$Year<2005)

# Load Data
data.all <- ISLR::Smarket

# View Data
names(data.all)
dim(data.all)
summary(data.all)
pairs(data.all)
cor(data.all)
plot(data.all$Volume)
contrasts(data.all$Direction)

# Partition Data for Training and Testing  
data.train <- data.all[(data.all$Year<2005),]
data.test <- data.all[!(data.all$Year<2005),]

# Logistic Regression: Lag 1-5 and Volume
glm.fit <- glm(Direction ~ Lag1+Lag2+Lag3+Lag4+Lag5+Volume, data=data.all, family=binomial, subset=(data.all$Year<2005))
glm.prob <- predict(glm.fit, data.test, type="response")
glm.pred <- rep("Down", 252)
glm.pred[glm.prob >.5] <- "Up"
table(glm.pred, data.test$Direction)
mean(glm.pred==data.test$Direction)

# Logistic Regression: Subset of predictors (Lag1 and Lag2)
glm.fit <- glm(Direction~Lag1+Lag2, data=data.train, family=binomial)
glm.prob <- predict(glm.fit, data.test, type="response")
glm.pred <- rep("Down", 252)
glm.pred[glm.prob >.5] <- "Up"
table(glm.pred, data.test$Direction)
mean(glm.pred==data.test$Direction)

# LDA 
library(MASS)
lda.fit <- lda(Direction ~ Lag1+Lag2, data=data.all, subset=(data.all$Year<2005))
lda.fit
plot(lda.fit)
lda.pred <- predict(lda.fit, newdata=data.test)
table(lda.pred$class, data.test$Direction)
mean(lda.pred$class==data.test$Direction)

# QDA 
library(MASS)
qda.fit <- qda(Direction ~ Lag1+Lag2, data=data.all, subset=(data.all$Year<2005))
qda.fit
qda.pred <- predict(qda.fit, newdata=data.test)
table(qda.pred$class, data.test$Direction)
mean(qda.pred$class==data.test$Direction)

# Interestingly, the QDA predictions are accurate almost 60 % of the time,
# even though the 2005 data was not used to ﬁt the model. This level of 
# accuracy is quite impressive for stock market data, which is known to be quite
# hard to model accurately. This suggests that the quadratic form assumed
# by QDA may capture the true relationship more accurately than the linear
# forms assumed by LDA and logistic regression. 

# k-Nearest Neighbors (knn)
set.seed(1)
library(class)

data.train.X <- data.all[(data.all$Year<2005), c("Lag1", "Lag2")]
data.test.X <- data.all[!(data.all$Year<2005), c("Lag1", "Lag2")]
data.train.Y <- data.all[(data.all$Year<2005), c("Direction")]

# k = 1
knn.pred <- knn(data.train.X, data.test.X, data.train.Y, k=1)
table(knn.pred, data.test$Direction)
mean(knn.pred==data.test$Direction)

# k = 3
knn.pred <- knn(data.train.X, data.test.X, data.train.Y, k=3)
table(knn.pred, data.test$Direction)
mean(knn.pred==data.test$Direction)

# The results have improved slightly. But increasing K further turns out
# to provide no further improvements. It appears that for this data, QDA
# provides the best results of the methods that we have examined so far.

# Because the KNN classiﬁer predicts the class of a given test observation by
# identifying the observations that are nearest to it, the scale of the variables
# matters. Any variables that are on a large scale will have a much larger
# eﬀect on the distance between the observations, and hence on the KNN
# classiﬁer, than variables that are on a small scale.
# 
# A good way to handle this problem is to standardize the data so that all 
# variables are given a mean of zero and a standard deviation of one. Then
# all variables will be on a comparable scale. The scale() function does just this. 
# In standardizing the data, we exclude column 86, because that is the
# qualitative Purchase variable.
```

### Count Data 
If data comes in the form of a whole number of occurances, such as claim counts, it may be best to use Poisson regression because it places positive probability on 0, 1, 2, ..., n. Here, the mean is a positive number, so a link function that forces positive numbers is the best. The `log` link function is commonly used.


```{r}
# This approach uses the dplyr package.
library(dplyr)

data.all <- read.csv("warpbreaks.csv")
data.all %>% 
  group_by(wool, tension) %>% 
  summarise(
    mean_breaks=mean(breaks), 
    var_breaks=var(breaks)
  )

# It is clear that means are related to the two variables. The Poisson distribution assuems the mean and variance are equal. Here, the variance is consistently higher than the mean --> Quasi-Poisson.


# Fits a Poisson GLM and prints the results
model.fit <-glm(breaks ~ wool+tension, data=data.all, family=poisson)
summary(model.fit)

# Call:
# glm(formula = breaks ~ wool + tension, family = poisson, data = warpbreaks)
# 
# Deviance Residuals: 
#     Min       1Q   Median       3Q      Max  
# -3.6871  -1.6503  -0.4269   1.1902   4.2616  
# 
# Coefficients:
#             Estimate Std. Error z value Pr(>|z|)    
# (Intercept)  3.17347    0.05567  57.002  < 2e-16 ***
# woolB       -0.20599    0.05157  -3.994 6.49e-05 ***
# tensionL     0.51849    0.06396   8.107 5.21e-16 ***
# tensionM     0.19717    0.06833   2.885  0.00391 ** 
# ---
# Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
# 
# (Dispersion parameter for poisson family taken to be 1)
# 
#     Null deviance: 297.37  on 53  degrees of freedom
# Residual deviance: 210.39  on 50  degrees of freedom
# AIC: 493.06
# 
# Number of Fisher Scoring iterations: 4


# Linear prediction: exp(3.17347 - 0.20599 + 0.51849 = 3.48597) = 32.654

data.test <- data.frame(
  wool=c("A", "A", "A", "B", "B", "B"),
  tension=c("L", "M", "H", "L", "M", "H")
  )

# Make predictions on test data
model.pred <- predict(model.fit, newdata=data.test, type="response")
model.pred <- cbind(data.test, model.pred)
model.pred

# Fit an Over-Dispersed Poisson (Quasi-Poisson) Model
model.fit.odp <-glm(breaks ~ wool + tension, data=data.all, family=quasipoisson)
summary(model.fit.odp)

model.pred.odp <- predict(model.fit.odp, newdata=new.data, type="response")
predictions <- cbind(model.pred, model.pred.odp)
predictions
```


