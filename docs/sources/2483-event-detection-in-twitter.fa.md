# Event Detection in Twitter

<div dir="rtl" markdown>

| نویسندگان | Jianshu Weng, Bu‐Sung Lee |
|---|---|
| **سال** | 2021 |
| **دسته‌بندی** | Electoral system design and implementation |
| **مطالعه موردی** | Iran |
| **امتیاز ارتباط** | در انتظار |
| **تعداد استنادها** | 714 |
| **شناسه دیجیتال** | [10.1609/icwsm.v5i1.14102](https://doi.org/10.1609/icwsm.v5i1.14102) |

## چکیده

Twitter, as a form of social media, is fast emerging in recent years. Users are using Twitter to report real-life events. This paper focuses on detecting those events by analyzing the text stream in Twitter. Although event detection has long been a research topic, the characteristics of Twitter make it a non-trivial task. Tweets reporting such events are usually overwhelmed by high flood of meaningless “babbles”. Moreover, event detection algorithm needs to be scalable given the sheer amount of tweets. This paper attempts to tackle these challenges with EDCoW (Event Detection with Clustering of Wavelet-based Signals). EDCoW builds signals for individual words by applying wavelet analysis on the frequencybased raw signals of the words. It then filters away the trivial words by looking at their corresponding signal autocorrelations. The remaining words are then clustered to form events with a modularity-based graph partitioning technique. Experimental results show promising result of EDCoW.

</div>