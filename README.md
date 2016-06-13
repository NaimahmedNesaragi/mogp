# Multi-Output Gaussian Processes

Python implementation for
[Collaborative Multi-Output Gaussian Processes for Collections of
Sparse Multivariate Time Series]
(https://drive.google.com/open?id=0Bx7depbNYaFIRndmU1cxM1YtaXo3U2RsLVZSWkxsdjZOcFpF).


## Example

Here is an example to illustrate how to train
Collaborative Multi-Output Gaussian Processes (COGPs) given a
collection of sparse multivariate time series,
and make predictions.

We first create an instance of `MultiOutputGP`.
In this example, there are three channels which are assumed to be
generated by two latent Gaussian processes.

```python
from elbo_pqn import MultiOutputGP
mogp = MultiOutputGP(n_channels=3, n_latent_gps=2)
```

For multi-output GPs, we consider several schemes to regularize the
combination weight matrix with size `n_channels` by `n_latent_gps`.
We can pass the `w_reg_group` argment into `MultiOutputGP` to specify the
regularization option.
Four options are available:

* `'none'`: no regularization.
* `'individual'`: L1 regularized over each individual entries.
* `'row'`: group lasso with each row as a group.
* `'column'`: group lasso with each column as a group.

For `w_reg_group='individual'`,
an extra argument `w_reg` specifies the regularization strength.
The smaller the value of `w_reg` is the sparser the combination weight matrix
would be.
On the other hand, for `'row'` and `'column'`,
an extra argument `w_reg` specifies the regularization strength.
The larger the value of `w_reg` is the sparser the combination weight matrix
would be.

Here is an example that regularizes each column as a group:

```python
mogp = MultiOutputGP(n_channels=3, n_latent_gps=2, n_inducing_points=20,
                     w_reg_group='column', w_reg=1.5)
```

Given the training data `train_data` in the form of a list of
three tuples, each of which contains two numpy vectors `(xi, yi)`
as the time series for the i-th channel.
The data structure `train_ts` contains the hyperparameters to be trained
using `mogp.train()`.

```python
train_ts = mogp.gen_collection(train_data)
mogp.train(train_ts, maxiter=50)
```

For a test time series `(test_obs_x, test_obs_y)`, where
`test_obs_x` contains three numpy vectors corresponds to the time points
in each channel, and the `test_obs_y` that contains three numpy vectors
stores the corresponding observed values.
The numpy vector `test_mis_x` is a numpy vector that stores the time points
to compute the predictive Gaussians over.
The predictive Gaussian mean and covariance matrix are `post_mean`
and `post_cov` respectively.

```python
test_obs_ts = TimeSeries(test_obs_x, test_obs_y, mogp.shared)
post_mean, post_cov = mogp.predictive_gaussian(test_obs_ts, test_mis_x)
```