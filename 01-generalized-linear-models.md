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

### Continous Positive-Value Data
In insurance applications, it is common to measure quanities that are strictly positive. May be sufficient to a log transformation of the variable. However, it may be useful to use a regression model particularly suited to positive-valued data.

Three common examples are the gamma, lognormal, and inverse gaussian distributions. These would generally be used with a log link function to ensure positive predictions.

Another possibility is a normal distribution with a log link function. TThe predictions of the mean for new observations will always be positive.

**Tweedie Distribution**
Compound Poisson and Gamma Distribution. As an example, supplose a single drive has $N$ claims in a year where $N ~ Poisson(\lambda)$ and each claim $X_1,...,X_N$ has a gamma distribution with parameters $\alpha$ and $\beta$. The total claims are then $S = X_1 + ... + X_N$ and they have a Tweedie distribution. The mean and variance are the following:

$$\mu = \lambda*\alpha/\beta, \sigma^2 = \lambda\alpha(\alpha+1)/\beta^2$$

```{r}
library(tidyverse)
library(dplyr)
library(ggplot2)
library(gridExtra)

data.all <- read.csv("AutoClaim.csv")

data.all <- data.all %>% 
  dplyr::select(c('GENDER', 'AGE', 'BLUEBOOK', 'CLM_AMT')) %>% 
  filter(CLM_AMT > 0) %>%
  mutate(
    AGE_BAND = cut(x=AGE, breaks=c(0, 25, 35, 45, 55, 65, 85)),
    AGE = NULL
  )

comma_format <- function(x){
  x <- format(x, nsmall=0, scientific=FALSE, big.mark=",")
  return(x)
} 

p <- ggplot(data=data.all) 

p1 <- p + 
  geom_density(aes(x=CLM_AMT), fill='blue', alpha=0.2) + 
  geom_histogram(aes(x=CLM_AMT, ..density..), fill = "grey50", alpha = 0.5) +
  scale_x_continuous(labels=x_format) + 
  labs(x="Claim Amount ($)", y="Density", title="Claim Amount",
       subtitle="Distribution (Density Plot)", caption="Source: Auto Claim") 
p2 <- p + 
  geom_density(aes(x=CLM_AMT), fill='blue', alpha=0.2) + 
  geom_histogram(aes(x=CLM_AMT, ..density..), fill = "grey50", alpha = 0.5) +
  scale_x_log10(labels=x_format) + 
  labs(x="Log Claim Amount ($)", y="Density", title="Log Claim Amount",
       subtitle="Distribution (Density Plot)", caption="Source: Auto Claim") 
p3 <- p + 
  geom_boxplot(aes(x="Claim Amount", y=CLM_AMT))
  labs(x="Log Claim Amount ($)", y="Density", title="Log Claim Amount",
       subtitle="Distribution (Box Plot)", caption="Source: Auto Claim") 
p4 <- p + 
  geom_boxplot(aes(x="Claim Amount", y=CLM_AMT)) + 
  scale_y_log10(labels=x_format)
  labs(x="Log Claim Amount ($)", y="Box Plot", title="Log Claim Amount",
       subtitle="Distribution (Box Plot)", caption="Source: Auto Claim") 

grid.arrange(p1, p2, p3, p4, ncol=2)


data.all$logCLM_AMT <- log(data.all$CLM_AMT)

model1.fit <- lm(CLM_AMT~GENDER+AGE_BAND+BLUEBOOK, data=data.all)
model2.fit <- lm(logCLM_AMT~GENDER+AGE_BAND+BLUEBOOK, data=data.all)
model3.fit <- glm(CLM_AMT ~ GENDER + AGE_BAND + BLUEBOOK, data=data.all, family=Gamma(link="log"))

model1.pred <- predict(model1.fit)
model2.pred <- predict(model2.fit)
model3.pred <- predict(model3.fit, type = "response")

# Which model performs the best?
model1.sse = sum((data.all$CLM_AMT - model1.pred)^2)
model2.sse = sum((data.all$CLM_AMT - exp(model2.pred))^2)
model3.sse = sum((data.all$CLM_AMT - model3.pred)^2)
data.frame(
  model=c("Linear Model", "Linear Model (Log Target)", "GLM Gamma with log link"),
  sse = c(model1.sse, model2.sse, model3.sse)
)

```


