# AcyclicGraphs.jl
Package to Deal with AcyclicGraphs

```julia
edges =[
  :B=>:T,
  :T=>:Y,
  :T=>:M,
  :G=>:M,
  :G=>:Y,
  :M=>:Y,
  :S=>:M,
  :S=>:P,
  :S=>:Y,
  :S=>:T,
  :P=>:T,
  :A=>:T,
  :A=>:P,
  :A=>:Y] |> unique


directcause(x::Symbol, edges) = [v for v in edges if v[2] == x]

directcause(:Y, edges)
# julia> directcause(:Y, edges)
# 5-element Array{Pair{Symbol,Symbol},1}:
#  :T => :Y
#  :G => :Y
#  :M => :Y
#  :S => :Y
#  :A => :Y

function anycausepath(x::Symbol, edges; used=[], target=missing)
  starter = (target===missing)
  (target===missing) && (target = x)
  (length(used) > 1) && (used[end]==target) && (return (used[1:(end-1)]))

  edgereduced = [v for v in edges if !(v[1] ∈ used)]

  causes = directcause(x, edgereduced)
  isa(causes, Array) && (length(causes)==0) && return [], []


  nodesout = []
  for i in 1:length(causes)
    push!(nodesout, [causes[i][1]])
    xparent = causes[i][1]
    nodes = anycausepath(xparent, edges, used = [used..., x], target=target)
    for ii in 1:length(nodes)
        isa(nodes[ii], Array) &&  push!(nodesout, ([xparent, nodes[ii]...]))
        !isa(nodes[ii], Array) && push!(nodesout, [xparent, nodes[ii]])
    end
  end

  nodesout |> unique
end

zparents = directcause(:Y, edges)

z = anycausepath(:Y, edges)
for v in z
    x = (isa(v, Array) ? [:Y, v...] : [:Y, v])
    println(join(string.(reverse(x)),"->"))
end
# T->Y
# B->T->Y
# S->T->Y
# P->T->Y
# S->P->T->Y
# A->P->T->Y
# A->T->Y
# G->Y
# M->Y
# T->M->Y
# B->T->M->Y
# S->T->M->Y
# P->T->M->Y
# S->P->T->M->Y
# A->P->T->M->Y
# A->T->M->Y
# G->M->Y
# S->M->Y
# S->Y
# A->Y

anycause  = [(isa(v, Array) ? last(v) : v) for v in z]
anycause_parent =  [v ∈ first.(zparents) for v in anycause]

for v in z[.!anycause_parent]
    x = (isa(v, Array) ? [:Y, v...] : [:Y, v])
    println(join(string.(reverse(x)),"->"));
end
# B->T->Y
# P->T->Y
# B->T->M->Y
# P->T->M->Y

function inferedges(nodes; start=missing, finish=missing, rev=true)
  edgesout = fill([], length(nodes))
  for i in 1:length(nodes)
      nodeset = nodes[i]
      rev && isa(nodeset, Array) && (nodeset = reverse(nodeset))
      myset = []
      (start !== missing) && push!(myset, start)
      isa(nodeset, Array) && append!(myset, nodeset)
      !isa(nodeset, Array) && push!(myset, nodeset)
      (finish !== missing) && push!(myset, finish)

      edgesout[i] = [myset[i] => myset[i+1] for i in 1:(length(myset)-1)]
  end
  edgesout
end

zedges = inferedges(z, finish=:Y)
for v in zedges; println(v) ; end
# Any[:T => :Y]
# Any[:B => :T, :T => :Y]
# Any[:S => :T, :T => :Y]
# Any[:P => :T, :T => :Y]
# Any[:S => :P, :P => :T, :T => :Y]
# Any[:A => :P, :P => :T, :T => :Y]
# Any[:A => :T, :T => :Y]
# Any[:G => :Y]
# Any[:M => :Y]
# Any[:T => :M, :M => :Y]
# Any[:B => :T, :T => :M, :M => :Y]
# Any[:S => :T, :T => :M, :M => :Y]
# Any[:P => :T, :T => :M, :M => :Y]
# Any[:S => :P, :P => :T, :T => :M, :M => :Y]
# Any[:A => :P, :P => :T, :T => :M, :M => :Y]
# Any[:A => :T, :T => :M, :M => :Y]
# Any[:G => :M, :M => :Y]
# Any[:S => :M, :M => :Y]
# Any[:S => :Y]
# Any[:A => :Y]
```
