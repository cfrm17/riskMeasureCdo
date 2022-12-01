# Risk Measures for FNM and CDO2&3 Trades

The purpose of the submitted model is to calculate the risk measures for FirstNofM trade (FNM) trades and CDO2&3 trades. They are bucketed credit spread sensitivities for FNM trades; bucketed credit spread sensitivities for CDO2&3 trades; default sensitivities and correlation sensitivities for CDO2&3 trades. 

The submitted new model is a great improvement of the existing risk measures in several aspects:

•	The credit spread sensitivity is switched to a bucketed one (CSPDH). In the model, parallel shift credit spread sensitivity has been used for many years. However, recent development in the market, especially the popularity of longer term trades, makes this measure inaccurate.
 
•	A new method of computing the credit spread sensitivity, namely semi-analytic Monte Carlo (MC) sensitivity, is adopted in the submitted model [1,3]. It applied to FNM trades and CDO2&3 trades. In the computation of risk, now it is known that a default event that passes across the maturity in the perturbed scenario has a large contribution. Using a usual bump/revaluation approach, it is very hard to model such an event if the perturbation is tiny in the case such as computing CSPDH. In the new model, a deterministic part is added to directly address this part of contribution to credit spread sensitivities; hence the computational efficiency and precision are greatly improved.  

•	The CSPDH of CDO2&3 trades is the most complicated model in the Oscar/Fritz credit library.  In the model, a CDO2&3 trade is flattened to a risk equivalent vanilla CDO trade (RE-CDO) and then valued within the base correlation framework. When the credit spread of an obligor is perturbed, the attachment point of the RE-CDO trade is changed, which can be found through a re-flattening process. Using the analytical sensitivity approach as discussed in Refs. [1, 4, 5], the computation of CSPDH for a CDO2&3 trade is decomposed into two parts. One is CSPDH for RE-CDO, which is the same as that of a bespoke trade. The other part is CSPDH attributed to change of the attachment of RE-CDO. This part is calculated via a Jacobian of the attachment point to the credit spread change, in which the analytical MC sensitivity is employed. The first part of CSPDH can be tested straightforwardly by switching off re-flattening option and benchmarking against the approved bespoke CDO pricing template.  We focus on the second part of CSPDH for CDO2&3 trades. 

•	The default sensitivity model of CDO2&3 trades in the model is basically an internal bump/revaluation approach. Compared to the ones for bespoke trades [6], the only difference is that the attachment point of RE-CDO is re-flattened in the perturbed scenario for CDO2&3 trades. Note that in the old risk measure, there are no re-flattening of the RE-CDO and re-mapping of the base correlations in the perturbed scenario. The new method enables us to capture the underlying risks more accurately.

•	The correlation sensitivities of CDO2&3 trades is the same as the ones with bespoke CDO trades, because a perturbation of the base correlation does not have any effect on the attachment of RE-CDO. 

The implementation of the MC risk model was first verified by a full bump and revaluation approach using the model and an independent test model. The CSPDH of FNM trades is also tested against the semi-closed form valuation model, in the case of the homogeneous loss given defaults (LGD). The CSPDH of CDO2&3 trades was tested against an independent test model and a full bump/revaluation approach using the model.  The default sensitivity and correlation sensitivity for CDO2&3 trades were verified by replicating them using the bespoke CDO template.

The MC convergence of CSPDH was also assessed. It is verified that 500,000 paths and 1,000,000 paths are adequate for CDO2&3 trades and FNM trades, respectively.  

The MC risk model is based on the assumption that the payoff is a continuous function of default time. Therefore, certain types of trades are excluded from using this method. For example, the FNM trade without accrual can not use this method. A full bump and revaluation approach is used instead.  Furthermore, in order to monitor such uncertainties, a monthly benchmark test for all outstanding FNM trades will be conducted by GRMMR London.  

Define a collateral pool of a CDO trade as a set of N reference names, , in which each reference name is described by a credit spread curve  , a recovery rate  , a default time  , and a notional amount  .  Within the current credit derivatives framework a reduced form of default probability of a reference name has been implemented and well maintained.  For the   reference name in the collateral pool, the hazard rate curve  is defined such that the default probability between s and s+ds  . With this definition the default probability functions built upon the hazard rate are

(1)	 = Survival probability,
(2)	 = Default probability,
(3)	  = Default probability density function.

For each credit spread curve, there is a standard term structure defined as  ={1w, 1m, 3m, 6m, 1y, 2y, 3y, 4y, 5y, 7y, 10y, 20y}).  The hazard rate is assumed to a stepwise linear function. For the   obligor, its   term is defined as  .

A copula is a mathematical function that combines marginal probability into a joint distribution. For N uniform random variables, , the joint distribution function is defined as

(4)	 

which is called the copula function.

The normal copula function is a multi-variate cumulative normal distribution with correlation matrix . Applying the normal copula function to the modeling of correlated default events of a collateral asset pool, the uniform random variables are mapped to the default probabilities with standard normal distribution. The normal copula function, or the cumulative joint default probability for the collateral pool with n assets, can be expressed as

(5)	 

where   is the standard cumulative normal distribution function .

As shown in Eq.(5), the normal copula function is actually an N-dimensional integral, which is hard to calculate directly if N is large. Instead of directly computing a complicated N-dimensional joint default probability, a MC simulation has been proved to be a powerful way of solving the correlated defaults problem. 

In each MC scenario in order to generate correlated defaults times using normal copula, a series of random variables   are first generated from an N-dimensional normal distribution with correlation matrix .  Within the one factor framework,   is generated a combination of two random variables.  For the obligor in the collateral pool, we assume that its asset process follows 

