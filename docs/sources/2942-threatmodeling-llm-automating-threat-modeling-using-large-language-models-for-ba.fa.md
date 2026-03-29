# ThreatModeling-LLM: Automating Threat Modeling using Large Language Models for Banking System

<div dir="rtl" markdown>

| نویسندگان | Shuiqiao Yang, Tingmin Wu, Shigang Liu, David Nguyen, Seunghwan Jang, A. Abuadbba |
|---|---|
| **سال** | 2024 |
| **دسته‌بندی** | اصلاح نظام بانکی و مالی |
| **مطالعه موردی** | ایران |
| **امتیاز ارتباط** | 1.0 |
| **تعداد استنادها** | 19 |
| **شناسه دیجیتال** | [10.48550/arXiv.2411.17058](https://doi.org/10.48550/arXiv.2411.17058) |

## چکیده

Threat modeling is a crucial component of cybersecurity, particularly for industries such as banking, where the security of financial data is paramount. Traditional threat modeling approaches require expert intervention and manual effort, often leading to inefficiencies and human error. The advent of Large Language Models (LLMs) offers a promising avenue for automating these processes, enhancing both efficiency and efficacy. However, this transition is not straightforward due to three main challenges: (1) the lack of publicly available, domain-specific datasets, (2) the need for tailored models to handle complex banking system architectures, and (3) the requirement for real-time, adaptive mitigation strategies that align with compliance standards like NIST 800-53. In this paper, we introduce ThreatModeling-LLM, a novel and adaptable framework that automates threat modeling for banking systems using LLMs. ThreatModeling-LLM operates in three stages: 1) dataset creation, 2) prompt engineering and 3) model fine-tuning. We first generate a benchmark dataset using Microsoft Threat Modeling Tool (TMT). Then, we apply Chain of Thought (CoT) and Optimization by PROmpting (OPRO) on the pre-trained LLMs to optimize the initial prompt. Lastly, we fine-tune the LLM using Low-Rank Adaptation (LoRA) based on the benchmark dataset and the optimized prompt to improve the threat identification and mitigation generation capabilities of pre-trained LLMs.

</div>