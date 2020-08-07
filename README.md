# CODEV
The codes in this repository are macros and complementary codes (often written in MATLAB language) for use with the [*CODE V* optical design software](https://www.synopsys.com/optical-solutions/codev.html). They have been written for very specific needs that will be detailed below. 

## bspax.seq

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
^a == 7		  !image surface number
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
wrx 3.5		  !x beam radius
wry 3.5	  	!y beam radius
```

- Choose the NFR value, which is the number of rings to use in the initial fit. The number of beamlets in the initial fit is [3NRI(1+NRI) + 7] for NRI > 2. For NRI = 2, the number of beamlets is 25, and for NRI = 1, the number of beamlets is 9.
  
```
!choose NFR value
nri nfr 250		
```
- Choose the name of the data file:

```
!copy data into a .dat file
buf exp b4 fichier
```

- Save the changes of the macro

4. In the CODE V software, in the command line, type `in bspax`. The sequence will run automatically. The software outputs a 2D map of the axial intensity profile and save the data into a .dat file.

## usergrn.seq
