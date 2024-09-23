---
layout: post
title:  "LLM vulnerability mitigation literature review"
date:   2024-09-22 23:35:09 -0700
categories: jekyll update
---

## LLMs and security vulnerability mitigation: research paper review

**Summary 1: Deep learning and web applications vulnerabilities detection: An approach based on large language models**

“Deep Learning and Web Applications Vulnerabilities Detection: An Approach Based on Large Language Models” by Nana et al (2024) presents a comprehensive review of vulnerability detection for web applications, and proposes the use of Large Language Models (LLMs) to improve current security measures. 

This paper begins with highlighting vulnerabilities commonly found in Web Applications, and then suggests some countermeasures and deep learning (DL) approaches for vulnerability detection. The Open Web Application Security Project (OWASP) regularly classifies the top 10 web application security risks, and in this paper, they classify vulnerabilities into three main categories: Improper Input Validation such as SQL injections (SQLi) and Cross-Site Scripting (XSS), Improper session management , and improper authorization and authentication. The existing countermeasures largely fall into 5 categories, and there is potential for DL techniques to be applied to these approaches. In short, they are Secure programming, Static Analysis, Dynamic Analysis, Black Box Testing, and Intrusion Detection Systems. These approaches do fall short in that some of them rely on pre-written rules, or limited coverage. DL techniques currently show promise; in one of the studies reviewed, the authors managed to improve performance of their web application firewall (WAF) using DL techniques, by implementing a model that makes use of a WAF layered architecture baked on long short-term memory (LSTM) for DDoS, and SQLi/XSS detection, achieving over 89% accuracy for both. However, DL techniques face limitations, such as limited availability of data due to enterprise security, and due to most datasets for literature being made publicly available, are often outdated as well. 

LLMs pose as a promising alternative to this limitation, as LLMs can be fine tuned for web vulnerability detection on small datasets. Notably, the authors cite several examples of fine-tuned BERT (Bidirectional encoder representations from transformers) models tuned specifically for cybersecurity. CyBERT was fine tuned with Masked Language Modelling (MLM) and has been trained on a large corpus of Cyber Threat Intelligence and is able to perform multi-label classification of attacks. SecureBert has a customized tokenizer specific for the cybersecurity domain, and outperforms other models in cybersecurity masked word prediction and sentiment analysis. Lastly, VulBERTa and VulDeBerta are vulnerability detection models for C/C++ code.  VulBERTa has demonstrated success in outperforming existing approaches on binary and multi-class vulnerability classification, while VulDeBert places an emphasis on detecting vulnerabilities in system function calls. 

The authors conclude that LLMs have provided several interesting results, but can’t be generalized to web applications as of now. In terms of future research, they then first propose to experiment with an LLM-based approach for detecting web attacks by using publicly available datasets, and to build a local dataset utilizing a to-be-derived data collection strategy to gather  malicious URLs from Burkinabe cyberspace. 

**Summary 2: Large Language Models in Cybersecurity: State-of-the-Art**

"Large Language Models in Cybersecurity: State-of-the-Art" explores the growing role of large language models (LLMs) in the cybersecurity domain, focusing on both defensive and adversarial applications. The paper provides a comprehensive review of how LLMs are transforming traditional cybersecurity practices, aligning these innovations with standards like the NIST Cybersecurity Framework (CSF) and MITRE Adversarial Tactics, Techniques, and Common Knowledge (ATT&CK), and the challenges posed by research gaps in the field.

LLMs have demonstrated significant promise in defensive cybersecurity applications, particularly in identifying, protecting, and detecting cyber threats. The "Identify" function emphasizes LLMs' ability to assess risks by analyzing massive amounts of unstructured data and providing risk mitigation strategies and risk reporting. For "Protect," LLMs contribute to proactive monitoring and maintenance through tasks like web content filtering, password leakage prevention via honeywords, automated vulnerability fixing, and secure coding assistance. Most of these models were based on GPT-3 or GPT-3.5 and optimized through specific prompt phrasing and improved context understanding. LLMs also excel in the "Detect" function, where they are applied to anomaly detection in system logs, which can be noisy or overwhelming for traditional tools, penetration testing processes, tabletop exercise writing, and real-time vulnerability detection, further revolutionizing threat detection methods.

On the offensive side, the paper delves into how malicious actors are using LLMs through the lens of the ATT&CK Matrix, which consists of fourteen categories. LLMs are primarily used for automating reconnaissance and information gathering processes, generating phishing websites for credential harvesting, and creating customized malware that can successfully bypass antivirus and endpoint protection systems. Minimal effort is required to employ tools like PassGPT, a GPT-2 based password modeling solution, to streamline the initial access and credential access stages of the attack chain, and even harder to track with the code obfuscation and encoding capabilities that are accessible.

The authors identified gaps in knowledge in the defensive application of LLMs particularly for the "Respond" and "Recover" functions. While LLMs are promising in enhancing real-time response strategies, fewer studies focus on incident response or digital forensics. This gap points to future research directions where LLMs could offer more robust post-attack solutions to prevent data loss and accelerate system recovery. Such solutions are rather achievable considering LLM’s generative capabilities to write incident summaries and deliver consistency and speed, which are the most essential factors in limiting damage.

Reviewing these latest developments in LLM use in cybersecurity help illustrate their dual utility, offering powerful tools for both defense and attack. The authors finally call for more research into adversarial applications to balance the opportunities and threats posed by LLMs, hence helping security researchers accurately assess risks of LLM-based attacks and keep ahead of advancements by malicious actors.