```{r}
library(tidyverse)
library(dplyr)
library(ggplot2)
library(gridExtra)

data.all <- read.csv("AutoClaim.csv")

data.all <- data.all %>% 
  dplyr::select(c('GENDER', 'AGE', 'BLUEBOOK', 'CLM_FREQ5')) %>% 
  mutate(
    AGE_BAND = cut(x=AGE, breaks=c(0, 25, 35, 45, 55, 65, 85)),
    AGE = NULL
  )


# Normal linear regression model with Identity Link
model1.fit <- glm(CLM_FREQ5 ~ ., data=data.all)
model1.pred <- predict(model1.fit, type='response')
model1.sse <- sum((data.all$CLM_FREQ5 - model1.pred)^2)

# Poisson regression model with Identity Link
model2.fit <- glm(CLM_FREQ5 ~ ., data=data.all, family=poisson(link="log"))
model2.pred <- predict(model2.fit, type='response')
model2.sse <- sum((data.all$CLM_FREQ5 - model2.pred)^2)

# Summary
data.frame(
  model=c("Normal Linear Model w/ Identity Link", "Poisson Regression with Log Link"),
  sse=c(model1.sse, model2.sse)
)

# Performance is similar. Although the normal model 
# has the potential to predict negative values, it 
# turns out that the minimum prediction is 0.48 claims.
min(model1.pred)

```

### Feature Generation

**Variable:** Represents the predictors in our models. Variables are more closely associated with raw data, which makes up part of the datasets before any transformation takes place.

**Feature:** Derivations from the original variables or final inputs into the model. 

```{r}
library(tidyverse)

data.mortality <- read.csv("soa_mortality_data.csv")

data.mortality <- data.mortality %>% 
  filter(exposure_face > 0) %>% 
  mutate(
    actual_q = actual_face / exposure_face
  )


nrow(data.mortality)
summary(data.mortality)

data.mortality %>% 
  select_if(is.numeric) %>%
  gather(feature,value) %>%
  ggplot(aes(value)) +
  geom_histogram() +
  facet_wrap(vars(feature))

# Log transform example
data.mortality$duration_log <- log(data.mortality$duration)

# Normalization
data.mortality$issage_norm <- scale(data.mortality$issage)

# Binned versions of variables example
data.mortality$issage_bin10 <- cut(data.mortality$issage, 10)
data.mortality$issage_bin20 <- cut(data.mortality$issage, 20, labels = FALSE) # Note the difference between using labels or not

vars.bin <- c("sex", "smoker", "prodcat", "region", "distchan", "uwkey", "uwtype", "resind_ind")
dummies <- dummyVars(survived ~ ., data = etitanic)

data.mortality <- data.mortality %>% 
  select(sex, smoker, prodcat, region, distchan, uwkey, uwtype, resind_ind)) %>% 
  mutate(sapply(., as.character))


library(caret)

# List the variables we want to binarize
vars.bin <- c("sex", "smoker", "prodcat", "region", "distchan", "uwkey", "uwtype", "resind_ind")



```


### GLM Variable Selection

**Forward Stepwise Selection:**

1. Start with no predictors in the model;
2. Evaluate all p models which use only one predictor and choose the one with the best performance (Highest $R^2$ or Lowest $RSS$);
3. Repeat the process when adding one additional predictor, and continue until there is a model with one predictor, a model with two predictors, a model with three predictors, and so forth until there are p models;
4. Select the single best model which has the best $AIC$, $BIC$, or $Adjusted R^2$

**Backward Stepwise Selection:**

1. Start with a model that contains all predictors;
2. Create a model which removes all predictors;
3. Choose the best model which removes all-but-one predictor;
4. Choose the best model which removes all-but-two predictors;
5. Continue until there are p models;
6. Select the single best model which has the best $AIC$, $BIC$, or $Adjusted R^2$

**Both Forward & Backward Selection:** 

A hybrid approach is to consider using both forward and backward selection. This is done by creating two lists of variables at each step, one from forward and backward selection. `stepAIC()` 

| Metric | Formula | Interpretation
----------------------------------------------------
Residual Mean Sq Error	| $sqrt(\sigma(y_hat-y_i)^2)$ | Lower = Better
Mean Absolute Error	| $sqrt(\sigma(y_hat-y_i)^2)$ | Lower = Better
R-Squared | . Higher = Better
AIC | |  | Lower = Better
BIC | | | Lower = Better
Pearson's Chi Sq | | , Lower = Better| 

