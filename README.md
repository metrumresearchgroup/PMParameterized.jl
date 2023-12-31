# PMParameterized

## Disclaimer: This is an initial release of packages presented in a workshop and poster at ACoP14. Still undergoing validation and qualification. 

PMParameterized is a package that builds upon [ModelingToolkit](https://docs.sciml.ai/ModelingToolkit/stable/) for quantitative systems pharmacology (QSP) and pharmacometrics (PM). 


### Quick Example
A brief example of a basic TDM-1 PK model is presented below. Please see our poster for more details.

```julia
# define model
mdl = @model mod begin
    @IVs t [unit = u"hr", description = "time", tspan = (0.0, 100.0)]
    D = Differential(t)
    @constants day_to_h = 24.0, [unit=u"hr/d", 
                                 description = "Convert days to hours"]
    @parameters begin 
        CL_ADC = 0.0043, [unit = u"L/d", 
                            description = "central clearance"]
        CLD_ADC = 0.014, [unit = u"L/d", 
                          description = "intercompartmental clearance"]
        V1_ADC = 0.034,       [unit = u"L", 
                               description = "central volume"]
        V2_ADC = 0.04,        [unit = u"L", 
                               description = "peripheral volume"]
    end
    CL_ADC = CL_ADC/day_to_h
    CLD_ADC = CLD_ADC/day_to_h
    @variables begin
        (X1_ADC(t) = 0.0), [unit = u"nmol", 
                                 description = "ADC amount in Compartment 1"]
        (X2_ADC(t) = 0.0), [unit = u"nmol", 
                                 description = "ADC amount in Compartment 2"]
    end
    @observed begin
        C_X1 = X1_ADC/V1_ADC
        C_X2 = X2_ADC/V2_ADC
    end
    @eq D(X1_ADC) ~ -(CL_ADC/V1_ADC)*X1_ADC - 
                        (CLD_ADC/V1_ADC)*X1_ADC + 
                        (CLD_ADC/V2_ADC)*X2_ADC
                        
    @eq D(X2_ADC) ~ (CLD_ADC/V1_ADC)*X1_ADC - 
                        (CLD_ADC/V2_ADC)*X2_ADC
end;
```

Solving the model

```julia
MW = 148781.0                                   # [g/mol] T-DM1 molecular weight (Scheuher et al 2022)
BW = 70                                         # [kg] human body weight
dose_in_mgkg = 3.6                              # mg/kg per Girish 2012 data
dose = (dose_in_mgkg * 1e-3 * BW) / (MW / 1e9)  # nmol
mdl.states.X1_ADC = dose
mdl.tspan = (0.0, 24*21)
sol = solve(mdl, Tsit5(), saveat=0.5);
plot(sol.t, sol.C_X1, label = "X1", xlabel="Time (hours)", ylabel="T-DM1 (nM)", dpi=600)
plot!(sol.t, sol.C_X2, label = "X2")
```
<p align="center">
<img src='images/tdm11.png' width='500'>
</p>


## Event Handling and Dosing
Please see our [PMSimulator](https://github.com/metrumresearchgroup/PMSimulator.jl) package for details
