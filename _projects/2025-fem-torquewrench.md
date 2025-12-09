---
layout: project
title: Torque Wrench FEM Analysis
description: Advanced FEM Project
technologies: [Autodesk Fusion, MATLAB, Ansys]
---



In this project for my MAE 3270 Materials class, I was tasked with using beam theory analysis and FEA to select the dimensions 
and material of a torque wrench to meet a set of design constraints. I began with doing hand calculations to relate parameters to the constraints 
and then created a MATLAB script to check my design iterations. The final code is as listed below:
```matlab
M = 600; % max torque (in-lbf)
L = 16; % length from drive to where load applied (inches)
h = .6; % width
b = 0.5; % thickness
c = 1.0; % distance from center of drive to center of strain gauge
E = (16.3+16.7)/2*10^6; % Young's modulus (psi)
nu = (0.332+.349)/2; % Poisson's ratio
su = (114+130)/2*10^3; % tensile strength use yield or ultimate depending on material (psi)
KIC = (94.2+104)/2*10^3; % fracture toughness (psi sqrt(in))
sfatigue = (88.9+116)/2*10^3; % fatigue strength from Granta for 10^6 cycles
name = 'Titanium, alpha-beta alloy, Ti-6Al-4V, cast'; % material name
%bend stress
bendstress = 6*M/(b*h^2);  
fprintf('Max normal stress = %.2f ksi\n', bendstress/1000)
%strain
strain = bendstress / E;             
strain_micro = strain * 1e6;% microstrain
fprintf('Strain at gauge = %.0f microstrain\n', strain_micro)
%deflection
deflection = (4*M*L^2)/(E*b*h^3);
fprintf('Load point deflection = %.4f in\n', deflection)
%factors of safety
SF_strength = su / (bendstress);
if SF_strength >= 4
   fprintf('Yield/Brittle FoS constraint: PASS (%.2f ≥ 4)\n', SF_strength);
else
   fprintf('Yield/Brittle FoS constraint: FAIL (%.2f < 4)\n', SF_strength);
end
a = 0.04;
ki = 1.12*bendstress*sqrt(pi*a);
SF_crack = KIC / (ki);
if SF_crack >= 2
   fprintf('Crack Growth FoS constraint: PASS (%.2f ≥ 2)\n', SF_crack);
else
   fprintf('Crack Growth FoS constraint: FAIL (%.2f < 2)\n', SF_crack);
end
SF_fatigue = sfatigue / (bendstress);
if SF_fatigue >= 1.5
   fprintf('Fatigue FoS constraint: PASS (%.2f ≥ 1.5)\n', SF_fatigue);
else
   fprintf('Fatigue FoS constraint: FAIL (%.2f < 1.5)\n', SF_fatigue);
end
%voltage output
k = 2.0;   
Vratio = (k * strain)/2;      % V/V
Vout_mV = Vratio*1000;          % mV/V
if Vout_mV >= 1
   fprintf('Output voltage constraint: PASS (%.2f ≥ 1 mV/V)\n', Vout_mV);
else
   fprintf('Output voltage constraint: FAIL (%.2f < 1 mV/V)\n', Vout_mV);
end
```
The results for my design are below: 

Max normal stress = 20.00 ksi
Strain at gauge = 1212 microstrain
Load point deflection = 0.3448 in
Yield/Brittle FoS constraint: PASS (6.10 ≥ 4)
Crack Growth FoS constraint: PASS (12.48 ≥ 2)
Fatigue FoS constraint: PASS (5.12 ≥ 1.5)
Output voltage constraint: PASS (1.21 ≥ 1 mV/V)

For my design, we selected Ti-6Al-4V because its high yield strength allows the wrench to avoid permanent deformation and failure 
under peak torque while its elastic modulus is relatively low so that it can be slightly flexible and allow higher-sensitivity strain-gauge 
measurement. The material also offers strong fatigue resistance to support several loading cycles and high fracture toughness to slow crack 
growth from small flaws. 


After I had checked that all constraints were fulfilled, I created my design in Fusion.

/assets/images/mae3270pic1.png

/assets/images/mae3270pic2.png

/assets/images/mae3270pic3.png

/assets/images/mae3270pic4.png

For the FEM model, we first applied the displacement condition (Yellow) on the drive, telling the software that it does not move. We set the displacement to (0, 0, 0). Then we applied the force at the edge of the beam (red face) with a force of 37.5 lbf in the y direction, which allowed the software to determine the bending of the beam. 

/assets/images/mae3270pic5.png

Next, we defined to the software what kind of material we were using by giving it the Young’s modulus and the Poisson's ratio. Giving the software the necessary components to calculate the displacement, stress, and strain of the beam.

/assets/images/mae3270pic6.png

Normal strain contours (in the strain gauge direction) from FEM

/assets/images/mae3270pic7.png


Contour plot of maximum principal stress from FEM

/assets/images/mae3270pic8.png

(Used max stress under clamped boundary line since indicated max is likely artificial)

/assets/images/mae3270pic9.png

We can see at the base of the drive that the maximum and minimum normal stress occur (indicated max/min stresses at the clamp-handle
intersection are assumed to be artificial), with the largest magnitude value being 31,727 psi in tension. Using this value gives a safety
factor against yielding of 3.8 which is slightly lower than the required safety factor against yielding of 4. The hand calculations 
predicted that the wrench would have a significantly larger factor of safety with the selected material and dimensions based on beam theory.

/assets/images/mae3270pic10.png

The load point deflection from the FEM is 0.40865in which is only slightly higher than the hand-calculated displacement of 0.3448 in.

/assets/images/mae3270pic11.png

The strain at the location of the strain gauge has a magnitude of 1136.6 microstrain or 0.0011366 strain. The hand-calculated strain gauge strain was 1212 microstrain which is very close.


Torque wrench sensitivity in mV/V using strains from the FEM analysis
Based on the FEM strain gauge strain of 0.0011366 and the hand calculations, the torque wrench sensitivity is 0.0011366 mV/V, 
which meets the minimum sensitivity requirement of 0.001 mV/V.

Strain Gauge Selection
From the DwyerOmega catalog, we found that gauge “SGD-3/120-LY11” works best with our design. The type is 
described as “3 mm Grid Length, 1.5 mm Grid Width 120 Î© Resistance, ST STC Number”. Since we want a gauge on both 
sides of the wrench where tension and compression are happening, we need the gauge length to be less than the wrench length
and the gauge width to be less than the wrench thickness. The carrier length of 7.8mm is significantly smaller than
the length of the wrench and the carrier width of 3.8mm is smaller than the wrench thickness (0.5 in or 12.7 mm), 
therefore, this gauge fits well within the wrench. The thickness of the gauges (3.15 mm) is also much smaller than 
the width of the wrench (0.6 in or 15.24 mm) so the gauges will definitely have enough space.
