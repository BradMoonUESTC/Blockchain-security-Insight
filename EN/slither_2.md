# Slither Insight 102

### What Are We Talking About When We Talk About Reentrancy

Written on June 22, 2024, 22:44

---

As I write this article, I have just received the notification that our previous [SAST paper](https://2024.esec-fse.org/details/fse-2024-research-papers/16/Static-Application-Security-Testing-SAST-Tools-for-Smart-Contracts-How-Far-Are-We-) has been awarded the ACM SIGSOFT Distinguished Paper award. It feels great, even though I was the second author and didn't contribute much. However, it is related to Slither, so I will mention it here.

## Reentrancy

The reentrancy vulnerability has been discussed and researched extensively over the years. In academia, it is usually considered a read-write problem, and research often addresses similar issues. Whether through source code or bytecode, both static analysis and dynamic execution aim to find abnormal reads and writes to identify reentrancy.

This approach is indeed effective, but we need to expand our discussion beyond just the reentrancy vulnerability. There are two directions for this expansion:
1. Expansion of reentrancy types—can read-write issues be further refined to explain more types of reentrancy?
2. Expansion of read-write issues—do read-write issues occur in other vulnerabilities as well? Is there a general model to describe a series of vulnerabilities?

Over the past few years, I have intermittently abstracted and summarized these questions, and I can provide some preliminary conclusions:
1. Expansion of reentrancy types—read-write issues can be further refined, and different types of read-write conflicts can explain more types of reentrancy.
2. Expansion of read-write issues—indeed, there is a general model that can incorporate price manipulation, reentrancy, and classic MEV front-running, forming a theoretical model for read-write issues in smart contracts.

The content on reentrancy will be divided into three articles, addressing the two questions and the optimization direction in Slither. This Slither Insight 102 will focus on the first question.

## Expansion of Reentrancy Types—Can Read-Write Issues Be Further Refined to Explain More Types of Reentrancy?

I am grateful to my master's advisor for the insightful theory introduced when I first encountered this vulnerability, which attempts to explain reentrancy using database theory, namely, dirty read, non-repeatable read, and phantom read.

Through extensive analysis of reentrancy scenarios, we have found interesting conclusions: indeed, by extrapolating read-write conflicts through database theory, we can differentiate different types of reentrancy.

Before diving into this, let's review the basic definitions:

1. **Dirty Read**: A transaction reads data that has been written by another transaction but not yet committed.
2. **Non-Repeatable Read**: A transaction reads data that has been modified and committed by another transaction after the initial read.
3. **Phantom Read**: A transaction reads newly inserted data that has been committed by another transaction after the initial read.

Based on these definitions, we can see that the most primitive form of reentrancy, such as reentrancy via fallback to read an unupdated balance, corresponds to a dirty read. In simple terms, this type of reentrancy reads the state before the completion of the function logic, akin to reading old data to bypass a balance check. Similarly, a dirty read involves reading uncommitted data from another transaction—both scenarios involve reading relatively old data.

Now, let's explore whether non-repeatable read and phantom read have their corresponding cases. Taking the [lendf.me ERC777 reentrancy attack](https://blog.csdn.net/BradMoon/article/details/123133041) as an example, the essence of the attack is as follows:
1. The project calculates the difference in balances before and after deposit to track the amount.
2. The attacker deposits fake tokens initially and then deposits real tokens during reentrancy, performing two separate difference calculations.
3. The two difference calculations result in a discrepancy where depositing 1 unit of tokens results in 2 units of recorded deposits.
4. The core issue is that during the first deposit, the outer layer incorrectly reads the new data when it should be using the old data to calculate the difference, leading to the vulnerability.

From this attack, we see that the inner reentrancy introduces new data, which the outer layer incorrectly reads, linking it to non-repeatable read or phantom read. Essentially, the inner reentrancy causes the outer layer to experience non-repeatable read or phantom read. Conversely, we could argue that the outer layer's non-repeatable read or phantom read leads to the inner layer's dirty read.

In summary, by leveraging database theory, we can identify the relationship between reentrancy and database read-write conflicts. There are many avenues for further exploration, such as the connection with reentrancy locks and other theoretical extensions. These are all worthy of investigation.

---

