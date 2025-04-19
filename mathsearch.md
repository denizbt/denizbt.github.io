---
layout: project-desc
title: "MathSearch"
project_logo: /assets/images/mathsearch-logo.png
project_date: September 2023 - May 2024
project_role: Technical Lead (Cornell Data Science)
permalink: /projects/mathsearch/
---

**Project Summary**

MathSearch is a scalable web search engine that uses ML to find math equations within PDFs. The machine learning pipeline uses optical character recognition and a fine-tuned YoloV5 model to detect and extract equations from PDFs, convert them into LaTeX strings, and use a Python package for String similarity to match our identified equations with text queries. In May 2024, my team and I successfully deployed the web application which ran the full data pipeline and supported concurrent user requests. Unfortunately due to our financial constraints, the product is not currently deployed.

As Technical Lead, I led a team of 15 frontend, backend, and fullstack developers from the initial stages of research through our pipeline design and to final deployment of our product. I also wrote code connecting our ML pipeline to our AWS backend.

**Challenges & Lessons**

Our primary challenge was our financial budget. As a student organization, we had a limited budget for perpetual deployment. Due to this, we conducted a cost-benefit analysis to optimize resource usage and cut costs without compromising performance.  For instance, we chose a no-cost String similarity Python package as the final step in our pipeline because it was more accurate and cost-effective than using the GPT API or tree similarity metrics. Unfortunately, the cost accrued from AWS services (especially Sagemaker which we used to host the YoloV5 model), was too high for us to keep our product deployed.

From this project, I learned how important it is to keep the future in mind even as you work hard on ironing out the details of a software project. I am proud of the work I did on MathSearch, and hope that one day we will have the funds to deploy it continuously.