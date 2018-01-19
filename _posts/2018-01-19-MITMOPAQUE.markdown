---
layout: post
title:  "Shuffle-in-the-middle vs Opaque"
date:   2018-01-19 10:00:28 -0500
categories: jekyll update
---

Consider this query: how many apple records and how many banana records in the fruit table? The CCS paper and Opaque answer this query from two approaches. 

The CCS paper answers the question based on two operators - mappers and reducer. The mapper filters out all the non-apple-banana records and the reducer count the number of apples and bananas.  To hide leaky information (which input records are apples), the CCS paper introduces shuffler operator. Shuffler operates on the intermediate data and breaks the link between the intermediate key (apple) and the input records. Note that, even with this shuffler, the information like - how many apples - is still leaked. 

The Opaque paper answers the question using one operator - oblivious sorter. In the first run, the input records are sorted by their fruit type, so that two apples are neighbors in the output. Next, the query could be answered by scanning the dataset. But this is not enough. When the dataset is too big to fit into one machine, the simple scan could still leak the information like - how many apples.  To solve this problem, the Opaque first let machines to exchange some statistical information. After doing this, all the apples or bananas are extended to contain one information, the count. Note that only the last apple in the apple group contains the real count, the count attribute for all the other apples are just dummy value. And then, the Opaque apply oblivious sorter for the second time, with the count attribute being the sort key. After this, we only need to throw away the dummy items and the query can be answered.
