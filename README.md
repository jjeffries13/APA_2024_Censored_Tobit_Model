# Beyond the Threshold: Harnessing Censored Data in Student Feedback Systems
**Jay Jeffries, Justin Andersson, Om Joshi, James Bovaird**

**Teacher Analysis by Students (TABS) inventory -- shortened form**

The response options fo rhte five-point scale includes Strongly disagree (1) to Strongly agree (5).

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

Naive Model Syntax:
```
PROC MIXED DATA=LONG_DAT NOCLPRINT;
	CLASS ID CID;
	MODEL SCORE=TREATC1 TREATC2 TIMEC1 TIMEC2 TIMEC3 TRXT2 TRXT3 TRXT4 / SOLUTION 
		DDFM=BW;
	RANDOM INT / SUBJECT=ID(CID);
	RANDOM INT / SUBJECT=CID;
RUN;
```

Multilevel Tobit Model Syntax:
```
PROC NLMIXED DATA=LONG_DAT XTOL=1E-12 METHOD=GAUSS QPOINTS=100;
 PARMS beta0=38.9918 /* Intercept */
 beta1=1.5503 /* TREATC1 */
 beta2=-0.3143 /* TREATC2 */
 beta3=0.6011 /* TIMEC1 */
 beta4=0.8252 /* TIMEC2 */
 beta5=0.9052 /* TIMEC3 */
 beta6=-0.6072 /* TRXT2 */
 beta7=-0.3113 /* TRXT3 */
 beta8=-0.3479 /* TRXT4 */
 sigma2_u=33.3459 /* Random ID(CID) intercept */
 sigma3_u=13.2812 /* Random CID intercept */
 sigma2=12.3180; /* Random residual */
 BOUNDS sigma2_u sigma3_u sigma2 >=0;
 pi=CONSTANT('pi');

mu=beta0 + b_0j2 + b_0j3 + beta1*TREATC1 + beta2*TREATC2 + beta3*TIMEC1 + beta4*TIMEC2 + beta5*TIMEC3 + beta6*TRXT2 + beta7*TRXT3 + beta8*TRXT4;

if SCORE < 45 then 
 ll=(1 / (sqrt(2*pi*sigma2))) * exp(-(SCORE-mu)**2 / (2*sigma2) );
 if SCORE >=45 then
 ll=1 - probnorm((SCORE - mu) / sqrt(sigma2) );
 L=log(ll);

/* Likelihood function stays same, only score bounds change */
MODEL SCORE ~ GENERAL(L);
 RANDOM b_0j3 ~ NORMAL(0, sigma3_u) SUBJECT=CID;
&nbsp;&nbsp;&nbsp;&nbsp;RANDOM b_0j2 ~ NORMAL(0, sigma2_u) SUBJECT=ID(CID);
```

Code adapted from: <https://stats.oarc.ucla.edu/sas/faq/how-do-i-run-a-random-effect-tobit-model-using-nlmixed/>
