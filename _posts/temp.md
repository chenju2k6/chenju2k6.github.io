## Distinct subsequences

Key point: 
*  Maintain a two-dimensional array which says the answer of the partial input s[0..i] and t[0..j]
* mem[i+1][j+1] = mem[i][j] + mem[i+1][j] if s[i] = t[j] 
* otherwise mem[i+1][j+1] = mem[i+1][j]

Review:
* A problem considers two string can sometime uses a two-dimensional array to store the answer for the sub-problem (Dynamic programming)

Almost same questions:

* Longest common subsequences

## Max Sum of Rectangle No Large than K

Key point:

* Fix one dimension, for another dimension, compute the max sum of the subarray no large than K.

The questions with the same idea:

*  Number of submatrices that sum to target


## Maximal Rectangle

Key point:

* Maintain a matrix of the same size of the original matrix, each item in the matrix is a tuple of <left,right,height>, which means the left, right and the height of the rectangle that covers the current item.

## Binary Tree Cameras

Key point:

* Tree problems can almost be solved recursively. The subproblem is the same problem with the children as the root node. The question is, how to figure out the states that are returned by the subproblem?

## Concatenated words

Key point:

This problem can be reduced to word break problem. Word break problem is solved by using DP. That is, maintain a array to record if not a word can be broken at a position.

Related question:

Word break


