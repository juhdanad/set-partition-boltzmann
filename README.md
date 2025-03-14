# Integer Partition Boltzmann calculator

## Goal

This package contains functions to calculate statistical properties of integer partitions, when the distribution of the partitions follows the Boltzmann distribution. You will need numpy to use it. It optionally works with sympy (for symbolic results) and numba (for faster calculation). This package aims to be efficient, with each method being az most quadratic in running time and linear in memory consumption (calculated with the input array length).

## Theory

An [integer partition](https://en.wikipedia.org/wiki/Partition_(number_theory)) is a way to write a positive integer as a sum of positive integers. In an other sense, it is a way we can partiton a set if the elements are indistinguishable.

For a given positive integer N we are able to list all of its partitions. For example, the partitions of 4 are:
 * 1 + 1 + 1 + 1
 * 1 + 1 + 2
 * 2 + 2
 * 1 + 3
 * 4

This package treats integer partitions as set partitions, so for example the integer partition 4=2+2 means that we had an input set of size 4, and we took a partition of that: two subsets, each having size 2.

The [Boltzmann distribution](https://en.wikipedia.org/wiki/Boltzmann_distribution) is a probability distribution used in statistical physics. It assigns a probability to each state based on its energy. This package assumes that the energy of a partition is the sum of the subsets' energies, and that a subset's energy only depends on its size. Then we have some `E_i` energy values, where i is the subset size. In the Boltzmann distribution we also have a temperature parameter `beta`. The probability of a partition is then proportional to `exp(-beta*E)`, where `E` is the energy of the partition.

## Weights

We can unify the `E_i` and `beta` constants. To do that, we introduce the weights: `w_i=exp(-beta*E_i)`. Then the probability of a partition is proportional to the product of the weights of its addends.

`w_1` should always be positive, to avoid division by zero.

## Usage

First you have to create a calculator object.
`import integer_partition_boltzmann`
 * `integer_partition_boltzmann.calculator.for_sympy()` returns a calculator object that you can use for symbolic calculations.
 * `integer_partition_boltzmann.calculator.for_numba()` returns a calculator object that contains functions compiled by the numba library.

Now suppose that you have a calculator object `calc` for sympy.
Also, suppose that you executed `w0,w1,w2,w3=sympy.symbols("w0 w1 w2 w3")` to get the weights. `w0` will remain unused.

### Partition function

One important aspect of the Boltzmann distribution is the partition function.

```
>>> calc.get_partition_functions(numpy.array([w0,w1,w2,w3]))
array([1, w1, w1**2 + w2, w1**3 + w1*w2 + w3], dtype=object)
```

 * We can ignore the 0th element.
 * The first element is the partition function for input array size 1. It is just `w1`, as the only partition is 1.
 * The second element is for the set with two elements. The corresponding partitions are 2=1+1 and 2=2. These give the partition functiion `w1**2 + w2`.
 * The partitions of 3 are 3=1+1+1, 3=1+2, 3=3. These correspond to `w1**3 + w1*w2 + w3`.

### Expected number of subsets with given size

We can ask: if we take a random partition and count the number of subsets which have exactly i elements, what will be the expected value of the result?

Suppose that the input set has always two elements. Then the probability mass function for the input set size is `[0,0,1]` (the 0th element is the probability that the set has no elements).

```
>>> calc.get_subset_quantity_expectation_values(numpy.array([w0,w1,w2]),numpy.array([0,0,1]))
array([0, 2*w1**2/(w1**2 + w2), w2/(w1**2 + w2)], dtype=object)
```

The probability that we will get the 2=1+1 partition is proportional to `w1**2`. The other partition (2=2) has probability proportional to `w2`. Then the probability of the first case (since the sum of probabilities must be 1) is `w1**2/(w1**2 + w2)`. This case yields two singleton sets, and the other case yields none, therefore the expected value of singleton sets is `2*w1**2/(w1**2 + w2)`. Similarly, the last element of the result array is the expected number of subsets having two elements.

### Probability mass function of number of subsets having given size

We now have the expected number of subsets with a fixed size. We might also need the probability mass function of the number of such subsets. Suppose again that the input has two elements. We are interested in the distribution of singletons.

```
>>> calc.get_subset_number_pmf_for_size(numpy.array([w0,w1,w2]),numpy.array([0,0,1]),1)
array([w2/(w1**2 + w2), 0, w1**2/(w1**2 + w2)], dtype=object)
```

We can use the last subsection to explain the result. The 2=1+1 case yields two singleton sets and has probability `w1**2/(w1**2 + w2)`, this explains the 2nd (last) element of the result array. There is no output that has only one singleton set, this explains the middle element. Finally, the case 2=2 has probability `w2/(w1**2 + w2)` and produces no singletons, therefore this is the 0th element of the result.

## Numba support

You can use the same functions on the numba calculator. The only caveat is that the functions only accept float64 arrays in this case. For example:

```
>>> calc.get_subset_quantity_expectation_values(numpy.array([1.,1.,1.]),numpy.array([1.,1.,1.]))
array([0. , 2. , 0.5])
```

## Numerical stability

I did not check the calculations for numerical instabilities. However, if you multiply each weight with a corresponding power, for example: `w_i=w_i*(1.1**i)`, the resulting probabilities should not change (partition functions should change). You can use this fact to get an idea of the instability, or find a magnitude range which works for you.

## Supported by

Supported by the ÚNKP-20-2-SZTE-411 New National Excellence Program of the Ministry for Innovation and Technology from the source of the National Research, Development and Innovation Fund.

![Project logo](https://drive.google.com/uc?export=view&id=18E1JWPujcyyvmyUSCFIZjUuIKohy54Zr)
