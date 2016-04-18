# thunder-factorization

[![Latest Version](https://img.shields.io/pypi/v/thunder-factorization.svg?style=flat-square)](https://pypi.python.org/pypi/thunder-factorization)
[![Build Status](https://img.shields.io/travis/thunder-project/thunder-factorization/master.svg?style=flat-square)](https://travis-ci.org/thunder-project/thunder-factorization) 

> algorithms for large-scale matrix factorization

Many common factorization algorithms do not have standard parallelized implementations in the Python ecosystem. This package provides distributed implementations of `PCA`, `ICA`, `NMF`, and others that target the distributed computing engine [`spark`](https://github.com/apache/spark), as well as wrapping local implementations from [`scikit-learn`](https://github.com/scikit-learn/scikit-learn) with an identical API.

The package includes a collection of `algorithms` that can be `fit` to data, all of which return matrices with the results of the factorization. Compatible with Python 2.7+ and 3.4+. Built on [`numpy`](https://github.com/numpy/numpy), [`scipy`](https://github.com/scipy/scipy), and [`scikit-learn`](https://github.com/scikit-learn/scikit-learn). Works well alongside [`thunder`](https://github.com/thunder-project/thunder) and supprts parallelization via [`spark`](https://github.com/apache/spark), but can also be used on local [`numpy`](https://github.com/numpy/numpy) arrays.

## installation
```
pip install thunder-factorization
```

## example

Here's an example computing PCA

```python
# create high-dimensional low-rank matrix

from sklearn.datasets import make_low_rank_matrix
X = make_low_rank_matrix(n_samples=100, n_features=100, effective_rank=5)

# use PCA to recover low-rank structure

from factorization import PCA
algorithm = PCA(k=5)
W, T = algorithm.fit(X)
```

## api

All algorithms have a `fit` method with returns the components of the factorization

#### `fit(X)`

Fits the algorithm to a data matrix
- `X`: data matrix, in the form of an [`numpy`](https://github.com/numpy/numpy) `ndarray`, a [`bolt`](https://github.com/bolt-project/bolt) `array`, or a [`thunder`](https://github.com/thunder-project/thunder) `series`, with dimensions `ncols x nrows`
- returns multiple arrays representing the factors, in the same form as the input

## algorithms

Here are all the available algorithms with their options.

#### `W, S, A = ICA(k=3, kPCA=None, svdMethod='auto', maxIter=10, tol=0.000001, seed=None).fit(X)`

Unmixes statistically independent sources: `S = XW^T` (unmixing) or `X = S * A^T` (mixing).

Parameters to constructor:
- `k`: number of sources
- `maxIter`: maximum number of iterations
- `tol`: tolerance for stopping iterations
- `seed`: seed for random number generator that initializes algorith.

`spark` mode only:
- `kPCA`: number of principal components used for initial dimensionality reduction,
   default is no dimensionality reduction.
- `svdMethod`: method for computing the SVD; `"auto"`, `"direct"`, or `"em"`; see
   SVD documentation for details.

Return values:
- `W`: demixing matrix, dimensions `k x ncols`
- `S`: sources, dimensions `nrows x k`
- `A`: mixing matrix (inverse of `W`), dimensions `ncols x k`

#### `W, H = NMF(k=5, maxIter=20, tol=0.001, seed=None).fit(X)`

Factors a non-negative matrix as the product of two small non-negative matrices: `X = H * W`.

Parameters to constructor:
- `k`: number of components
- `maxIter`: maximum number of iterations
- `tol`: tolerance for stopping iterations
- `seed`: seed for random number generator that initializes algorithm.

Return values from `fit`:
- `H`: components, dimensions `nrows x k`
- `W`: weights, dimensions `k x ncols`

#### `T, W = PCA(k=3, svdMethod='auto', maxIter=20, tol=0.00001, seed=None).fit(X)`

Performs dimensionality reduction by finding an ordered set of components formed by an orthogonal projection
that successively explain the maximum amount of remaining variance: `T = X * W^T`.

Parameters to constructor:
- `k`: number of components
- `maxIter`: maximum number of iterations
- `tol`: tolerance for stopping iterations
- `seed`: seed for random number generator that initializes algorithm.

`spark` mode only:
- `svdMethod`: method for computing the SVD; `"auto"`, `"direct"`, or `"em"`; see
   SVD documentation for details.

Return values from `fit`
- `T`: components, dimensions `nrows x k`
- `W`: weights, dimensions `k x ncols`


#### `U, S, V = SVD(k=3, method="auto", maxIter=20, tol=0.00001, seed=None).fit(X)`
Generalization of the eigen-decomposition to non-square matrices: `X = U * diag(S) * V^T`.

Parameters to constructor:
- `k`: number of components
- `maxIter`: maximum number of iterations
- `tol`: tolerance for stopping iterations
- `seed`: seed for random number generator that initializes algorithm.

`spark` mode only:
- `svdMethod`: method for computing the SVD; `"auto"`, `"direct"`, or `"em"`;
      * `direct`: explicit computation based eigenvalue decomposition of the covariance matrix.
      * `em`: approximate iterative method based on expectation-maximization algorithm.
      * `auto`: uses `direct` for `ncols` < 750, otherwise uses `em`.

Return values from `fit`:
- `U`: left singular vectors, dimensions `nrows x k`
- `S`: singular values, dimensions `k`
- `V`: right singular vectors, dimensions `ncols x k`

## tests

Run tests with 

```bash
py.test
```

Tests run locally with [`numpy`](https://github.com/numpy/numpy) by default, but the same tests can be run against a local [`spark`](https://github.com/apache/spark) installation using

```bash
py.test --engine=spark
```