| Metric  | Description        |   |   |
|---------|--------------------|---|---|
| RMSE    | Lower = Better     | Predicts to mean  |
| MAE     | Lower = Better     | Predicts to median  |
| $R^2$   | Higher = Better    | % of variation in the response that is explained by the model  |   |
| AIC     | Lower = Better     | Log Likelihood with penality for complexity  |
| BIC     | Lower = Better     | Log Likelihood with penality for complexity and sample size   |
| Pearson | **Lower = Better** | Used when target is count |


```{r}
get_rmse <- function(y, y_hat){
  sqrt(mean((y - y_hat)^2))
}

get_mae <- function(y, y_hat){
  sqrt(mean(abs(y - y_hat)))
}

# Is there a pattern in the residuals? If there is, this means that the model is missing key information.
plot(model, which = 1)

# The normal QQ shows how well the quantiles of the predictions fit a theoretical normal distribution. 
plot(model, which = 2)

```

```{r}
library(MASS)
stepAIC() 

glm <- glm(formula, data, family)
AIC(glm)
```

**Elastic Net/Lasso/Ridge Advantages**  

1. All benefits from GLMS
2. Automatic variable selection for Lasso; smaller coefficients for Ridge
3. Better predictive power than GLM

### Elastic Net/Lasso/Ridge

1. All cons of GLMs

```{r}
# Ridge and Lasso
x <- model.matrix(Salary~., Hitters)[,-1] 
y <- df %>% 
  select(target) %>%
  unlist() %>%
  as.numeric()

#alpha = 0 --> ridge
#alpha = 1 --> lasso

grid <- 10^seq(10, -2, length=100)
model.ridge <- glmnet(x, y, alpha=0, lambda=grid, standardize=TRUE)

coef(ridge_mod)
coef(ridge_mod)[,50] #

sqrt(sum(coef(ridge_mod)[-1,50]^2)) # Calculate l2 norm
predict(model.ridge, s=50, type='coefficients')

index <- sample(c(TRUE, FALSE), nrow(data.all), replace=TRUE, prob=c(0.7, 0.3))

data.train <- data.all[index,]
data.test <- data.all[!index,]

grid = 10^seq(10, -2, length = 100)
x.train <- model.matrix(target~., data.train)[,-1]
x.test <- model.matrix(target~., data.test)[,-1]
y.train <- data.train %>% select(target) %>% unlist() %>% as.numeric()
y.train <- data.test %>% select(target) %>% unlist() %>% as.numeric()

model.fit = glmnet(x.train, y.train, alpha=0, lambda = grid, thresh = 1e-12)
model.pred <- predict(model.fit, s=50, type="coefficients")
model.pred <- predict(model.fit, s=4, newx=x.test, )
mean((model.pred - y.test)^2)
## [1] 139858.6

# Test set MSE
model.pred = predict(model.fit, s=1e10, newx=x.test)
mean((model.pred - y.test)^2)
## [1] 224692.1

model.pred = predict(model.fit, s=0, x=x.train, y=y.train, newx=x.test, exact=T)
mean((ridge_pred - y_test)^2)
## [1] 175051.7


cv.out = cv.glmnet(x_train, y_train, alpha = 0) # Fit ridge regression model on training data
plot(cv.out) # Draw plot of training MSE as a function of lambda
bestlam = cv.out$lambda.min  # Select lamda that minimizes training MSE
bestlam

ridge_pred = predict(ridge_mod, s = bestlam, newx = x_test) # Use best lambda to predict test data
mean((ridge_pred - y_test)^2) # Calculate test MSE

out = glmnet(x, y, alpha = 0) # Fit ridge regression model on full dataset
predict(out, type="coefficients", s=bestlam)[1:20,] # Display coefficients using lambda chosen by CV



lasso_mod = glmnet(x_train, y_train, alpha = 1, lambda = grid)  # Fit lasso model on training data
plot(lasso_mod)                                                 # Draw plot of coefficients

set.seed(1)
cv.out = cv.glmnet(x_train, y_train, alpha = 1) # Fit lasso model on training data
plot(cv.out) # Draw plot of training MSE as a function of lambda
bestlam = cv.out$lambda.min # Select lamda that minimizes training MSE
lasso_pred = predict(lasso_mod, s = bestlam, newx = x_test) # Use best lambda to predict test data
mean((lasso_pred - y_test)^2) # Calculate test MSE

# However, the lasso has a substantial advantage over ridge regression in that the resulting coefficient estimates are sparse.
out = glmnet(x, y, alpha = 1, lambda = grid) # Fit lasso model on full dataset
lasso_coef = predict(out, type = "coefficients", s = bestlam)[1:20,] # Display coefficients using lambda chosen by CV
lasso_coef

lasso_coef[lasso_coef!=0] # Display only non-zero coefficients
```


