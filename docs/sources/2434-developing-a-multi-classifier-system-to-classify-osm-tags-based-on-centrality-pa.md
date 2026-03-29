# Developing a multi-classifier system to classify OSM tags based on centrality parameters

| Field | Value |
|-------|-------|
| **Authors** | Sajjad Hassany Pazoky, Parham Pahlavani |
| **Year** | 2021 |
| **Category** | Electoral system design and implementation |
| **Case Study** | Iran |
| **Relevance Score** | 1.0 |
| **Citation Count** | 10 |
| **Source** | openalex |
| **DOI** | [10.1016/j.jag.2021.102595](https://doi.org/10.1016/j.jag.2021.102595) |
| **Full Text** | [Open Access](https://doi.org/10.1016/j.jag.2021.102595) |

## Abstract

Misclassification of features is a major source of uncertainty in OpenStreetMap (OSM). This study is an automated data-enrichment study whose primary goal is predicting road classes based on multi-classifier systems (MCSs). In this regard, fourteen parameters (thirteen centrality parameters and length) that were assumed to have the highest impact on the classes of the features were calculated for the features. Choosing Tehran, Iran, as the test case, no ground truth was available; therefore, the tags assigned by the Iranian identified experts were fed to several classifiers including random forest, decision tree, support vector machine (SVM), Artificial Neural Network (ANN), and naïve Bayes. Using the five-fold cross-validation method, the overall accuracy of the classifiers was 93.55%, 90.19%, 88.50%, 83.06%, and 25.06%, respectively. Using Gini importance showed that closeness, eccentricity, and length were the most important parameters affecting the classification. To further enhance the accuracy, MCSs were used to fuse the results. The employed methods were weighted majority voting, naïve Bayes, Dempster-Shaffer, decision template, and behavior knowledge space (BKS). Experimenting different fusion methods with different combinations of the input classifiers led to enhanced results, in which BKS being fed with the combination of SVM and random forest scored the highest with an accuracy of 97.19%. Also, the methodology was tested on two other major cities of Iran, namely Mashhad and Karaj, and the BKS fusion method resulted in the accuracy of 93.18% and 87.86%, respectively.
