# Implementation notes

The multi-threaded implementation of SMC is fairly straightforward. The attempt to efficiently utilize multiple cores is largely focused around low-overhead resampling / selection, since the remaining steps are embarrassingly parallel.

There is also some attention paid to Julia's threading interface and, for both parallel and serial execution, the need to avoid dynamic memory allocations. All code for the main package was written using Julia's core and base functionality.

Generating the ancestor indices in sorted order is accomplished primarily through use of the uniform spacings method for generating a sequence of sorted uniform random variates developed by Lurie & Hartley (1972) and described by Devroye (1986, p. 214). In multi-threaded code, the multinomial variate giving the number of ancestor indices each thread should simulate using a thread-specific local vector of particle weights is simulated by using a combination of inversion sampling and a a Julia implementation of the ```btrd``` algorithm of Hörmann (1993), which simulates binomially distributed variates in expected $\mathcal{O}(1)$ time. Simulating a multinomial random variate is accomplished by simulating a sequence of appropriate Binomial random variates. The code-generating function for copying particles was adapted from [a suggestion on Julia Discourse](https://discourse.julialang.org/t/how-to-copy-all-fields-without-changing-the-referece/945/5) by Greg Plowman.

For the demonstration code, use of the [StaticArrays.jl](https://github.com/JuliaArrays/StaticArrays.jl) package provides dramatic improvements for multivariate particles, and is highly recommended for fixed-size vector components of particles. It is possible to use the counter-based RNGs in the [RandomNumbers.jl](https://github.com/sunoru/RandomNumbers.jl) package in place of the default random number generator; this was not working on the Julia-0.7 master branch at the time of development, and is slightly slower than the MersenneTwister RNG provided by Julia's base.