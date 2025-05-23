# Simple example to use multiple imputation

This is the example used during the seminar to illustrate a simple case of MI. 

🔰 First we generate 3 variables and simulate a simple confounding mechanism.

```ruby
*-----------------------------------------------
* 1. Generate data
*-----------------------------------------------
clear
set seed 250520
set obs 1000

gen confounder = round(rnormal(60, 8))
gen exposure = rbinomial(1, invlogit(-2 + 0.04*confounder))
gen outcome = 2*exposure + 0.5*confounder + rnormal(0, 5)

gen y_full = outcome
gen c_full = confounder
gen x_full = exposure
```

🔰 We then generate missing data under MAR.

```ruby
*-----------------------------------------------
* 2. Create missing data in confounder 
*    -> MAR: missingness in confounder depends on exposure and outcome
*-----------------------------------------------
gen p_missing = invlogit(-1 + 1.5*exposure + 0.03*outcome) 
gen u = runiform()
replace confounder = . if u < p_missing
drop p_missing u

misstable summarize
```

🔰 We can check the predictors that predict missing values (although we already know :) )

```ruby
gen missing = 0 if confounder != . 
replace missing = 1 if confounder == . 
tab missing 

logit missing exposure outcome , or
```

🔰 We can now analyse the reference scenario (no missing), the complete cases (only observed values), and the MI analysis (imputed values).

```ruby
*-----------------------------------------------
* 3. ANALYSES
*-----------------------------------------------

* (a) Reference model: using full data (no missing; not possible with real data)
reg y_full x_full c_full

* (b) Complete case analysis
reg outcome exposure confounder if !missing(confounder)

* (c) Multiple imputation for missing confounder
mi set wide
mi register imputed confounder
mi register regular outcome exposure

* Impute missing age with linear regression
mi impute regress confounder exposure outcome, add(20) rseed(2505)

* MI analysis
mi estimate: regress outcome exposure confounder
```
