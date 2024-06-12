# Slither Insight 101

Written at 2:04 AM on June 13, 2024
## Introduction
Slither is a Solidity static analysis tool based on AST. I started using it around the beginning of 2022, which is quite late compared to many experts. I spent a year and a half fixing and adding many rules, encountering numerous pitfalls. Of course, I also developed some personal ideas about writing rules, involving some theories and covering some potential vulnerability understandings, which I record here.

## Some Conclusions
Firstly, to address whether Slither is useful, I personally think it is extremely useful (obviously). Developers can use Slither to create many specific rules to check particular Solidity logic.

However, for those who want to turn Slither into an in-house product, there are two points to note:
1. Slither's license is AGPL-V3, which I like to call a "viral" license. Its characteristics are as follows:
    - You must provide the source code to users.
    - Any modifications to the code must also be open-sourced and use the AGPLv3 license.
    - The code must include the license and copyright notices.
    - Changes made must be documented.
    - Users can freely use, modify, and share the code.
    - Quality assurance is waived, liability is limited.
    - The license applies to the entire program.
    - Additional restrictions are prohibited.

    For commercial use of AGPL-V3:
    1. You must open-source the complete modified source code of AGPLV3.
    2. If integrating AGPLV3 code into a Network Service, you must provide users with the modified complete source code.
    3. You cannot link AGPLV3 code with code under an incompatible license.
    4. You must continue to publish the open-source code under the AGPLV3 license.
    5. You must provide installation information so users can modify and recompile the program.
    6. If the license is violated, the copyright holder can terminate the user's license.

Simply put, if you use it, you must open-source it. I assume many audit companies use it to make products, or as a tool to attract clients, but you haven't open-sourced it, which is quite interesting.

2. If you want to make it a product for external use, then the requirements become very high, and the original version of Slither has a terrible rate of false positives and false negatives, which means you must:
    - Spend significant effort to reduce false positives by using lots of test cases, especially real contracts and those affected by security incidents, to run Slither and identify every false positive.
    - Conduct in-depth analysis of smart contract vulnerabilities, or those issues that real auditors would be concerned about, understand their universal models, and understand their patterns, which actually requires a high level of abstract understanding.

Based on these points, optimizing and developing Slither is not an easy task. It has consumed a lot of my energy but allowed me to advance in abstract understanding of some vulnerabilities, which I believe is effective and a necessary process for auditors who want to progress.

In subsequent Slither articles, I will specifically introduce each Slither rule I modified, explaining the reasons for the modifications, insights, and compromises.

Yes, this requires compromises. You need to strike a balance between false negatives and false positives. If your pattern is too strict, you'll have a high rate of false negatives; if it's too loose, you'll have a high rate of false positives.

## Some Background
My modifications are based on the original version of Slither. The modified repository is mainly located at https://github.com/MetaTrustLabs/falcon-metatrust, but you can also check out https://github.com/BradMoonUESTC/slither, although this only contains some core rules.