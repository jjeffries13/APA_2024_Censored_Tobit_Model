# Beyond the Threshold: Harnessing Censored Data in Student Feedback Systems
**Jay Jeffries, Justin Andersson, Om Joshi, James Bovaird**

**Teacher Analysis by Students (TABS; shortened form)**

The five-point scale includes Strongly disagree (1), Somewhat disagree (2), Neither agree nor disagree (3), Somewhat agree (4), and Strongly agree (5). 

Each of the nine item begin with the stem: "*My teacher...*", followed by the item. Below are the nine items in the order they appeared on the survey:

1. Makes the purposes of each class session and learning activity clear
2. Clarifies material that needs explanation
3. Maintains an atmosphere that actively encourages learning
4. Inspires excitement or interest in the content of the course
5. Promotes mutual respect by relating to us students
6. Explains what is expected from each student
7. Makes clear how my performance is to be evaluated
8. Designs evaluation procedures consistent with course goals
9. Gives meaningful feedback on assignments

**SAS Code**

Naive estimates from a mixed-effect multilevel mondel informed the tobit model priors.

PROC NLMIXED DATA=LONG_DAT XTOL=1E-12 METHOD=GAUSS QPOINTS=100;
&nbsp;&nbsp;&nbsp;&nbsp;PARMS beta0=38.9918 /* Intercept */
&nbsp;&nbsp;&nbsp;&nbsp;beta1=1.5503 /* TREATC1 () */
&nbsp;&nbsp;&nbsp;&nbsp;beta2=-0.3143 /* TREATC2 () */
&nbsp;&nbsp;&nbsp;&nbsp;beta3=0.6011 /* TIMEC1 () */
&nbsp;&nbsp;&nbsp;&nbsp;beta4=0.8252 /* TIMEC2 () */
&nbsp;&nbsp;&nbsp;&nbsp;beta5=0.9052 /* TIMEC3 () */
&nbsp;&nbsp;&nbsp;&nbsp;beta6=-0.6072 /* TRXT2 () */
&nbsp;&nbsp;&nbsp;&nbsp;beta7=-0.3113 /* TRXT3 () */
&nbsp;&nbsp;&nbsp;&nbsp;beta8=-0.3479 /* TRXT4 () */
&nbsp;&nbsp;&nbsp;&nbsp;sigma2_u=33.3459 /* Random ID(CID) intercept */
&nbsp;&nbsp;&nbsp;&nbsp;sigma3_u=13.2812 /* Random CID intercept */
&nbsp;&nbsp;&nbsp;&nbsp;sigma2=12.3180; /* Random residual */
&nbsp;&nbsp;&nbsp;&nbsp;BOUNDS sigma2_u sigma3_u sigma2 >=0;
&nbsp;&nbsp;&nbsp;&nbsp;pi=CONSTANT('pi');
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;mu=beta0 + b_0j2 + b_0j3 + beta1*TREATC1 + beta2*TREATC2 + beta3*TIMEC1 + beta4*TIMEC2 + beta5*TIMEC3 + beta6*TRXT2 + beta7*TRXT3 + beta8*TRXT4;

if SCORE < 45 then 
&nbsp;&nbsp;&nbsp;&nbsp;ll=(1 / (sqrt(2*pi*sigma2))) * exp(-(SCORE-mu)**2 / (2*sigma2) );
&nbsp;&nbsp;&nbsp;&nbsp;if SCORE >=45 then
&nbsp;&nbsp;&nbsp;&nbsp;ll=1 - probnorm((SCORE - mu) / sqrt(sigma2) );
&nbsp;&nbsp;&nbsp;&nbsp;L=log(ll);

/* Likelihood function stays same, only score bounds change */
MODEL SCORE ~ GENERAL(L);
&nbsp;&nbsp;&nbsp;&nbsp;RANDOM b_0j3 ~ NORMAL(0, sigma3_u) SUBJECT=CID;
&nbsp;&nbsp;&nbsp;&nbsp;RANDOM b_0j2 ~ NORMAL(0, sigma2_u) SUBJECT=ID(CID);

Code adapted from: <https://stats.oarc.ucla.edu/sas/faq/how-do-i-run-a-random-effect-tobit-model-using-nlmixed/>
