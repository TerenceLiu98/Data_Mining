# Association Analysis: Basic Concepts and Algorithms

## Motivation

Many business enterprises accumulate large quantities of data from their day-to-day operations. For example, huge amounts of customer purchase data are collected daily at the checkout counters of grocery stores. This such data, commonly known as **market basket transactions** and here is a example:

| TID | ITEMS |
| --- | ----- |
| 1 | {Bread, Milk} |
| 2 | {Bread, Diapers, Beer, Eggs} |
| 3 | {Milk, Diapers, Beer, Cola} |
| 4 | {Bread, Milk, Diapers, Beer} |
| 5 | {Bread, Milk, Diapers, Cola} |

Each row in this table corresponds to a transaction, which contains a unique identifier labeled **TID** and a set of items bought by a given customer. The retailers are interested in the customers' purchasing behavior. This chapter presents a methodology known as **association analysis**, which is useful for discovering interesting relationships hidden in large data sets. The uncovered relationships can be represented in the form of **association rules** or sets of frequent items.

## Definition: Frequent Itemset

Using the example above:

| TID | ITEMS |
| --- | ----- |
| 1 | {Bread, Milk} |
| 2 | {Bread, Diapers, Beer, Eggs} |
| 3 | {Milk, Diapers, Beer, Cola} |
| 4 | {Bread, Milk, Diapers, Beer} |
| 5 | {Bread, Milk, Diapers, Cola} |

* **Itemset** is a collection of one or more items: {Bread, Milk}
  * k-itemset: An itemset that contains $k$ items
* **Support count($\sigma$)**
  * Frequency of occurrence of an itemset
  * E.g. $\sigma(\{\text{Milk, Bread, Diaper}\}) = 2$
* **Support**
  * Fraction of transaction that contain an itemset
  * E.g. $s(\{\text{Milk, Bread, Diaper}\}) = \frac{2}{5}$
* **Frequent Itemset**
  * An itemset whose support is greater than or equal to a **minsup** threshold

**An important property of** an itemset is its support count, which refers to the number of transactions that contain a particular itemset. Mahtematically, the support count, $\sigma(X)$, for an itemset $X$ can be stated as follows:
$$ \sigma(X) = |\{t_i|X \subseteq t_i, t_i \in T\}|$$

## Definition: Association Rule

An association rule is an implication expression of the form $X \longrightarrow Y$, where $X$ and $Y$ are disjoint itemsets, i.e., $X \cap Y = \emptyset$
* Example: {Milk, Diaper} $\rightarrow$ {Beer}
The strength of an association rule can be measured in terms of its **support** and **confidence**. Support determines how often a rule is applicable to a given data set, while confidence determines how frequently item in $Y$ appear in transactions that contain $X$. The formal definiation of these metrics are:
$\begin{align*}
  \text{Support, } s(X \rightarrow Y) &= \frac{\sigma(X \cup Y)}{N}\\
  \text{Confidence, } c(X \rightarrow Y) &= \frac{\sigma(X \cup Y)}{\sigma(X)}
\end{align*}$
In statistical way to comprehend these formulas, in my opinion, the **support** is the probability of $X \cup Y$ and **confidence** is the conditional probability of $Y$ given $X$, is $P\{Y|X\} = \frac{P\{X \cup Y\}}{P(X)}$

**Example**
Consider the rule {Milk, Diapers} $\rightarrow$ {Beer}. Since the support count for {Milk, Diapers, Beer} is $2$ and the total number of transactions is $5$, the rule's support is $\frac{2}{5} = 0.4$. The rule's confidence is obtained by dividing the support count for {Milk, Diapers, Beers} by the support count for {Milk, Diapers}. Since there are $3$ transactions that contain milk and diapers, the confidence for this rules is $\frac{2}{3} = 0.67$

**Why We Use Support and Confidence?**

Support is an important measure because a rule that has very low support may occur simply by chance. A low support rule is also likely to be uninteresting from a business perspective because it may not be profitable to promote itmes that customers seldom by together.

Confidence, on the other hand, measures the reliability of the inference made by a rule. For a given rule $X \rightarrow Y$, the higher the confidence, the more likely it is for $Y$ to be present in transactions that contain $X$. Confidence also provides an estimate of the conditional probability of $Y$ given $X$.

## Definition: Association Rule Discovery

Given a set of transaction $T$, find all the rules having support $\geq$ $minsup$ and confidence $\geq$ $minconf$, where $minsup$ and $minconf$ are the corresponding support and confidence thresholds.