# Decision Trees

Decision trees can be used for either classification or regression problems. The model structure is a series of yes/no questions. Depending on how each observation answers these questions, a prediction is made.

**The below example shows how a single tree can predict health claims:**

* For non-smokers, the predicted annual claims are 8,434. This represents 80% of the observations
* For smokers with a bmi of less than 30, the predicted annual claims are 21,000. 10% of patients fall into this bucket.
* For smokers with a bmi of more than 30, the prediction is 42,000. This bucket accounts for 11% of patients.

**Step 2: Continue doing this until a stopping criterion is reached. For example, the minimum number of observations is 5 or less.**
```{r}
library(rpart)

tree <- rpart(formula = charges ~  ., data = health_insurance,
              control = rpart.control(cp = 0.003))
rpart.plot(tree, type = 3)
```

**Step 3: Apply cost complexity pruning to simplify the tree**


**Step 4: Use cross-validation to select the best alpha**
```{r}
tree <- rpart(formula = charges ~  ., data = health_insurance,
              control = rpart.control(cp = 0.0001))
cost <- tree$cptable %>% 
  as_tibble() %>% 
  select(nsplit, CP, xerror) 
  
## # A tibble: 6 × 3
##   nsplit      CP xerror
##    <dbl>   <dbl>  <dbl>
## 1      0 0.620    1.00 
## 2      1 0.144    0.382
## 3      2 0.0637   0.238
## 4      3 0.00967  0.175
## 5      4 0.00784  0.169
## 6      5 0.00712  0.163 
```

# Prune using opitmal cp
```{r}
tree$cptable %>% 
  as_tibble() %>% 
  select(nsplit, CP, xerror, `rel error`) %>% 
  head()
```
```{r}
pruned_tree <- prune(tree, cp = tree$cptable[which.min(tree$cptable[, "xerror"]), "CP"])
```


```{r}
library(caret)
set.seed(42)
index <- createDataPartition(y = health_insurance$charges, 
                             p = 0.8, list = F)
train <- health_insurance %>% slice(index)
test <- health_insurance %>% slice(-index)

simple_tree <- rpart(formula = charges ~  ., 
              data = train,
              control = rpart.control(cp = 0.0001, 
                                      minbucket = 200,
                                      maxdepth = 10))
rpart.plot(simple_tree, type = 3)
```


## Random Forest

```{r}
rf_data <- health_insurance %>% 
  sample_frac(0.2) %>% 
  mutate(sex = ifelse(sex == "male", 1, 0),
         smoker = ifelse(smoker == "yes", 1, 0),
         region_ne = ifelse(region == "northeast", 1,0),
         region_nw = ifelse(region == "northwest", 1,0),
         region_se = ifelse(region == "southeast", 1,0),
         region_sw = ifelse(region == "southwest", 1,0)) %>% 
  select(-region)
rf_data %>% glimpse(50)

## Rows: 268
## Columns: 10
## $ age       <dbl> 42, 44, 31, 36, 64, 28, 45, 27…
## $ sex       <dbl> 1, 1, 1, 1, 1, 0, 1, 1, 0, 1, …
## $ bmi       <dbl> 26.125, 39.520, 27.645, 34.430…
## $ children  <dbl> 2, 0, 2, 2, 0, 3, 0, 0, 0, 0, …
## $ smoker    <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, …
## $ charges   <dbl> 7729.646, 6948.701, 5031.270, …
## $ region_ne <dbl> 1, 0, 1, 0, 1, 0, 0, 0, 0, 1, …
## $ region_nw <dbl> 0, 1, 0, 0, 0, 1, 1, 0, 1, 0, …
## $ region_se <dbl> 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, …
## $ region_sw <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, …
```

```{r}
library(caret)
set.seed(42)
index <- createDataPartition(y = rf_data$charges, p = 0.8, list = F)
train <- rf_data %>% slice(index)
test <- rf_data %>% slice(-index)

rf <- randomForest(charges ~ ., data = train, ntree = 400)
plot(rf)

varImpPlot(x = rf)

# smoker, bmi, and age are the most important predictors of charges
```

```
pred <- predict(rf, test)
get_rmsle <- function(y, y_hat){
  sqrt(mean((log(y) - log(y_hat))^2))
}

get_rmsle(test$charges, pred)
## [1] 0.5252518
```

