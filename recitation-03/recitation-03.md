# CMPS 2200  Recitation 03

**Name (Team Member 1):**Jordan Sztejman  
**Name (Team Member 2):**_________________________



## Analyzing a recursive, parallel algorithm


You were recently hired by Netflix to work on their movie recommendation
algorithm. A key part of the algorithm works by comparing two users'
movie ratings to determine how similar the users are. For example, to
find users similar to Mary, we start by having Mary rank all her movies.
Then, for another user Joe, we look at Joe's rankings and count how
often his pairwise rankings disagree with Mary's:

|      | Beetlejuice | Batman | Jackie Brown | Mr. Mom | Multiplicity |
| ---- | ----------- | ------ | ------------ | ------- | ------------ |
| Mary | 1           | 2      | 3            | 4       | 5            |
| Joe  | 1           | **3**  | **4**        | **2**   | 5            |

Here, Joe (somehow) liked *Mr. Mom* more than *Batman* and *Jackie
Brown*, so the number of disagreements is 2:
(3 <->  2, 4 <-> 2). More formally, a
disagreement occurs for indices (i,j) when (j > i) and
(value[j] < value[i]).

When you arrived at Netflix, you were shocked (shocked!) to see that
they were using this O(n^2) algorithm to solve the problem:



``` python
def num_disagreements_slow(ranks):
    """
    Params:
      ranks...list of ints for a user's move rankings (e.g., Joe in the example above)
    Returns:
      number of pairwise disagreements
    """
    count = 0
    for i, vi in enumerate(ranks):
        for j, vj in enumerate(ranks[i:]):
            if vj < vi:
                count += 1
    return count
```

``` python 
>>> num_disagreements_slow([1,3,4,2,5])
2
```

Armed with your CMPS 2200 knowledge, you quickly threw together this
recursive algorithm that you claim is both more efficient and easier to
run on the giant parallel processing cluster Netflix has.

``` python
def num_disagreements_fast(ranks):
    # base cases
    if len(ranks) <= 1:
        return (0, ranks)
    elif len(ranks) == 2:
        if ranks[1] < ranks[0]:
            return (1, [ranks[1], ranks[0]])  # found a disagreement
        else:
            return (0, ranks)
    # recursion
    else:
        left_disagreements, left_ranks = num_disagreements_fast(ranks[:len(ranks)//2])
        right_disagreements, right_ranks = num_disagreements_fast(ranks[len(ranks)//2:])
        
        combined_disagreements, combined_ranks = combine(left_ranks, right_ranks)

        return (left_disagreements + right_disagreements + combined_disagreements,
                combined_ranks)

def combine(left_ranks, right_ranks):
    i = j = 0
    result = []
    n_disagreements = 0
    while i < len(left_ranks) and j < len(right_ranks):
        if right_ranks[j] < left_ranks[i]: 
            n_disagreements += len(left_ranks[i:])   # found some disagreements
            result.append(right_ranks[j])
            j += 1
        else:
            result.append(left_ranks[i])
            i += 1
    
    result.extend(left_ranks[i:])
    result.extend(right_ranks[j:])
    print('combine: input=(%s, %s) returns=(%s, %s)' % 
          (left_ranks, right_ranks, n_disagreements, result))
    return n_disagreements, result

```

```python
>>> num_disagreements_fast([1,3,4,2,5])
combine: input=([4], [2, 5]) returns=(1, [2, 4, 5])
combine: input=([1, 3], [2, 4, 5]) returns=(1, [1, 2, 3, 4, 5])
(2, [1, 2, 3, 4, 5])
```

As so often happens, your boss demands theoretical proof that this will
be faster than their existing algorithm. To do so, complete the
following:

a) Describe, in your own words, what the `combine` method is doing and
what it returns.
The combine method takes the two sorted lists left_ranks and right_ranks compares them to find times when the right_ranks element is smaller than the element from left ranks. It returns a list of the merge sorted list as well as the number of disagreements.
.  
.  
.  
.  
.  
.  
.  
.  
.  

b) Write the work recurrence formula for `num_disagreements_fast`. Please explain how do you have this.
The work recurrence for this formula is W(n) = 2W(n/2) + O(n). I got this work recurrence because the algorithm makes a recursive call to the two sorted lists which would have half the elements, making it n/2. This gives us two lists of n/2 making it 2W(n/2). Then the combine feaure works linearly making it O(n). The combined formula would be W(n) = 2W(n/2) + O(n).
.  
.  
.  
.  
.  
.  

c) Solve this recurrence using any method you like. Please explain how do you have this.
The recursion has log (base 2)(n) levels because we divide the problem size by 2 each time. Starting with size n, we get n/2, then n/4, then n/8, and so on until we reach size 1. This happens when n/2^k = 1, which means k = log₂(n). At each level, the combine operations do O(n) total work, so with log₂(n) levels, we get O(n log n) total work.

.  
.  
.  
.  
.  
.  
.  
.  
.  
.  
.  
.  


d) Assuming that your recursive calls to `num_disagreements_fast` are
done in parallel, write the span recurrence for your algorithm. Please explain how do you have this.
The span recurrence for this algorithm is similary S(n) = S(n/2) + O(n). this is because the lists can be run parallel but they require the slower of the lists to finish. Since both calls have the same size (n/2), the span is just S(n/2). Then the combine function must run sequentially after both recursive calls complete, adding O(n) to the span. This would make it S(n) = S(n/2) + O(n).
.  
.  
.  
.  
.  
.  
.  
.  
.  
.  
.  
.  

e) Solve this recurrence using any method you like. Please explain how do you have this.
S(n) = S(n/2) + O(n) -> S(n) = Θ(n). The recursion has log(base 2) (n) levels, and at each level the combine operation takes O(n/2^k) time where k is the level. The total span is O(n) + O(n/2) + O(n/4) + ... which forms a geometric series that sums to O(n). The first term dominates, giving us S(n) = Θ(n).
.  
.  
.  
.  
.  
.  
.  
.  
.  
.  
.  
.  

f) If `ranks` is a list of size n, Netflix says it will give you
lg(n) processors to run your algorithm in parallel. What is the
upper bound on the runtime of this parallel implementation? (Hint: assume a Greedy
Scheduler). Please explain how do you have this.
The upper bound of this parrallel implementation would be O(n). I got this using the Greedy Scheduler formula T_P(n) = O(W(n)/P + S(n)). With P = lg(n) processors, W(n) = O(n log n) from part c, and S(n) = O(n) from part e, we get T_P(n) = O(n log n / log n + n) = O(n + n) = O(n). This means the parallel runtime is bounded by the larger of the distributed work and the span, which gives us the upper bound runtime of O(n).
