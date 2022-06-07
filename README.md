# MultiScaleArrays

[![Join the chat at https://gitter.im/JuliaDiffEq/Lobby](https://badges.gitter.im/JuliaDiffEq/Lobby.svg)](https://gitter.im/JuliaDiffEq/Lobby?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Build Status](https://github.com/SciML/MultiScaleArrays.jl/workflows/CI/badge.svg)](https://github.com/SciML/MultiScaleArrays.jl/actions?query=workflow%3ACI)
[![Coverage Status](https://coveralls.io/repos/github/JuliaDiffEq/MultiScaleArrays.jl/badge.svg)](https://coveralls.io/github/JuliaDiffEq/MultiScaleArrays.jl)
[![codecov](https://codecov.io/gh/JuliaDiffEq/MultiScaleArrays.jl/branch/master/graph/badge.svg)](https://codecov.io/gh/JuliaDiffEq/MultiScaleArrays.jl)

MultiScaleArrays.jl allows you to easily build multiple scale models which are
fully compatible with native Julia scientific computing packages like
DifferentialEquations.jl or Optim.jl. These models utilize
a tree structure to describe phenomena of multiple scales, but the interface allows
you to describe equations on different levels, using aggregations from lower
levels to describe complex systems. Their structure allows for complex and dynamic
models to be developed with only a small performance difference. In the end, they present
themselves as an `AbstractArray` to standard solvers, allowing them to be used
in place of a `Vector` in any appropriately made Julia package.

## Tutorials and Documentation

For information on using the package,
[see the stable documentation](https://multiscalearrays.sciml.ai/stable/). Use the
[in-development documentation](https://multiscalearrays.sciml.ai/dev/) for the version of
the documentation, which contains the unreleased features.

## Example

The usage is best described by an example. Here we build a hierarchy where
Embryos contain Tissues which contain Populations which contain Cells, and the
cells contain proteins whose concentrations are modeled as simply a vector
of numbers (it can be anything linearly indexable).

```julia
using MultiScaleArrays
struct Cell{B} <: AbstractMultiScaleArrayLeaf{B}
    values::Vector{B}
end
struct Population{T<:AbstractMultiScaleArray,B<:Number} <: AbstractMultiScaleArray{B}
    nodes::Vector{T}
    values::Vector{B}
    end_idxs::Vector{Int}
end
struct Tissue{T<:AbstractMultiScaleArray,B<:Number} <: AbstractMultiScaleArray{B}
    nodes::Vector{T}
    values::Vector{B}
    end_idxs::Vector{Int}
end
struct Embryo{T<:AbstractMultiScaleArray,B<:Number} <: AbstractMultiScaleArrayHead{B}
    nodes::Vector{T}
    values::Vector{B}
    end_idxs::Vector{Int}
end
```

This setup defines a type structure which is both a tree and an array. A picture of a possible
version is the following:

<img src="https://user-images.githubusercontent.com/1814174/27211626-79fe1b9a-520f-11e7-87f1-1cb33da91609.PNG">

Let's build a version of this. Using the constructors we can directly construct leaf types:

```julia
cell1 = Cell([1.0; 2.0; 3.0])
cell2 = Cell([4.0; 5.0])
```

and build types higher up in the hierarchy by using the `constuct` method. The method
is `construct(T::AbstractMultiScaleArray, nodes, values)`, though if `values` is not given it's
taken to be empty.

```julia
cell3 = Cell([3.0; 2.0; 5.0])
cell4 = Cell([4.0; 6.0])
population  = construct(Population, deepcopy([cell1, cell3, cell4]))
population2 = construct(Population, deepcopy([cell1, cell3, cell4]))
population3 = construct(Population, deepcopy([cell1, cell3, cell4]))
tissue1 = construct(Tissue, deepcopy([population, population2, population3])) # Make a Tissue from Populations
tissue2 = construct(Tissue, deepcopy([population2, population, population3]))
embryo = construct(Embryo, deepcopy([tissue1, tissue2])) # Make an embryo from Tissues
```
