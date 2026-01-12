## Jaccard similarity and distance
For 2 sets $A$ and $B$ 
$$\text{Jaccard Similarity}(A,B) = \frac{∣A \cap B∣}{∣A \cup B∣}$$
Range between 0 and 1. 
![](Pasted%20image%2020260112014341.png)
Jaccard distance $$\text{Jaccard Distance = 1-Jaccard Similarity}$$
## Bootstrapping dataset
- Statistical resampling technique, uses sampling with replacement, create multiple datasets, from OG one, improve stability+accuracy.
- Compute statistic on synthetic dataset, repeated $B$ times, and use distribution of results as needed.
