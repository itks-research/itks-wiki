# Application of Randomness for Security and Privacy in Multi-Party Computation

<div dir="rtl" markdown>

| نویسندگان | R. Saha, Gulshan Kumar, G. Geetha, Mauro Conti, William J. Buchanan |
|---|---|
| **سال** | 2024 |
| **دسته‌بندی** | Political party formation and pluralism |
| **مطالعه موردی** | Iran |
| **امتیاز ارتباط** | 1.0 |
| **تعداد استنادها** | 6 |
| **شناسه دیجیتال** | [10.1109/TDSC.2024.3381959](https://doi.org/10.1109/TDSC.2024.3381959) |

## چکیده

A secure Multi-Party Computation (MPC) is one of the distributed computational methods, where it computes a function over the inputs given by more than one party jointly and keeps those inputs private from the parties involved in the process. Randomization in secret sharing leading to MPC is a requirement for privacy enhancements; however, most of the available MPC models use the trust assumptions of sharing and combining values. Thus, randomization in secret sharing and MPC modules is neglected. As a result, the available MPC models are prone to information leakage problems, where the models can reveal the partial values of the sharing secrets. In this paper, we propose the first model of utilizing a random function generator as an MPC primitive. More specifically, we analyze our previous development of the Symmetric Random Function Generator (SRFG) for information-theoretic security, where the system is considered to have unconditional security if it is secure against adversaries with unlimited computing resources and time. Further, we apply SRFG to eradicate the problem of information leakage in the general MPC model. Through a set of experiments, we show that SRFG is a function generator that can generate the combined functions (combination of logic GATEs) with <inline-formula><tex-math notation="LaTeX">$n/2$</tex-math><alternatives><mml:math><mml:mrow><mml:mi>n</mml:mi><mml:mo>/</mml:mo><mml:mn>2</mml:mn></mml:mrow></mml:math><inline-graphic xlink:href="kumar-ieq1-3381959.gif"/></alternatives></inline-formula> -private to <inline-formula><tex-math notation="LaTeX">$n$</tex-math><alternatives><mml:math><mml:mi>n</mml:mi></mml:math><inline-graphic xlink:href="kumar-ieq2-3381959.gif"/></alternatives></inline-formula>-private norms. As the main goal of MPC is privacy preservation of the inputs, we analyze the applicability of SRFG properties in secret sharing and MPC and observe that SRFG is eligible to be a cryptographic primitive in MPCdevelopments. We also measure the performance of our proposed SRFG-based MPC framework with the other randomness generation-based MPC frameworks and analyze the comparative attributes with the state-of-the-art models. We observe that our posed SRFG-based MPC is <inline-formula><tex-math notation="LaTeX">$\approx 30\%$</tex-math><alternatives><mml:math><mml:mrow><mml:mo>≈</mml:mo><mml:mn>30</mml:mn><mml:mo>%</mml:mo></mml:mrow></mml:math><inline-graphic xlink:href="kumar-ieq3-3381959.gif"/></alternatives></inline-formula> better in terms of throughput and also shows 100% privacy attainment.

</div>