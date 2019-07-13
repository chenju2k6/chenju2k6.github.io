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