```{r}

get_rmsle(test$charges, mean(train$charges))
## [1] 1.118947

```
**Partial Dependency Plot: This attempts to measure the change in the predicted value by taking the average $y_hat$ after removing the effects of all other predictors.**
```{r}
library(pdp)

bmi <- pdp::partial(rf, pred.var = "bmi",  grid.resolution = 15) %>% 
  autoplot() + theme_bw()
age <- pdp::partial(rf, pred.var = "age",  grid.resolution = 15) %>% 
  autoplot() + theme_bw()
ggarrange(bmi, age)

```

## Gradiant Boosting Model
```{r}
library(gbm)
gbm <- gbm(charges ~ ., data = train,
           # Boosting Parameters
           n.trees = 100,
           shrinkage = 0.1,
           # Tree Parameters
           interaction.depth = 2,
           n.minobsinnode = 50)
           
pred <- predict(gbm, test, n.trees = 100)
get_rmsle(test$charges, pred)
get_rmsle(test$charges, mean(train$charges))
```

```{r}
set.seed(42)
#Take only 250 records 
#Uncomment this when completing this exercise
data <- health_insurance %>% sample_n(250) 

index <- createDataPartition(
  y = data$charges, p = 0.8, list = F) %>% 
  as.numeric()
train <-  health_insurance %>% slice(index)
test <- health_insurance %>% slice(-index)

control <- trainControl(
  method='boot', 
  number=2, 
  p = 0.2)

tunegrid <- expand.grid(.mtry=c(1,3,5))
rf <- train(charges ~ .,
            data = train,
            method='rf', 
            tuneGrid=tunegrid, 
            trControl=control)

pred_train <- predict(rf, train)
pred_test <- predict(rf, test)

get_rmse <- function(y, y_hat){
  sqrt(mean((y - y_hat)^2))
}

get_rmse(pred_train, train$charges)
get_rmse(pred_test, test$charges)
```

### Boosting with Caret
```{r}
library(caret)
set.seed(42)
index <- createDataPartition(y = health_insurance$charges, 
                             p = 0.8, list = F)
#To make this run faster, only take 50% sample
df <- health_insurance %>% sample_frac(0.50) 
train <- df %>% slice(index) 
test <- df %>% sample_frac(0.05)%>% slice(-index)

tunegrid <- expand.grid(
    interaction.depth = c(1,5, 10),
    n.trees = c(50, 100, 200, 300, 400), 
    shrinkage = c(0.5, 0.1, 0.0001),
    n.minobsinnode = c(5, 30, 100)
    )
nrow(tunegrid)

control <- trainControl(
  method='repeatedcv', 
  number=5, 
  p = 0.8)

gbm <- train(charges ~ .,
            data = train,
            method='gbm', 
            tuneGrid=tunegrid, 
            trControl=control,
            #Show detailed output
            verbose = FALSE
            )

results <- gbm$results %>% arrange(RMSE)
top_result <- results %>% slice(1)%>% mutate(param_rank = 1)
tenth_result <- results %>% slice(10)%>% mutate(param_rank = 10)
twenty_seventh_result <- results %>% slice(135)%>% mutate(param_rank = 135)

rbind(top_result, tenth_result, twenty_seventh_result) %>% 
  select(param_rank, 1:5)
  
 pdp::partial(gbm, pred.var = "bmi", grid.resolution = 15, plot = T)
 pdp::partial(gbm, pred.var = "bmi", grid.resolution = 20, plot = T, ice = T, alpha = 0.1, palette = "viridis")
 
```

## Principle Component Analysis
```{r}

pca = prcomp(USArrests, scale=TRUE)
names(pca)

#The center and scale components correspond to the means and standard deviations of the variables that were used for scaling prior to implementing PCA:
pca$center
pca$scale

# The rotation matrix provides the principal component loadings; each column of pr.out$rotation contains the corresponding principal component loading vector:
pca$rotation

# This is to be expected because there are in general min(n − 1, p) informative principal components in a data set with n observations and p variables.
head(pca$x)

biplot(pca, scale=0)

pca$sdev
pca_var=pca$sdev^2
pve=pca_var/sum(pca_var)
plot(pve, xlab="Principal Component", ylab="Proportion of Variance Explained", ylim=c(0,1),type='b')

plot(cumsum(pve), xlab="Principal Component", ylab="Cumulative Proportion of Variance Explained", ylim=c(0,1),type='b')


```
