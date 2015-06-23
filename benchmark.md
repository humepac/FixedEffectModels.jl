

## Comparison

Julia
```julia
using DataArrays, DataFrames, FixedEffectModels
N = 10000000
K = 100
df = DataFrame(
  v1 =  pool(rand(1:N/K, N)),
  v2 =  pool(rand(1:K, N)),
  v3 =  randn(N), 
  v4 =  randn(N),
  w =  abs(randn(N)) 
)
@time reg(v4 ~ v3, df)
# elapsed time: 1.22074119 seconds (1061288240 bytes allocated, 22.01% gc time)
@time reg(v4 ~ v3, df, weight = :w)
# elapsed time: 1.56727235 seconds (1240040272 bytes allocated, 15.59% gc time)
@time reg(v4 ~ v3, df, absorb = [:v1])
# elapsed time: 1.563452151 seconds (1269846952 bytes allocated, 17.99% gc time)
@time reg(v4 ~ v3, df, absorb = [:v1], weight = :w)
# elapsed time: 2.063922289 seconds (1448598696 bytes allocated, 17.96% gc time)
@time reg(v4 ~ v3, df, absorb = [:v1, :v2])
# elapsed time: 2.494780022 seconds (1283607248 bytes allocated, 18.87% gc time)
````

R (lfe package, C)
```R
library(lfe)
N = 10000000
K = 100
df = data.frame(
  v1 =  as.factor(sample(N/K, N, replace = TRUE)),
  v2 =  as.factor(sample(K, N, replace = TRUE)),
  v3 =  runif(N), 
  v4 =  runif(N), 
  w = abs(runif(N))
)
system.time(lm(v4 ~ v3, df))
#   user  system elapsed 
# 15.712   0.811  16.448 
system.time(lm(v4 ~ v3, df, w = w))
#   user  system elapsed 
# 10.416   0.995  11.474 
system.time(felm(v4 ~ v3|v1, df))
#   user  system elapsed 
# 19.971   1.595  22.112 
system.time(felm(v4 ~ v3|v1, df))
#   user  system elapsed 
# 19.971   1.595  22.112 
system.time(felm(v4 ~ v3|v1, df, w = w))
#   user  system elapsed 
# 19.971   1.595  22.112 
system.time(felm(v4 ~ v3|(v1+v2), df))
#   user  system elapsed 
# 23.980   1.950  24.942 
```



Stata
```
clear all
local N = 10000000
local K = 100
set obs `N'
gen  v1 =  floor(runiform() * (`_N'+1)/`K')
gen  v2 =  floor(runiform() * (`K'+1))
gen  v3 =  runiform()
gen  v4 =  runiform()
gen  w =  abs(runiform()) 

timer clear

timer on 1
reg v4 v3
timer off 1

timer on 1
reg v4 v3 [w = w]
timer off 1

timer on 2
areg v4 v3, a(v1)
timer off 2

timer on 3
areg v4 v3 [w = weight], a(v1)
timer off 3

timer on 4
reghdfe v4 v3, a(v1 v2)
timer off 4

. timer list
   1:      1.56 /        1 =       1.5570
   2:      4.60 /        1 =       4.6020
   3:     68.34 /        1 =      68.3370
````