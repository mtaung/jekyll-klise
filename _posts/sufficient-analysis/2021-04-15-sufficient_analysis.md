---
title: Sufficient Analysis, or "How to do statistics with lost data"
date: 2021-04-15
tags: [statistics, python]
description: A real world example of salvaging lost statistical data, to carry out an ANOVA and independent samples t-test from only sufficient statistics in python.
usemathjax: true
---

This post is pertaining [one of my github repositories](https://github.com/mtaung/sufficient_analysis). If you ever need to do anything like this, may the gods bless your endeavours, because somewhere along the line things really got fucked up. If you're in a pinch and just want the how-to and good stuff, **skip all my rambling** and go to the repo, which is all you need to do this. The subjct is also likely more relevant for students in applied fields like Psychology, or undergraduates just starting to study statistics with more rigour.

Despite the lack of novelty, it's an incredibly efficient technique. Lehman{% sidenote lehman 'Richard Lehman provided some additional [commentary](https://go.gale.com/ps/anonymous?id=GALE%7CA13838424&sid=googleScholar&v=2.1&it=r&linkaccess=abs&issn=00031305&p=AONE&sw=w) to Larson, whom I refer to below.' %} has credited this method with allowing him to reanalyze textbook examples or published results where the raw data is not available, optimising large datasets by computin sufficient statistics only once, and catching out publication errors in a manner similar to Heathers & Quintana's data thugging {% sidenote datathugs 'Great article about them [here](https://www.sciencemag.org/news/2018/02/meet-data-thugs-out-expose-shoddy-and-questionable-research), if you are not aware of their work.' %} that was a big part of raising awareness of the replication crisis.

#### Our Particular Scenario

In 2018, as part of an AI bootcamp I participated in a small hackathon where we trained agents using a rolling horizon evolutionary algorithm{% sidenote rhea 'A collection of papers on RHEA by the original authors can be found [here]'(https://gaigresearch.github.io/projects/rhea). %}. It was a pretty gruelling week due to the time constraint, and was the tail end of a 6 week short-fat course so everyone was tired{% sidenote 1 'One of my teammates was the reinforcement learning researcher [Dani Hernandez](https://danielhp95.github.io/), who introduced me to Jekyll, and I am grateful for his friendship through grad school.' %}. At the end of the final testing session, a working model was produced but with everyone burning out the comprehensive data for the agent was not correctly recorded.  Mistakes like this happen all the time, especially among fraught and tired students during their studies, as it did to us. By the time we realised our error, it was too late to amend the code and compute more runs, and we just wanted to be done with it all so I was asked to salvage what we could.

All we had were some aggregate results, and one question we had to answer was "How well did the agent perform?".

#### Working with nothing but means

The best I could come up with at the time was to try and salvage the dataset by fabricating a surrogate dataset with the same sample statistics as our observed results.

I suspected that I wasn't the first person to come up with this, so I searched on google and found one paper {% sidenote larson 'Larson actually authored two papers on this approach, this is specifically from Larson, D. A. (1992). Analysis of variance with just summary statistics as input. The American Statistician, 46(2), 151-152.' %} on this. It's something that anyone with a background in maths and statistics would be able to work out themselves, but there's a source to show my banality.

The paper opened with a quote that made us feel especially guilty.

> "More times than I care to recount, I have been asked how to use various statistical software packages to perform a one-way analysis of variance (ANOVA) and appropriate multiple comparison procedures on summary statistics for which the original data have been discarded."

If, like us, you still managed to record your mean and variance/standard deviation, then you're in luck because you have what Larson called the *sufficient statistics*, and you can compute a surrogate sample:

$$
\begin{align*}
y_i = x\bar + \frac{s}{\sqrt{n}},  i = 1, 2, ..., n-1 \\ 

y_n = nx - y_i (n-1) \\
\end{align*}
$$

where you have a sample of size $n$, mean $\bar{x}$, and sample variance $s^2$, which the surrogate sample of $y_i$ all possess.

This means that you should also have some idea about the underlying structure of your samples, such as your $n$ number of observations and $k$ groups. 

In other words, you end up with $n$ number of observations (per group) with a value of $\bar{x}$, and just one observation (per group) with a value of $nx - y_i (n-1)$, that is used to retain the original variance.

Once you have this sample, it is possible to compute your desired test (in our case a one-way ANOVA). On my repo from a few years back, there are some rough example code showing how to do this for both an [ANOVA](https://github.com/mtaung/sufficient_analysis/blob/master/ANOVA%20from%20sufficient%20statistics.ipynb) and a [T test](https://github.com/mtaung/sufficient_analysis/blob/master/Independent%20Samples%20t-test%20from%20sufficient%20statistics.ipynb), though it is not very thorough and only sits as a proof of concept. With some additional effort, it's also possible to do this for an n-way ANOVA. The original papers display how to conduct this method in Excel, and SAS.

#### But it's not *really* sufficient for real work

It shouldn't be needed to say, but this isn't at all close enough to what could be accomplished were the full data intact. You will miss out on very essential parts of doing real research, like exploratory graphing and statistics. You have a very rough idea of how your sample looks if you compute the confidence intervals for your sample statistics, or your test, but you don't have certainty about any outliers, nor can you examine for violations of critical assumptions like homogeneity of variance{% sidenote robustness 'Linear models are generally more robust than some undergrad psych classes would lead you to believe when there is a strong reliance on ticking boxes about assumptions. Ideally it is always better to  meet assumptions, but [monte carlo simulations](https://psycnet.apa.org/record/2010-18785-001) have shown that they can withstand a lot. **But, I do not think this lenience should be held for violations about homogeneity of variance.** If that is violated, you really ought to use another test.' %}.

It really is a last ditch effort to salvage *something* at all, and it shouldn't be relied upon unless absolutely needed.