---
layout: post
title: Peak Finding
tags: [algorithm, 6006]
---
This is a problem from MIT Course 6.006.

# 1D Peak Finding
**Problem** Given an 1d array $A$ with n elements, find the index $i$ of the
peak element $A[i]$ where $A[i] \geq A[i+1]$ and $A[i] \geq A[i-1]$. For the
boundaries, assume that $A[-1] = A[n] = -\infty$.

**Lemma 1** Any array with length great than 0 has at least one peak element.  
*Proof.* Any array has a maximum element which is also a peak element. QED

*Algorithm.* Given an array $A$ with $n$ elements:

1.   Take the middle element of $A$, $A[\frac{n}{2}]$, and compare that element to its neighbors
2.   If it's great or equal to its neighors, then $A[\frac{n}{2}]$ by definition is a
     peak element. Return $\frac{n}{2}$.
3.   Else, if the left neighbor is greater than the middle element, recurse on the
     left subarray, not including the middle element. Otherwise, recurse on the
     right subarray.

*Analysis.* By lemma 1, each subarray of $A$ must have a peak element. If the
left neighbor is great than the middle element, for the boundary, we have
$A[\frac{n}{2} - 1] > A[\frac{n}{2}]$, which ensures that any peak element found
in the left subarray is also a peak element in $A$. Same argument can be made
for the right neighbor.  
In each recursion, we reduce size $n$ to size $\frac{n}{2}$ in $O(1)$ time and combine the
result in $O(1)$ time. Thus the recurrence of this problem is $ T(n) = T(\frac{n}{2}) + O(1) $,
which gives the running time, $O(n \log n)$.

*Python Impelementation*
```Python
def peak1d(A, left=None, right=None):
    if left is None:
        left = 0
    if right is None:
        right = len(A) - 1
    if left > right:
        return None

    middle = (left + right) // 2
    neighbor = find_better_neighbor(A, middle, left, right)

    if middle == neighbor:
        return middle
    elif middle > neighbor:
        return peak1d(A, left, middle - 1)
    else:
        return peak1d(A, middle + 1, right)


def find_better_neighbor(A, i, left, right):
    """
    If A[i] has a better neighbor, return it. Otherwise return itself.
    """
    best = i
    if i - 1 >= left and A[i] < A[i - 1]:
        best -= 1
    elif i + 1 <= right and A[i] < A[i + 1]:
        best += 1
    return best
```

# 2D Peak Finding
**Problem** Given $n x n$ matrix $M$, find the indices of a peak element
$M[i][j]$ where the element is great or equal to its neighbors,
$M[i-1][j], M[i+1][j], M[i][j-1]$ and $M[i][j+1]$. For the boundaries, assume
that $M[-1][k] = M[k][-1] = M[n][k] = M[k][n] = -\infty(k = {0...n-1})$.

*Algorithm 1.* Given $n x n$ matrix $M$:

1.   Take the middle column of $M$ and find the maximum element in that column.
     Call that element divider. Compare the divider to its left and right
     neighbors.  
2.   If it's great or equal to its left and right neighbors, then the divider by
     definition is a peak element of $M$.  
3.   Else, if the left neighbor is greater than the divider, recurse on the left
     submatrix, not including the middle column. Otherwise recurse on the right
     submatrix.  

*Analysis.* Assume that the indices of the divider are (r1, c1), the indices of
the left neighbor are (r1, c2)(c2 = c1 - 1) and the indices of right neighbor
are (r1, c3)(c3 = c1 + 1).

**Claim 1** If $M[r1][c1] < M[r1][c2]$, any peak element in the left submatrix
of $M$ is a peak element of $M$.  
*Proof.* There are two cases according to where we find the peak element of the
submatrix.

-   case one. The peak element of the submatrix is at some column left to the
    column c2.  
    Obviously this is also a peak element of $M$.  
-   case two. The peak element of the submatrix is at column c2.  
    Suppose the indices of the peak element are (r2, c2).
    Because $M[r2][c2] >= M[r1][c2], M[r1][c2] > M[r1][c1]$ and $M[r1][c1]$ is
    greater than any element at column c1, $M[r2][c2] > M[r2][c1]$. Thus by
    definition $M[r2][c2]$ is also a peak element of $M$. QED  

Similarly, we can proof  
**Claim 2** If $M[r1][c1] < M[r1][c3]$, any peak element in the right submatrix
of $M$ is a peak element of $M$.

By claim 1 and claim 2, algorithm 1 always return a correct peak element of $M$.  
Now let's analysis the running time of algorithm 1. In each recursion, we reduce
size $m x n$ to size $m x \frac{n}{2}$ in $O(n)$ time and combine the result in
$O(1)$ time. Thus the recurrence is $ T(m, n) = T(m, \frac{n}{2}) + O(m) $, which
gives the running time $O(m \log n)$.

*Python Impelementation*(almost the same as the 1D version)
```Python
def peak2dv1(M, left=None, right=None):
    nrow = len(M)
    if nrow == 0:
        return None
    if left is None:
        left = 0
    if right is None:
        right = len(M[0]) - 1
    if left > right:
        return None

    middle = (left + right) // 2
    divider = find_maximum([(M[i][middle], (i, middle)) for i in range(nrow)])
    neighbor = find_better_neighbor(M, divider, left, right, 0, nrow - 1)

    if divider == neighbor:
        return divider
    elif divider[1] > neighbor[1]:
        return peak2dv1(M, left, middle - 1)
    else:
        return peak2dv1(M, middle + 1, right)


def find_maximum(l):
    max, max_indices = l[0]
    for e, ind in l[1:]:
        if e > max:
            max = e
            max_indices = ind
    return max_indices


def find_better_neighbor(M, ind, left, right, up, down):
    best = ind
    i, j = ind
    if i - 1 >= left and M[i][j] < M[i - 1][j]:
        best = (i - 1, j)
    elif i + 1 <= right and M[i][j] < M[i + 1][j]:
        best = (i + 1, j)
    elif j - 1 >= up and M[i][j] < M[i][j - 1]:
        best = (i, j - 1)
    elif j + 1 <= down and M[i][j] < M[i][j + 1]:
        best = (i, j + 1)
    return best
```
