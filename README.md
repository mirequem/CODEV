# CODEV
The codes in this repository are macros and complementary codes for use with the [*CODE V* optical design software](https://www.synopsys.com/optical-solutions/codev.html). They have been written for very specific needs that will be detailed below. 

## `bspax.seq`

### Overview

CODE V has a built-in tool, beam synthesis propagation (BSP), which performs wavefront propagation based on the Huygens-Fresnel principle. The algorithm calculates the diffraction pattern at the image plane, based on the propagation of a number of beamlets.

Using the software interface, the user is constrained to perform only one BSP at a time, with only one set of given parameters. This can be particularly constraining when analyzing non-diffracting beams, where obtaining the axial intensity profile is a key issue. 

In other words, the CODEV interface only allows to have the pattern of a section of the non-diffracting beam modeled. To obtain its axial profile, the BSP algorithm must be repeated subsequently over the entire desired axial length. 

The macro `bspax.seq` provides a solution to this problem. By executing the macro, the BSP calculation is automatically performed over a given axial distance (with a specified number of planes and other calculation parameters, as discussed below), generates a graph in the form of a 2D map and exports the data to a .dat file.

### Getting started

1. In CODE V software, choose a working directory (File > Working directory...). The `bspax.seq` macro must be saved in this folder. 

2. In the CODE V software, create an optical system generating a non-diffracting beam (or any other system of interest where an axial BSP is required). The non-diffracting beam must be between the second-to-last and the image surface.

3. Change the simulation parameters manually in the `bspax.seq` file.

- Change the general simulation parameters:
```
!Parameters
^a == 7		    !image surface number
^m == 200		!number of surfaces (axial)
^c == 200		!number of points (radial)
^f == 0.02		!radial width [mm]
^lo == 20		!axial distance[mm]
```

- Choose the beam type (OPU or Gaussian beam) by uncommenting the appropriate line (remove !):

```
!OPU: uncomment the two following lines
!spx 3.5		!x beam radius
!spy 3.5		!y beam radius
!Gaussian beam: uncomment the two following lines and place object at a finite distance (no infinity distance)
!wrx 3.5		  !x beam radius
!wry 3.5	  	!y beam radius
```

- Choose the NFR value, which is the number of rings to use in the initial fit. The number of beamlets in the initial fit is [3NRI(1+NRI) + 7] for NRI > 2. For NRI = 2, the number of beamlets is 25, and for NRI = 1, the number of beamlets is 9.
  
```
!choose NFR value
nri nfr 250		
```
- Choose the name of the data file:

```
!copy data into a .dat file
buf exp b4 filename
```

- Save the changes of the macro

4. In the CODE V software, in the command line, type `in bspax`. The sequence will run automatically. The software outputs a 2D map of the axial intensity profile and save the data into a .dat file.

## `usergrn.seq`

### Overview
This macro was written specifically for a master's project consisting of testing optical fiber preforms with a very particular radial refractive index profile. CODE V allows the design of components with special index profiles, but these functions are very complex and only exact equations can be used. In other words, it is impossible to use experimental refractive index data as a function of radial position and insert them as input into CODE V. The built-in macro `usergrn.seq` has been modified to achieve this goal.

Specifically, the macro allows two cases to be processed:

1. Select parameters for a radial refractive index profile in hyperbolic secant. This is the exact profile for making a GRIN-Axicon component. For more details about this comopsant and the meaning of the parameters, read the corresponding [scientific article](https://www.sciencedirect.com/science/article/abs/pii/S0030401820304545). 
2. Import refractive index data according to radial position. The software then interpolates between each data to make the curve continuous.

### Getting started
This macro is not very robust and easily generates bugs (still unresolved). The following steps must therefore be meticulously followed.

1. In the local repository where the built-in CODE V macro are saved, delete the original `usergrn.seq` and replace it by the modified one from Github (in my case, C:\CODEV112_SR1\macro). 

2. In the new `usergrn.seq` file, select the appropriate case by choosing the case number:
```
FCT @profil(num ^erx, num ^ery, num ^x, num ^y)
	^cas == 1   	!choose appropriate case
```
- If case 1 is chosen, specify the two GRIN-axicon design parameters (annular radius and alpha parameter):
```
if ^cas = 1			!hyperbolic secant model (exact)
	^a == 0.5		  !ring radius
	^b== 0.2346		!alpha parameter
```
- If case 2 is chosen, adjust the datapoints alignment in respect to the radial distance:
```
else if ^cas=2		!lab data of a real preform
	^r == sqrtf((^x+^erx) * (^x+^erx) + (^y+^ery) * (^y+^ery))
	^m == roundf(^r / 0.0007) + 5225		!radial and scale adjustment
	^temp ==((buf.num b5 i^m j2)+ 1.48) / 1.48
```
- Again, for the case 2, specify the dataset (must be a .dat file). This file must be a two-column vector which the first one represent the radial position and the second one is the refractive index data. This file must be saved in th working directory.
```
buf imp b5 filename.dat		!write file name of refractive index data file
```
3. Open CODE V and select the appropriate working directory (File > Working directory...)

4. For the first use, a special glass must be imported into the catalogue. Choose Lens > Add Private Catalog Glass > Gradient Index... Choose "User defined" as the glass type. ***To complete***.

5. In the `usergrn.seq`, choose case 1 and save the file. This step is very important because if the user choose case 2, the software will get into an infinite loop and a shut-down will be necessary. This bug needs to be fixed. Comment the line that imports the data from an expperimental index profile that is used in case 2 (line 2 of the code). 

6. Save the file grinax.seq in the working folder.

7. . In the CODE V command line, type:
``` 
in grinax
```
A refractive index graph will appear, which will probably be a simple line. 

8. In CODE V, open the .lens file in which the optical system is designed. The GRIN component must have the user-defined glass. 

9. In the command line, type again:
``` 
in grinax
```
The refractive index graph should appear again, but with the specified parameters of case 1. 

10. In the `usergrn.seq`, change to case 2 if needed. Type again in the command line `in grinax` and the refractive index graph will appear for the case 2 parameters. 
