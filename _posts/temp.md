## 1 Distinct subsequences

Key point: 
*  Maintain a two-dimensional array which says the answer of the partial input s[0..i] and t[0..j]
* mem[i+1][j+1] = mem[i][j] + mem[i+1][j] if s[i] = t[j] 
* otherwise mem[i+1][j+1] = mem[i+1][j]

Review:
* A problem considers two string can sometime uses a two-dimensional array to store the answer for the sub-problem (Dynamic programming)

Almost same questions:

* Longest common subsequences
* Shortest common supersequences

## 2 Max Sum of Rectangle No Large than K

Key point:

* Do exhaustive searching for one dimension, that is to say, consider all sub-section for rows or coloumns, and then for another dimension, compute the max sum of the subarray no large than K. 

The questions with the same idea:

*  Number of submatrices that sum to target


## 3 Maximal Rectangle

Key point:

* Maintain a matrix of the same size of the original matrix, each item in the matrix is a tuple of <left,right,height>, which means the left, right and the height of the rectangle that covers the current item.

## 4 Binary Tree Cameras

Key point:

* Tree problems can almost be solved recursively. The subproblem is the same problem with the children as the root node. The question is, how to figure out the states that are returned by the subproblem?

## 5 Concatenated words

Key point:

* This problem can be reduced to word break problem. Word break problem is solved by using DP. That is, maintain a array to record if not a word can be broken at a position.

Related question:

* Word break

## 6  Range Module TODO
## 7  Race Car TODO

## 8 Shortest path to get all keys

Key point: For find the optimal answer, BFS is a must.

## 9 Stamping the sequence TODO

Key point: 

* Think backwards. Generally, this is a searching problem, can then you realize that starting from the beginning seems like a impossible solution even with bruteforce method. 

## 10 Minimum cost to hire K workers
## 11 Find in Mountain Array

Key point:

* Find the peak in a array
* Binary search the increasing part and then decreasing part.

## 12 Longest Duplicate String TODO

Key point:

* Method 1: Binary search to probe the longest size the substring. For each size, use a slide window to find the duplicate string.
* Method 2: (standard) Suffix array 

## 13 Tallest Billboard (Need one more execerise)

Key point:

* Because the sum of the array is bouned, we can maintain a state for each possible sum.
* First to think how to solve the problem brute forcelly, and then to think what computation are repeatedly performed.

## 14 Largest Component size by common factor (Need one more exercise)

Key point:

* Connected component question can be answered by union-find.

Similar questions:

* Redundent connection

## 15 Find the shortest substring (TODO)

## 16 Distinct Subsequences 

Key point:

* It is easy to identify the sub-problem. The question is, how to write the transfer function.

## 17 Three equal parts
## 18 Remove boxes, Burst Ballons, Minimum Cost to merge stones

Key point:

* Burst Ballons: reverse thinking, as design mem[l][k] means to burst all the ballons between l and k.
* Remove boxes, design memorization as mem[l][r][k], means the solution for array from l to r with k boxes as the same color as array[r]. The key observation is that for strings like abcddd could be reduced to solution as abc + ddd

## 19 Longest Increasing Path in a Matrix

Key point: DFS and to cache the result for each point

## 20 Reverse pairs, Count of small numbers after self, Count of range sum

Key point: 

* Reverse pairs: using BIT or using modifed merge sort
* Count of small numbers after self: using merge sort
* Count of range sum: using merge sort

## 21 Kth smallest number in multiplication table, Find k-th smallest pair distance, kth-smallest prime fraction

https://leetcode.com/problems/k-th-smallest-prime-fraction/discuss/115819/Summary-of-solutions-for-problems-%22reducible%22-to-LeetCode-378

## 261,1081

Key point: Monotonic Stack

## Super washing machine

TODO

## Longest Increasing Subsequence

Key point: Because the longest increasing subsequence upto point i has nothing to do with the elements come after the current point, we can solve it using dynamic programming. Let's say we remember the longest subsequence upto point i-1, noted as dp[i-1], what should be do about number[i]? If number[i] is larger than dp[i-1], we are happily to append it to dp[i], but what if number[i] is smaller than dp[i-1]?

## Range sum query 2d

Binary indexed trees