A brute-force approach for mining association rules is to compute the support and confidence for every possible rule. This approach is prohibitively from a data set. More specifically, the total number of possible rules extracted from a data set that contains $d$ items is
$$ R = 3^d - 2^{d + 1} + 1$$
* List all possible association rules
* Compute the support and confidence for each rule
* Prune rules that fail the minsup and minconf thresholds

$\Rightarrow$ <font color="red">**Computationally prohibitive**</font>

Therefore, a common strategy adopted by many association rule mining algorithms is to decompose the problem into two major subtasks:

1. **Frequent Itemset Generation**, whose objective is to find all the itemsets that satisfy the $minsup$ thresold. These itemsets are called frequent itemsets.
2. **Rule Genreation**, whose objective is to extract all the high-confidence rules from the frequent itemsets found in the previous step. These rules are called strong rules.

### Frequent Itemset Generation

A lattice structure can be used to enumerate the list of all possible itemsets.

![Screenshot 2019-04-10 at 00.21.40.png](https://i.loli.net/2019/04/10/5cacc6988c899.png)

This figure shows an itemset lattice for $I = \{a, b, c, d, e\}$. In general, a data set taht contains $k$ items can potentially generate up to $2^k - 1$ frequent itemsets, excluding the null set. Because $k$ can be very large in many practical applications, the search space of itemsets that need to be explored is exponentially large.

* brute-force approach for finding frequent itemsets is to determine the support count for every **candidate itemset** in the lattice structure. To do this, we need to compare each candidate against every transaction, an option that is shown below.

  ![Screenshot 2019-04-10 at 00.29.10.png](https://i.loli.net/2019/04/10/5cacc85d2cd82.png)
1. **Reduce the number of candidate itemsets(M)**. The Apriori principle, describled in the next section, is an effective way to eliminate some of the candidate itemsets without counting their support values.
2. **Reduce the number of comparisions**. Instead of matching each candidate itemset against every transaction, we can reduce the number of comparisons by using more advanced data structures, either to store the candidate itemsets or to compress the data set.

**The Complexity $\sim$ O(NMw) $\Rightarrow$ Expensive since $M = 2^d$**

## The Apriori Principle

This section describes how the support measure helps to **reduce the number of candidate itemsets explored during frequent itemset generation**. The use of support for pruning candidate itemsets is guided by the following principle.

### Definition (*Apriori* Principle)

If an *itemset is frequent, then all of its subset must also be frequent*.

Suppose $\{c, d, e\}$ is a frequent itemset. Clearly, any transaction that contains $\{c, d, e\}$ must also contain its subset, $\{c,d\}, \{c, e\}, \{d, e\}, \dots$. As a result, if $\{c, d, e\}$ is frequent, than all subsets of $\{c, d, e\}$ must also be frequent.

**Conversely, if an itemset such as $\{a, b\}$ is infrequent, then all of its supersets must be infrequent too**.

This strategy of trimming the exponential search space based on the support measure is known as **support-based pruning**. Such a pruning strategy is made possible by a key property of the support measure, namely, that the support for an itemset never exceeds the support for its subsets. This property is also known as the **anti-monotone** property of the support measure.

Apriori uses **breadth-first search** and a Hash tree structure to count candidate item sets efficiently. It generates candidate item sets of length $k$ from item sets of length $k - 1$. Then it prunes the candidates which have an infrequent sub pattern. According to the downward closure lemma, the candidate set contains all frequent $k$-length item sets. After that, it scans the transaction database to determine frequent item sets among the candidates.

### Frequent Itemset Generation in the Apriori Algorithm 

*Apriori* is the first association rule mining algorithm that pioneered the use of support-based pruning to systematically control the exponential growth of candidate itemsets. 

![Screenshot 2019-04-11 at 00.29.42.png](https://i.loli.net/2019/04/11/5cae19fdd2a2f.png)

Initially, every item is considered as a candidate $1$-itmeset. After counting their supports, the candidate itemsets {coke} and {Eggs} 

### Apriori Algorithm 

* Method: 
  * Let $k = 1$ 
  * Generate frequent itemsets of length $1$ 
  * Repeat until no new freqent itemsets are identified 
    * Generate length $(k + 1)$ candidate itemsets from length $k$ frequent itemsets 
    * Prune candidate itemsets containing subsets of length $k$ that are infrequent 
    * Count the support of each candidate by scanning the DB 
    * Eliminate candidates that are frequent, leaving only those that are frequent 
    * 

