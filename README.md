## UnConditional Sequential Gaussian SIMulation (UCSGSIM)


<h3 align="center">An unconditional random field generation tools that are easy to use.</h3>


## Introduction of UCSGSIM
Sequential Gaussian Simulation is a random field generation method which was based on the kriging interporlation method. 

Unconditonal simulation don't follow the patterns of "data", but follow the users's settings like mean and variance.

**The core ideas of UCSGSIM are:**
1. Create the grid (no any data value exist now).

$$ \Omega\to R $$

2. Visit random point of the model (draw one random value of the x_grid) 

$$ X = RandomValue(\Omega),  X:\Omega\to R $$

3. Select the **theoritical covariance model** to use, and set the **sill** and **range** properly.

$$ Gaussian = (C_{0} - s)(1 - e^{-h^{2}/r^{2}a})$$

$$ Spherical = (C_{0} - s)(3h/2r - h^3/2r^3)$$

$$ Exponential = (C_{0} - s)(1 - e^{-h/ra})$$

4. If there have more than 1 data value closed to the visted point (depend on the **range** of covariance model), then go next step. Else draw the random value from normal distribution as the simulation results of this iteration.

$$ Z_{k}({X_{simulation}}) = RandomNormal(m = 0 ,\sigma^2 = Sill)$$

5. Calculate **weights** from the **data covaraince** and **distance coavariance**

$$ \sum_{j=1}^{n}\omega_{j} = C(X_{data}^{i},X_{data}^{i})C^{-1}(X_i,X_i), i=1...N $$

6. Calculate the **kriging estimate** from the **weight** and **data value**

$$ Z_{k}(X_{estimate}) = \sum_{i=1}^{n} \omega_{i} Z(X_{data}) + (1- \sum_{i=1}^{n} \omega_{i} m_{g}) $$

7. Calculate the **kriging error (kriging variance)** from **weights** and **data covariance** 

$$ \sigma_{krige}^{2} = \sum_{i=1}^{n}\omega_{i}C(X_{data}^{i},X_{data}^{i}) $$

8. Draw the random value from the normal distribution and add to the **kriging estimate**.

$$ Z(X_{simulation}) = Z(X_{estimate}) + RandomNormal(m = 0, \sigma^2 = \sigma_{krige}^{2}) $$

9. Repeat 2 ~ 8 until the whole model are simulated.

10. Repeat 1 ~ 9 with different **randomseed number** to produceed mutiple realizations.




## Features
* Python version (UC_SGSIM_py)
  * One dimensional unconditional randomfield generation with sequential gaussian simulation alogarithm
  * Enable to use muti-cores to run the simulation (mutiprocessing)
  * User can choose to use the computation kernel in python or C (DLL), C has a better computation efficiency.
  * User can combine the profits of python and c by using this package. Calculate in C(DLL) file, and plot the images by matplotlib package immediately.

* C version (UC_SGSIM_c)
  * One dimensional unconditional randomfield generation
  * Simple UI just text the input parameters and prees Enter.
  * Much better computation efficiency then python version.

## Example

**Python**

```py
import UC_SGSIM as UC
import numpy as np

if __name__ == '__main__':

    X_grid = range(150)      # Model X grid
    bw = 1                   # Lag steps (bandwidth) of Lag distance
    hs = np.arange(0, 35, 1) # Total lag distance (for variogram calculation)
    krige_range = 17.32      # effective range of covariance model
    krige_sill = 1           # Sill of covariance model
    n_realizations = 20      # number of realizations to calculate (numbers of random field to generate)

    Cov_model = UC.Guassian(hs, bw, krige_range, krige_sill)

    sgsim = UC.Simulation()     # Use python kernel to calculate
    sgsim_c = UC.Simulation_byC() # Use C kernel (dll) to calculate

    sgsim.compute_async(n_process = 4, randomseed = 12345)  # Use four process to do the parallel computing
    sgsim_c.compute_by_dll(n_process = 4, randomseed = 77875) # Use four process to do the parallel computing

    sgsim.MeanPlot("ALL")                 # Plot mean
    sgsim.VarPlot()                       # Plot variance 
    sgsim.Cdf_Plot(x_location=11)         # CDF at certain location
    sgsim.Hist_Plot(x_location=11)        # Hist at certain location
    sgsim.variogram_compute(n_process=8)  # Compute variogram before plotting
    sgsim.VarioPlot()                     # Plot Variogram of each realizations and it's mean value

    # Please note that the parameter "n_realizations" means the number of realizations calculate in each process, 
    # so this case will generate total 20 * 4(process) = 80 realizations

```

<p align="center">
   <img src="https://github.com/Zncl2222/Stochastic_SGSIM/blob/main/figure/Realizations.png"  width="40%"/>
   <img src="https://github.com/Zncl2222/Stochastic_SGSIM/blob/main/figure/Mean.png"  width="40%"/>
   <img src="https://github.com/Zncl2222/Stochastic_SGSIM/blob/main/figure/Variance.png"  width="40%"/>
   <img src="https://github.com/Zncl2222/Stochastic_SGSIM/blob/main/figure/Variogram.png"  width="50%"/>
   <img src="https://github.com/Zncl2222/Stochastic_SGSIM/blob/main/figure/HIST.png"  width="40%"/>
   <img src="https://github.com/Zncl2222/Stochastic_SGSIM/blob/main/figure/CDF.png"  width="50%"/>
</p>

## Future plans
* 2D unconditional randomfield generation
* GUI mode in pyhton package
* More covariance models
* More kriging methods (etc. Oridinary Kriging)
* Performance enhanced
* More completely documents and easy to use designs.

## Efficiency Comparison
<p align="center">
<img src="https://github.com/Zncl2222/Stochastic_SGSIM/blob/main/figure/C_Cpp_py_comparision.png"  width="70%"/>
</p>

```
Parameters for testing:

model len = 150

number of realizations = 1000

Range scale = 17.32

Variogram model = Gaussian model

---------------------------------------------------------------------------------------

Testing platform:

CPU: AMD Ryzen 9 4900 hs

RAM: DDR4 - 3200 40GB (Dual channel 16GB)

Disk: WD SN530
```