**Summary 3: SQLIGPT: Evaluating and utilizing large language models for automated SQL injection black-box detection**

SQL injection (SQLI) is a common web application vulnerability that involves inserting malicious SQL code into queries, allowing attackers to manipulate or access sensitive information. Various testing methods, detection methods, and tools have been developed to combat this. Recent advancements in technology, particularly the use of Large Language Models (LLMs), are becoming increasingly popular and showing promising results for detecting and mitigating SQLI vulnerabilities in a system. 

Large Language Models (LLMs) are increasingly used to enhance SQLI detection methods. Their ability to learn from datasets enables them to understand and identify complex attack patterns compared to traditional methods. LLMs benefit from large training datasets but can also work with smaller ones, allowing for adaptive and evolving detection capabilities. 

A black-box LLM-based SQLI detection scanner was developed and it is an open-source project called SQLIGPT. The results in their journal displayed that it was able to identify all 45 targets in SQL MicroBenchMark, outperforming six other advanced scanners. These scanners were ZAP, BurpSuite, Arachni, Sqlmap, SQIRL, and PentestGPT using GPT 3.5, GPT-4, and Claude-3.   

SqliGPT was created by evaluating different LLM's performances and expanding on their capabilities. There was a performance check between LLM4SQLI-Manual and LLM4Sqli-Sqlmap. The manual LLM generally performed poorly due to manual testing of SQLI. The SQLmap is an automated SQLI detection using LLMs to generate SQLmap commands, which showed promising results. As a result, LLM4Sqli-Sqlmap was used as a baseline for SqliGPT.
LLM4Sqli-Sqlmap faced multiple challenges. Its detection efficiency remains low, especially in advanced or difficult targets and constraints related to bypassing insufficient defense mechanisms. Two modules were added on top of the Sqlmap to create SqliGPT. These were the Defense Bypass Module and the Strategy Selection Module.

We can simplify SqliGPT into a 7-step process. Firstly, the user provides a URL for the web application that they want to test. Secondly, the tool observes the content from the URL to gather information. Thirdly, It uses a process called Strategy Selection. This analyzes where SQL injections might occur and decides which technique to use first and where the location of it is. It then uses its Sqlmap to test the spots for SQL injections. If Sqlmap is unable to find any vulnerabilities, a method called Defense Bypass is deployed. This tool examines any security defenses and tries to create new methods to bypass the defense. Sqlmap then uses new methods to test the application again and if any vulnerabilities are found, a report would be sent to the user.

LLMs continue to make significant advancements in SQLI detection. Tools like SqliGPT offer promising improvements over traditional methods, but there are however challenges that arise from using such tools. High costs, false positives, and a lack of deep logical reasoning are all challenges that arise from using LLMs for SQLIs. 

**Summary 4: Conclusion about the studied domain**

The integration of Large Language Models (LLMs) into vulnerability detection and mitigation has played a huge role in the advancement of cybersecurity practices. They offer enhanced capabilities in identifying attack patterns and automating detection, making them useful tools for identifying threats and vulnerabilities, especially SQL injections (SQLI).

An overview of the current state of web vulnerabilities highlighted by OWASP details the common threats that are faced by web applications, and paired with a review of the current best practices for counter-measures we can see deep learning offering the potential to improve on these counter-measures. However, due to limitations such as resource expense and data sample requirements for deep learning solutions, LLMs pose itself as a solution to limited data resources with specialized fine tuning and customized tokenizers.

While LLMs continue to make substantial progress in detection accuracy and efficiency, some challenges arise. High costs to continue operation, the potential of false positives, and the limitation in logical reasoning must all be accounted for. Continued research and refinement of these models are needed to overcome these obstacles. This will allow for the expansion of applicability in various cybersecurity frameworks. 

As LLMs continue to be used by both attackers and defenders, strategies must be established to maximize benefits and minimize risks associated with their continued usage. With time, LLMs will develop and be more advanced. They will play an important role in the cybersecurity field. LLMs will provide organizations with robust tools to proactively defend themselves against malicious actors and will be an effective vulnerability mitigation tool.

The NIST and MITRE ATT&CK frameworks help security researchers classify both defensive and adversarial uses of LLMs within the cybersecurity field. The frequency of publications across their various categories demonstrates how LLMs can improve understanding and awareness of cyber risk across the evolving security landscape.

Additionally, the dual-use nature of LLMs has been well stressed, leading to a better holistic understanding of the challenges and opportunities presented by the weaponization of LLMs. Current research highlights significant gaps in incident response and recovery, where LLMs could offer substantial value, as well as prioritizing strategies against LLM-based attacks targeting initial access, defense evasion, and execution phases.
	


#### References

Gui, Z., Wang, E., Deng, B., Zhang, M., Chen, Y., Wei, S., Xie, W., & Wang, B. (2024, August 7). SQLIGPT: Evaluating and utilizing large language models for automated SQL injection black-box detection. MDPI. https://www.mdpi.com/2076-3417/14/16/6929 
Nana, S. R., Bassole, D., Guel, D., & Sie, O. (2024). Deep learning and web applications vulnerabilities detection: An approach based on large language models. International Journal of Advanced Computer Science and Applications, 15(7). https://doi.org/10.14569/ijacsa.2024.01507135 
Motlagh, F. N., Hajizadeh, M., Majd, M., Najafi, P., Cheng, F., & Meinel, C. (2024). Large Language Models in Cybersecurity: State-of-the-Art. arXiv. https://arxiv.org/abs/2402.00891