(6)		 ,								

where ,  are independent standard normal random variables. The latent variable  is shared by all reference names in the collateral pool in the  MC scenario. The correlation   is a flat constant correlation for all obligors.

For the   MC scenario, the default time of the obligor is denoted as  , which can be obtained by 

(7)	 

Where   is the normal cumulative probability function. 

The generated correlated default times, associated with loss given default of each obligor, can then be used to price structrued credit derivatives.  Assume the payoff function, either the protection leg or the premium leg, is denoted as . The expected value can be calculated via MC simulation by

(8)	 

Assume the trade has a maturity  . In each MC scenario only the default time before maturity will trigger a payoff in Eq.(8). Therefore we are only interested in the default times before T.  There exist a value for each obligor

(9)	 

Therefore for each obligor we could divide   into regions: smaller than and equal to  or larger than . Conditional on the latent variable, the probability of the default time smaller than maturity can be expressed as:

(9)	 

When a risk factor of the first obligor, denoted as  , is perturbed by a amount of  , the change in the expected value can be expressed as

(10)
 
  is the scaled default time so that we can force a default before maturity for the perturbed obligor. The details about how to achieve this and Eq.(10) can be found in Refs. [1,3]. 

The general comments on the methodology:

1.	As shown in Eq. (10), the first part of the right side of the equation is addressed to the scenario that the perturbed obligor defaults right after maturity ( ) in the base scenario and then was brought into the maturity ( ) in the perturbed scenario. Note that this part is somehow deterministic, especially for FNM trades. As test results shown in Section 2, the add-on of this part indeed increase the precision significantly. In a normal CSPDH and 1bps shock to each node, we need up to 50MM paths to reasonably model such maturity effect.

2.	The second part of the right side of Eq.(10) is attributed to the default event of the perturbed obligor that falls within the maturity. Note that, because the default time is rescaled such that a default event is forced to occur before maturity in every perturbed MC path, the volatility in CSPDH is suppressed greatly.

3.	In order to make Eq.(10) work, the payoff function f has to be continuous function of default times. Therefore certain types of trades of trade are excluded from using this analytic MC sensitivity. For example, the FNM trade without accrual. 

4.	If the valuation date is prior to the trade start date, a similar effect to that at the maturity can be found. Contrary to the maturity, a default event that happens right after the start date can be kicked out of the pay off in the perturbed scenario. This effect is discussed in the original paper [3] but not implemented in the model. Therefore, a limitation is added to prevent the model being used in the case that the valuation date is prior to the trade start date. 

The credit spread sensitivity is defined as the change of MTM when the credit spread curve of an obligor is shocked by a small amount. Assume the credit spread curve of the   node of the ith obligor is perturbed by a small amount , the credit spread sensitivity can be expressed as

(10)	 
Within the current modeling framework, the credit spread sensitivities are all computed via Jacobians [4, 5, 6].  Suppose we have calculated the sensitivity of base tranche value of protection and Val01 with respect the change of all hazard rate nodes. The sensitivities with respect to the credit spread curves can then be expressed as

(11)	 

The Jacobian  can be calculated using the single name CDS valuation model.

Within the current pricing framework (ref. https://finpricing.com/lib/FiBond.html), the FNM trade is calculated via MC simulation and a compound correlation approach. Therefore the MTM can be expressed as

(12)	 

Where   and   are the values for the protection leg and premium leg, respectively.   is the traded spread. The information of how to calculate these two legs can be found in Ref. [1]. Using Eq.(10), we can work out the analytic MC sensitivity for a FNM trade.

The credit spread sensitivity for a CDO2&3 trade is the most complicated model in the Oscar/Fritz credit library. As we can find in Ref. [7], a CDO2&3 trade is valued by finding a flattened RE-CDO.  Assume that a RE-CDO trade has an attachment point A and detachment point D. Just like a regular bespoke CDO, the credit spread sensitivity can be expressed as

(13)	 

When the credit spread of an obligor is changed, the attachment point of the RE-CDO will change, if we run a re-flattening. In order words, we need to calculate the Jacobian with respect to the attachment point. Taking attachment point value of protection as the example, the Jacobian can be written as

(14)	 

Note that the first two parts in Eq.(14) are the same as that for a bespoke trade. Within the current analytic sensitivity modeling framework, the third part reflects the contribution of the changed attachment point in the perturbed scenario. Note that all the components in Eq. (14), except for  , can be computed within the analytic sensitivity framework for bespoke CDO. 

The computation of  has to use the CDO2&3 mapping model, in which a Monte Carlo simulation for a CDO2&3  trade and a semi-closed form valuation model for RE-CDO are involved.  In the model, the attachment point of RE-CDO is found such that MTM of both a CDO2&3 trade and its RE-CDO matches at compound correlation of 0.2. 

Mathematically,   can be calculated by

(15) 	 


Now we see that the semi-analytic MC sensitivity has to be used to calculate   via Eq.(10). 

The default sensitivity is a kind of stress tests matching situations in which a credit default event has occurred or is perceived to be imminent. Within the current market standard modeling framework, a loss given default (  for the ith obligor) is claimed in the event of default. The principal of most junior tranche is reduced by   and the ith obligor is excluded from the collateral pool. 

By definition, the default sensitivity for the ith obligor in a CDO2&3 trade can be expressed as

(22)	 .
 
When calculating , the CDO2&3 trade has a loss with the amount of    and adjusted a new attachment point and detachment point in the child pools, where the perturbed obligor belongs to. 


