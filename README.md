---
page_template: vega.html
...
:page_template: vega.html



# Conventional Calibration - Steel Frame

|  |  |
|----------|------|
| Problem files | [quo-07](https://github.com/claudioperez/SimCenterDocumentation/tree/examples/docs/common/user_manual/examples/desktop/quoFEM/quo-07) |

In this example, a parameter estimation routine is used to estimate column stiffnesses of a simple steel frame, given data about it's mode shapes and mass distribution.

Provided by Professor Joel Conte and his doctoral students Maitreya Kurumbhati and Mukesh Ramancha from UC San Diego, this example looks at the following simplified finite element model of a steel building.

![Schematic model of frame.](figures/frameFE.png){style="float:right" width="300" align="center"}

Each floor slab of the building is made of composite metal deck and is supported on four steel columns. The story heights are measured at $10'$ and in plan the side lengths measure $33'-4"$ by $30'$. Properties of the steel columns are taken deterministically  with an elastic modulus of $29,000 \mathrm{ksi}$, area of $110 \mathrm{in}^2$, and principal moment of inertial ($I_{yy}$) of $1190 \mathrm{ in}^4$. For modelling purposes, the four columns are assumed fixed at the base and the beams connecting them are assumed to be rigid. The first two vibration periods of the structure are determined to be $0.19 sec$ and $0.09 sec$. 

Using these properties, the following set of mode shapes and frequencies are generated by applying random perturbations to the analytic modal properties in order to simulate field data that might be obtained using a method for structural system identification. 

$$\begin{array}{l}
\lambda_{1}^{(1)}=1025.21, \lambda_{1}^{(2)}=1138.11, \quad \lambda_{1}^{(3)}=1099.39, \quad \lambda_{1}^{(4)}=1002.41, \quad \lambda_{1}^{(5)}=1052.69 \\
\phi_{12}^{(1)}=1.53, \quad \phi_{12}^{(2)}=1.24, \quad \phi_{12}^{(3)}=1.38, \quad \phi_{12}^{(4)}=1.50, \quad \phi_{12}^{(5)}=1.35
\end{array}$$

Our goal will be to reobtain the original column moments of inertia from this data using a conventional calibration procedure.



The unknown quantities of interest are the moments of inertia for the first and second story columns (`Ic1` and `Ic2` respectively), on which we impose the the following bounds and initial estimates:




1. First story column moment of inertia, `Ic1`: **ContinuousDesign** distribution with a  initial point $(X_0)$ of $1500.0$,  lower bound $(L_B)$ of $500.0$,  upper bound $(U_B)$ of $2000.0$, 

1. Second story column moment of inertia, `Ic2`: **ContinuousDesign** distribution with a  initial point $(X_0)$ of $500.0$,  lower bound $(L_B)$ of $500.0$,  upper bound $(U_B)$ of $2000.0$, 




## UQ Workflow


To define the uncertainty workflow in quoFEM, select **Parameters Estimation** for the **Dakota Method Category**, and enter the following inputs:



|   |   |
|---|---|
| **Convergence Tol** | 1e-10 |
| **Scaling Factors** |  |
| **Max No. Iterations** | 50 |
| **Method** | NL2SOL |



## Model Files


The following files make up the **FEM** model definition.


#. [Frame2FEM.tcl](src/Frame2FEM.tcl): This file is an OpenSees Tcl script that defines a FE model for a given realization, runs an analysis, and creates a `results.out` file. As a consequence, no postprocessing script is needed. The values placed in `results.out` file are the difference between computed and observed values. Expressed another way, the function `f(Ic1,Ic2)` computed and written to the `results.out` file is `f(Ic1,Ic2) = ObservedPeriods - ComputedPeriods(Ic1,Ic2)`. The UQ algorithm when running is searching for values of the random variable parameters (`Ic1` and `Ic2`) that minimize this loss function. The user must take this fact into account when formulating the output from their own scripts for their own problems.



<!-- <div class="admonition warning">Do not place the files in your root, downloads, or desktop folder as when the application runs it will copy the contents on the directories and subdirectories containing these files multiple times. If you are like us, your root, Downloads or Documents folders contains and awful lot of files and when the backend workflow runs you will slowly find you will run out of disk space!</div> -->

## Results

Once the analysis is complete the **RES** tab will be automatically selected and the results will be displayed as shown in the figure below. 

The resulting estimates for the column stiffnesses, `Ic1` and `Ic2` are **1168.83** and **1211.25** respectively.


<div id="vis"></div>
<script>
    // Assign the specification to a local variable vlSpec.
    var vlSpec = {
    $schema: 'https://vega.github.io/schema/vega-lite/v4.json',
    data: {
        values: [
        {a: 'C', b: 2},
        {a: 'C', b: 7},
        {a: 'C', b: 4},
        {a: 'D', b: 1},
        {a: 'D', b: 2},
        {a: 'D', b: 6},
        {a: 'E', b: 8},
        {a: 'E', b: 4},
        {a: 'E', b: 7}
        ]
    },
    mark: 'bar',
    encoding: {
        y: {field: 'a', type: 'nominal'},
        x: {
        aggregate: 'average',
        field: 'b',
        type: 'quantitative',
        axis: {
            title: 'Average of b'
        }
        }
    }
    };

    // Embed the visualization in the container with id `vis`
    vegaEmbed('#vis', vlSpec);
</script>
