# MMA.jl

[![Build Status](https://travis-ci.org/KristofferC/MMA.jl.svg?branch=master)](https://travis-ci.org/KristofferC/MMA.jl)

This module implements the MMA Algorithm in Julia as described by Krister Svanberg in [1].

The code in this module was made for a course in Structural Optimization and should be seen as educational. For real use it is likely better to use a more mature code base, for example [NLopt.jl](https://github.com/JuliaOpt/NLopt.jl) which contain a more modern MMA algorithm than the one implemented here.

## Usage

Ẁe will solve the nonlinear constrained minimization problem given [here](http://ab-initio.mit.edu/wiki/index.php/NLopt_Tutorial)

```julia
using MMA

# Define objective function
function f(x::Vector, grad::Vector)
    if length(grad) != 0
        grad[1] = 0.0
        grad[2] = 0.5/sqrt(x[2])
    end
    sqrt(x[2])
end

# Define a constraint function
function g(x::Vector, grad::Vector, a, b)
    if length(grad) != 0
        grad[1] = 3a * (a*x[1] + b)^2
        grad[2] = -1
    end
    (a*x[1] + b)^3 - x[2]
end

# Create the MMAModel with a relative tolerance on x
ndim = 2
m = MMAModel(ndim, f, xtol = 1e-6, store_trace=true)

# Add box constraints to the variables
box!(m, 1, 0.0, 100.0)
box!(m, 2, 0.0, 100.0)

# Add two nonlinear inequalities
ineq_constraint!(m, (x,grad) -> g(x,grad,2,0))
ineq_constraint!(m, (x,grad) -> g(x,grad,-1,1))

# Solve the problem
x0 = [1.234, 2.345]
results = optimize(m, x0)

# Print the results
print(results)

#Results of Optimization Algorithm
# * Algorithm: MMA._MMA
# * Starting Point: [1.234,2.345]
# * Minimizer: [0.3333332274782007,0.29629691337859027]
# * Minimum: 5.443316e-01
# * Iterations: 7
# * Convergence: true
#   * |x - x'| < 1.0e-06: true
#     |x - x'| = 8.33e-08
#   * |f(x) - f(x')| / |f(x)| < 1.5e-08: false
#     |f(x) - f(x')| / |f(x)| = 7.65e-08
#   * |g(x)| < 1.5e-08: false
#     |g(x)| = 9.19e-01
#   * stopped by an increasing objective: true
#   * Reached Maximum Number of Iterations: false
# * Objective Calls: 8
# * Gradient Calls: 8

# Print the trace
#println(results.trace)
#Iter     Function value   Gradient norm
#------   --------------   --------------
#     1     3.948114e-01     1.266427e+00
#     2     1.799667e-01     2.778292e+00
#     3     4.353204e-01     1.148579e+00
#     4     5.338213e-01     9.366429e-01
#     5     5.442500e-01     9.186955e-01
#     6     5.443315e-01     9.185578e-01
#     7     5.443316e-01     9.185577e-01
```

## References
[1] [The method of moving asymptotes - a new method for structural optimization](http://www.researchgate.net/publication/227631828_The_method_of_moving_asymptotesa_new_method_for_structural_optimization)

### Author
Kristoffer Carlsson - kristoffer.carlsson@chalmers.se
