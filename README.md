# Simulation-And-Reconstruction-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT

These tutorials are developed and maintained by Auer Benjamin from the Brigham and Women's Hospital and Harvard Medical School, Boston, MA, USA, and Pells Sophia from the University of Massachussets Chan Medical School, Worcester, MA, USA.

**Contact:** Auer, Benjamin Ph.D <bauer@bwh.harvard.edu>
Pells, Sophia Ph.D. <sophia.pells@umassmed.edu>

Table of contents:
```diff
- 1. Objective
- 2. Philips BrightView system specification
- 3. Simulating the BrightView system equiped with LEHR, LEHR-VXHR, MEGP and HEGP collimator in GATE
- 4. Simulating the BrightView system equiped with Single-Pinhole collimator in GATE
- 5. Simulating bone imaging with the BrightView system in GATE
- 6. Simulating brain perfusion and DaT imaging with the BrightView system in GATE
- 7. Simulating glioblastoma imaging with the BrightView system in GATE
- 8. Simulating brain perfusion and DaT imaging with the BrightView system incorporating the STL-based mesh50 attenuation phantom in GATE
- 9. Simulating a Lu-177 DOTATATE post-therapy SPECT with the BrightView system in GATE
```

## 1. Objective
In this tutorial, we offer a step-by-step walk through on how to build a realistic SPECT/scintigraphy simulation in [GATE source page](http://www.opengatecollaboration.org).  The Philips BrightView system with LEHR-VXHR, LEHR, MEGP, and HEGP parrallel-hole collimators and single-pinhole collimator. 

## 2. Philips BrightView system specification

The simulated Philips BrightView system models described here were adapted from one of previous projects where we aimed to evaluate the impact of downscatter contamination in I-123 SPECT imaging. The system model was validated against experimental data. More information can be found on the following article.
> - Könik A, Auer B, De Beenhouwer J, et al. (2019). [Primary, scatter, and penetration characterizations of parallel-hole and pinhole collimators for I-123 SPECT](https://iopscience.iop.org/article/10.1088/1361-6560/ab58fe/meta), Physics in Medicine & Biology, 64(24), 245001.

The collimator specifications adapted from the manufacturer's specification can be found in the table below,

| |    LEHR        |     LEHR-VXHR    |     MEGP   |     HEGP   |     Single Pinhole   |
|---------|----------|---------|-----------|------------------|----------------|
| Hole size/diameter (mm)           | 1.22 | 2.03 | 3.40 | 3.81 | 5.00 |
| Collimator thickness / Bore length (cm)| 2.70 | 5.4 | 5.84 | 5.84  |  4.15|  
| Septal thickness (mm)    | 0.152 | 0.152 | 0.86 | 1.73 | N/A |    

<p align="center">
<img width="277" alt="Screen Shot 2023-06-21 at 3 51 33 PM" src="https://github.com/BenAuer2021/Simulation-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/assets/84809217/367a83e8-999e-47eb-a075-0ce890336acf">
</p>

The detector consists of a 9.5-mm thick NaI(Tl) inorganic scintillator located at 3.5, 6.2, and 6.64 mm behind the surface of the collimator for the LEHR, LEHR-VXHR, MEGP/HEGP, respectively. The crystal dimension is 540 (transaxial) by 400 (axial) mm<sup>2</sup>.

## 3. GATE Simulation of the Philips BrightView SPECT System

We described below in details the modelling of BrightView SPECT System equipped with the MEGP collimator. 

### Define the world volume
The first step is to define the world and set the path to the file which defines all materials in the simulation:
 
 ```ruby 
############################################################## 
# Materials definition
/gate/geometry/setMaterialDatabase /PathTo/GateMaterials.db

############################### WORLD ################################
/gate/world/geometry/setXLength 150. cm 
/gate/world/geometry/setYLength 150. cm 
/gate/world/geometry/setZLength 260. cm 
/gate/world/vis/forceWireframe
/gate/world/vis/setVisible 1
/gate/world/vis/setColor green
/gate/world/setMaterial Air
```
The world defines the experimental framework of the simulation. It must be large enough to contain the entire SPECT system and phantom. The tracking of particles is stopped once they leave the `world`. The `world` is shown as a green wire frame in the visualization of the simulation shown in Figure 1. `GateMaterials.db` must contain all elemental compositions and densities of materials which are defined later in the macro. 

![Vis_BV_LuPatient](https://github.com/BenAuer2021/Simulation-And-Reconstruction-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/assets/55833314/0995922f-eeda-4069-9f25-926ca7a8e76c)
<br /> Figure 1: Visualisation of the simulation discussed in this Section. 

### Define the system geometry

Next we must define our detector and its components. For SPECT, we must define a volume of type `SPECThead`. This volume will contain all components of the SPECT system and can even include the phantom.

```ruby
#####  SPECTHead: ######
/gate/world/daughters/name SPECThead
/gate/world/daughters/insert box
/gate/SPECThead/geometry/setXLength 37.5 cm
/gate/SPECThead/geometry/setYLength 64 cm
/gate/SPECThead/geometry/setZLength 53 cm
/gate/SPECThead/placement/setTranslation -45.0 0  0 cm  # Move heads to allow source to be at 0 cm
/gate/SPECThead/setMaterial Air
/gate/SPECThead/vis/setColor cyan
```
The `SPECThead` volume **must** be large enough to include all components of the system (including collimators or shielding) and can include the phantom as well to enable close contouring. However, the `SPECThead` volume should not cover any part of the source. Any shielding material outside the `SPECThead` volume will be ignored in the simulation (i.e. photons will pass straight through). The `SPECThead` is shown as a cyan wire frame in Figure 1. Visualization commands e.g. `/vis/setColor` can be set for all volumes, but visualization will only occur when a viewer is specified (see Visualization section below). 

The positioning of the SPECT head will depend on the acquisition and the phantom that is used. The hierarchical structure of GATE means that any phantom volume will overwrite any SPECThead volume in the same position, this includes any air around the corners of a voxelised phantom. 

All detector components must be defined relative to the **center** of the `SPECThead` volume. Any translation set in `/gate/SPECThead/placement/setTranslation` will be applied to all components. Therefore, `/gate/SPECThead/placement/setTranslation` is a good way to set the radius of the detector during the acquisition. This is the radius to the center of the `SPECThead` volume, not to the front collimator or touch plate.

In the following examples, a radius of 45 cm for the Lu-177 Dotatate example and 40.75 cm for the Jaszczak phantom example was set to avoid overlap between the phantom volume and system components. To obtain the distance from the center of rotation to the collimator surface we need to substract half of the SPECTHead dimension along X. This results in radius of rotation of 26.25 cm and 22 cm for the Lu-177 Dotatate and the  Jaszczak phantom example, respectively.

Using multiple repetitions  of the `SPECThead` (`/gate/SPECThead/ring/setRepeatNumber` below)  and running  the visualization can be useful to check for overlap at all rotation angles as it can be seen in the figure below:

![Vis_BV_LuPatient_TestOverlap](https://github.com/BenAuer2021/Simulation-And-Reconstruction-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/assets/55833314/79ec0c40-9a79-4654-9a50-7dcc04983fc7)

Now we define the components within the `SPECThead`. We will start with the NaI(Tl) crystal and its cover.

```ruby 
#### Aluminium casing ####
/gate/SPECThead/daughters/name Al_casing
/gate/SPECThead/daughters/insert box
/gate/Al_casing/setMaterial Aluminium
/gate/Al_casing/geometry/setXLength 19.205 cm
/gate/Al_casing/geometry/setYLength 64.0 cm
/gate/Al_casing/geometry/setZLength 53.0 cm
/gate/Al_casing/placement/setTranslation 3.1075 0.0 0.0 cm # relative to center of SPECTHead (Mother Volume)
/gate/Al_casing/vis/setColor magenta
/gate/Al_casing/vis/forceWireframe
/gate/Al_casing/attachPhantomSD

# Lead casing
/gate/Al_casing/daughters/name lead_casing
/gate/Al_casing/daughters/insert box
/gate/lead_casing/setMaterial Lead
/gate/lead_casing/geometry/setXLength 16.505 cm
/gate/lead_casing/geometry/setYLength 58.6 cm
/gate/lead_casing/geometry/setZLength 47.6 cm
/gate/lead_casing/placement/setTranslation 1.35 0.0 0.0 cm # relative to center of Al_casing
/gate/lead_casing/vis/setColor grey
/gate/lead_casing/vis/forceWireframe
/gate/lead_casing/attachPhantomSD

# Crystal cover
/gate/lead_casing/daughters/name cry_cover
/gate/lead_casing/daughters/insert box
/gate/cry_cover/setMaterial Lead
/gate/cry_cover/geometry/setXLength 1.5525 cm
/gate/cry_cover/geometry/setYLength 54.0 cm
/gate/cry_cover/geometry/setZLength 40.0 cm
/gate/cry_cover/placement/setTranslation 7.47625 0.0 0.0 cm # relative to center of lead_casing
/gate/cry_cover/vis/setColor yellow
/gate/cry_cover/attachPhantomSD

# Air Gap
/gate/cry_cover/daughters/name Air_gap
/gate/cry_cover/daughters/insert box
/gate/Air_gap/setMaterial Air
/gate/Air_gap/geometry/setXLength 0.5 cm
/gate/Air_gap/geometry/setYLength 54.0 cm
/gate/Air_gap/geometry/setZLength 40.0 cm
/gate/Air_gap/placement/setTranslation 0.52625 0.0 0.0 cm # relative to center of cry_cover
/gate/Air_gap/vis/setColor red
/gate/Air_gap/vis/forceSolid

# Aluminium box
/gate/cry_cover/daughters/name Al_sheet
/gate/cry_cover/daughters/insert box
/gate/Al_sheet/setMaterial Aluminium
/gate/Al_sheet/geometry/setXLength 0.1 cm
/gate/Al_sheet/geometry/setYLength 54.0 cm
/gate/Al_sheet/geometry/setZLength 40.0 cm
/gate/Al_sheet/placement/setTranslation 0.22625 0.0 0.0 cm # relative to center of cry_cover
/gate/Al_sheet/vis/setColor white
/gate/Al_sheet/vis/forceWireframe
/gate/Al_sheet/attachPhantomSD

# Crystal
/gate/cry_cover/daughters/name crystal
/gate/cry_cover/daughters/insert box
/gate/crystal/setMaterial NaI
/gate/crystal/geometry/setXLength 0.9525 cm
/gate/crystal/geometry/setYLength 54.0 cm
/gate/crystal/geometry/setZLength 40.0 cm
/gate/crystal/placement/setTranslation -0.3 0.0 0.0 cm # relative to center of cry_cover
/gate/crystal/vis/setColor yellow
/gate/crystal/vis/forceSolid
```
Next, we will define the collimator. These will be the MEGP collimators of the Philips BrightView described above. This is defined as a block of lead with an array of hexagonal holes. First we define the lead block,

```ruby 
/gate/SPECThead/daughters/name collimator
/gate/SPECThead/daughters/insert box
/gate/collimator/geometry/setXLength 53.0 cm   
/gate/collimator/geometry/setYLength 64.0 cm 
/gate/collimator/geometry/setZLength 5.84 cm # Collimator thickness / Bore length 
/gate/collimator/placement/alignToX
/gate/collimator/placement/setTranslation 15.63 0.0 0.0 cm 
/gate/collimator/setMaterial CollLead
/gate/collimator/vis/setColor grey
/gate/collimator/vis/forceSolid
/gate/collimator/attachPhantomSD
```
`alignToX` reorients the block along the x-axis (such that the patient bed is on the z axis). It is similar to perform a 2D 90-degree rotation along the X-axis (`setRotationAxis 1 0 0` / `setRotationAngle 90 deg`).

Now we insert a hexagonal hole of air into the collimator block and repeat it to create the array. Note that we have to rotate the hole by 90 degrees to orientate it correctly with the block. 

```ruby
# Insert the first hole of air in the collimator
/gate/collimator/daughters/name hole
/gate/collimator/daughters/insert hexagone
/gate/hole/geometry/setHeight 5.84 cm # bore length
/gate/hole/geometry/setRadius 1.7 mm
/gate/hole/placement/setRotationAxis 0 0 1
/gate/hole/placement/setRotationAngle 90 deg
/gate/hole/setMaterial Air
/gate/hole/vis/setColor white

# Repeat the hole in a cubic array
/gate/hole/repeaters/insert cubicArray
/gate/hole/cubicArray/setRepeatNumberX 93
/gate/hole/cubicArray/setRepeatNumberY 73 
/gate/hole/cubicArray/setRepeatNumberZ 1
/gate/hole/cubicArray/setRepeatVector  4.26 7.3785 0.0 mm 

# Now shift the hole and repeat these holes in a linear array
/gate/hole/repeaters/insert linear
/gate/hole/linear/setRepeatNumber 2
/gate/hole/linear/setRepeatVector 2.13 3.6893 0.0 mm
```

The figure below shows a close-up of the initial hexagonal hole, the hexagonal array after the first repeater and then after the second. 
![coll_hole_array](https://github.com/BenAuer2021/Simulation-And-Reconstruction-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/assets/55833314/09f44bf2-6f94-4138-b0d1-8474dc31d173)

Other detector components, e.g. light guide, PMTs, and electronics, are defined in the same way. For modeling the compartments behind the crystal (i.e., back compartment), we used the “intermediate model” described in the paper cited above. The back-compartment is defined via multiple layers and PMTs by a box filled with a mixture of materials as follow,

```ruby
# Back Material
/gate/lead_casing/daughters/name back_material
/gate/lead_casing/daughters/insert box
/gate/back_material/setMaterial Lead
/gate/back_material/geometry/setXLength 13.1525 cm
/gate/back_material/geometry/setYLength 55.0 cm
/gate/back_material/geometry/setZLength 44.0 cm
/gate/back_material/placement/setTranslation 0.12375 0.0 0.0 cm
/gate/back_material/attachPhantomSD
/gate/back_material/vis/forceSolid

# Light Guide
/gate/back_material/daughters/name Light_Guide
/gate/back_material/daughters/insert box
/gate/Light_Guide/setMaterial Glass
/gate/Light_Guide/geometry/setXLength 0.9525 cm
/gate/Light_Guide/geometry/setYLength 54.0 cm
/gate/Light_Guide/geometry/setZLength 40.0 cm
/gate/Light_Guide/placement/setTranslation 6.1 0.0 0.0 cm
/gate/Light_Guide/vis/setColor white
/gate/Light_Guide/vis/forceWireframe
/gate/Light_Guide/attachPhantomSD
/gate/Light_Guide/vis/forceSolid

# Photo-muliplier tubes
/gate/back_material/daughters/name PMTs
/gate/back_material/daughters/insert box
/gate/PMTs/setMaterial PMTs
/gate/PMTs/geometry/setXLength 11.7 cm
/gate/PMTs/geometry/setYLength 54.0 cm
/gate/PMTs/geometry/setZLength 40.0 cm
/gate/PMTs/placement/setTranslation -0.22625 0.0 0.0 cm
/gate/PMTs/vis/setColor magenta
/gate/PMTs/vis/forceWireframe
/gate/PMTs/attachPhantomSD
/gate/PMTs/vis/forceSolid

# Electronics
/gate/back_material/daughters/name electronics
/gate/back_material/daughters/insert box
/gate/electronics/setMaterial ElectronicBoard
/gate/electronics/geometry/setXLength 0.5 cm
/gate/electronics/geometry/setYLength 54.0 cm
/gate/electronics/geometry/setZLength 40.0 cm
/gate/electronics/placement/setTranslation -6.32625 0.0 0.0 cm
/gate/electronics/vis/setColor green
/gate/electronics/vis/forceWireframe
/gate/electronics/attachPhantomSD
/gate/electronics/vis/forceSolid
```
After the components of the `SPECThead` have been defined, we can repeat the detector for dual- (or more) head acquisition. 
In this case we create two SPECTheads in H-mode configuration (180 degrees apart along the Z-axis). Note, positioning the heads in L-mode (90-degree apart) is also feasible. 

```ruby
/gate/SPECThead/repeaters/insert ring
/gate/SPECThead/ring/setRepeatNumber 2
/gate/SPECThead/ring/setPoint1 0.0 0.0 0.0 cm
/gate/SPECThead/ring/setPoint2 0.0 0.0 1.0 cm
```

We now set the orbit speed for SPECT acquisitions. Here we consider 120 projections/angular steps over 360 degree with an acquisition time of 1 second per projection. This results in 60 views per head. The orbitting speed here must correspond to the frame time slice set later (see Running the Simulation). Here we set the orbitting about the z axis (where our patient bed will be). 
```ruby
/gate/SPECThead/moves/insert orbiting
/gate/SPECThead/orbiting/setSpeed 3.0 deg/s
/gate/SPECThead/orbiting/setPoint1 0. 0. 0. cm
/gate/SPECThead/orbiting/setPoint2 0. 0. 1. cm
```
To consider 32 projections per head with an acquisition time of 40 seconds per projection. The lines above must be modified as follow,

```ruby
/gate/SPECThead/moves/insert orbiting
# Want each head to move 180 degrees in 32 projections
# 5.625 deg/frame movement.
# Setting 40 seconds per frame so 0.140625 deg/s
/gate/SPECThead/orbiting/setSpeed 0.140625 deg/s
/gate/SPECThead/orbiting/setPoint1 0. 0. 0. cm
/gate/SPECThead/orbiting/setPoint2 0. 0. 1. cm
```

### Attenuation phantom definition

Now we define the attenuation phantom. Here the attenuation phantom consists of a Jaszczak voxelized phantom used in SPECT quality control procedures. It is defined in interfile format (unsigned integer 16 bits) at the center of the world` volume.
We provide the phantoms and GATE macros for the Jaszczak phantom here https://github.com/BenAuer2021/Simulation-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/blob/main/GATE_MACROS_%20Jaszczak.zip. We also provide a phantoms and macros of an actual sup>177</sup>Lu-DOTATATE patient, available here https://github.com/BenAuer2021/Phantoms-For-Nuclear-Medicine-Imaging-Simulation/blob/main/patient_LuDOTATATE_phantoms.zip

```ruby
/gate/world/daughters/name voxelPhantom
/gate/world/daughters/insert ImageNestedParametrisedVolume 
/gate/voxelPhantom/geometry/setImage ./PathTo/Attenuation_Jas.h33
/gate/voxelPhantom/geometry/setRangeToMaterialFile ./PathTo/Attenuation_Jas_Range.dat
/gate/voxelPhantom/placement/setTranslation  0. 0. 0. cm
/gate/voxelPhantom/attachPhantomSD
```
The file `Attenuation_Jas_Range.dat` can be used to assign a given material (defined in the 'GateMaterials.db' file) to each voxel of the image. The materials in this file must correspond to ones defined a file set with the `/gate/geometry/setMaterialDatabase` command. See https://github.com/BenAuer2021/Phantoms-For-Nuclear-Medicine-Imaging-Simulation#readme for the materials used in this image. Be sure to include `/gate/voxelPhantom/attachPhantomSD` so that all scatter interactions within the phantom are recorded. The number of Compton and Rayleigh events will be recorded throughout all volumes assigned `attachPhantomSD` for any photons absorbed in the crystal. 

We kindly refer the users to https://github.com/BenAuer2021/Phantoms-For-Nuclear-Medicine-Imaging-Simulation#readme for a list of digital phantoms available for simulation. The attenuation phantom can also be defined as a series of STL-based objects, we provide several STL-based phantoms [here](https://github.com/BenAuer2021/Mesh-based-Human-Phantom-for-Simulation/edit/main/README.md). This approach has the advantage to enable closer object contouring as the large air volume surrounding any voxelized phantom is not present.

### Sensitive detectors
We must now set our crystal as a sensitive volume, so that photon interactions within it will be stored as _hits_ and processed according to the Digitizer set below which role is to emulate realistic detection process. 

```ruby
######################## SENSITIVE DETECTORS #########################
/gate/systems/SPECThead/crystal/attach crystal
/gate/systems/SPECThead/describe
/gate/crystal/attachCrystalSD
```
The line `/gate/crystal/attachCrystalSD` sets this crystal as a _sensitive detector_ which means that _hits_ in this volume are recorded (see the Digitizer section). 

### Digitizer 

The digitizer modules mimic the response of the detector and electronic signal chain in the physical system. The energy of all photons absorbed in the crystal volume will be recorded as an exact value, so digitiser modules must be used to simulate the finite resolution of the physical detectors. 

For SPECT, digitizers for energy and spatial blurring should be added, based on the specific system being modelled. Here, an energy blurring of 10% at 159 keV and an intrinsic spatial resolution of 3 mm are set. 

```ruby
############################# Digitizer ##############################
/gate/digitizer/Singles/insert adder
/gate/digitizer/Singles/insert blurring

#### Linear energy resolution
/gate/digitizer/Singles/blurring/setLaw linear
/gate/digitizer/Singles/blurring/linear/setResolution 0.10
/gate/digitizer/Singles/blurring/linear/setEnergyOfReference 159.0 keV

/gate/digitizer/Singles/insert spblurring
/gate/digitizer/Singles/spblurring/setSpresolution 3.0 mm       
```
Other Digitizer modules can be set for e.g. deadtime and thresholding.  Energy windows can also be set to be used for projection output later. For example for I-123 with a 15% energy window,
```ruby
/gate/digitizer/name Photopeak
/gate/digitizer/insert singleChain
/gate/digitizer/Photopeak/setInputName Singles
/gate/digitizer/Photopeak/insert thresholder
/gate/digitizer/Photopeak/thresholder/setThreshold 143.0 keV
/gate/digitizer/Photopeak/insert upholder
/gate/digitizer/Photopeak/upholder/setUphold 175.0 keV
```

### Physics, cuts and initialization

Geant4 containes a library of pre-built physics list which describe all processes for photon and particle interaction with material. This can be set with e.g.
```/gate/physics/addPhysicsList emstandard_opt4```

There appears to be an issue with scatter recording when atomic de-excitation is included. Therefore, for simulations where accurate scatter information is required, I would recommend to turn off atomic de-excitation with `/process/em/pixe false` with the physics definition and then `/process/em/fluo 0` after the simulation has been initialised. 

We also define cuts which determine a threshold below which no secondary particles will be generated. Each volume can be given specific cuts, or else it will inheret from its parent volume. The default cuts for the world are 1 mm. 

``` ruby
##############################  C U T S ##############################
/gate/physics/Gamma/SetCutInRegion      SPECThead 0.1 cm
/gate/physics/Electron/SetCutInRegion   SPECThead 0.1 cm
```
The initialisation step must be performed **after** the geometry, phantom and digitizer is set and **before** the definition of the source and root output:

```
/gate/run/initialize
/gate/physics/processList Initialized
```
### Source definition

The source image `Activity_Jas.h33` allows simulation of a Jaszczak phantom filled with a uniform concentration of I-123. The gamma emission data is from the IAEA database: https://www-nds.iaea.org/relnsd/vcharthtml/VChartHTML.html.
Note that this example shows the gamma emissions only, so the simulation output will be missing the X-ray peaks seen in true <sup>123</sup>I spectra. See https://github.com/BenAuer2021/Phantoms-For-Nuclear-Medicine-Imaging-Simulation#readme for more information on this source and other phantoms that can be used for simulation.

```ruby
#  Voxelised gamma source of 123I
/gate/source/addSource VS_gamma voxel
/gate/source/VS_gamma/reader/insert image
/gate/source/VS_gamma/imageReader/translator/insert linear
/gate/source/VS_gamma/imageReader/linearTranslator/setScale 0.0000003 Bq
/gate/source/VS_gamma/imageReader/readFile Activity_Jas.h33
/gate/source/VS_gamma/imageReader/verbose 1
/gate/source/VS_gamma/gps/particle gamma
/gate/source/VS_gamma/gps/ang/type iso
/gate/source/VS_gamma/gps/ang/mintheta 0.0 deg
/gate/source/VS_gamma/gps/ang/maxtheta 180.0 deg
/gate/source/VS_gamma/gps/ang/minphi 0.0  deg
/gate/source/VS_gamma/gps/ang/maxphi 360.0 deg

# Force unstable
/gate/source/VS_gamma/setForcedUnstableFlag  true
# Half life is 0.550958 days
/gate/source/VS_gamma/setForcedHalfLife 47602.8 s
/gate/source/VS_gamma/gps/particle    gamma
/gate/source/VS_gamma/gps/energytype  User
/gate/source/VS_gamma/gps/histname    energy
/gate/source/VS_gamma/gps/emin  0.150 MeV
/gate/source/VS_gamma/gps/emax  1.03663 MeV

# ---------------------------------------------------- #
/gate/source/VS_gamma/gps/histpoint 0.1589999 0.0
/gate/source/VS_gamma/gps/histpoint 0.159 83.6
/gate/source/VS_gamma/gps/histpoint 0.1590001 0.0

/gate/source/VS_gamma/gps/histpoint 0.17419999 0.0
/gate/source/VS_gamma/gps/histpoint 0.1742 0.0008
/gate/source/VS_gamma/gps/histpoint 0.1742001 0.0

/gate/source/VS_gamma/gps/histpoint 0.18261999 0.0
/gate/source/VS_gamma/gps/histpoint 0.18262 0.013
/gate/source/VS_gamma/gps/histpoint 0.18262001 0.0

/gate/source/VS_gamma/gps/histpoint 0.19069999 0.0
/gate/source/VS_gamma/gps/histpoint 0.1907 0.0005
/gate/source/VS_gamma/gps/histpoint 0.19070001 0.0

/gate/source/VS_gamma/gps/histpoint 0.19217999 0.0
/gate/source/VS_gamma/gps/histpoint 0.19218 0.0177
/gate/source/VS_gamma/gps/histpoint 0.19218001 0.0

/gate/source/VS_gamma/gps/histpoint 0.1972299 0.0
/gate/source/VS_gamma/gps/histpoint 0.19723 0.00033
/gate/source/VS_gamma/gps/histpoint 0.19723001 0.0

/gate/source/VS_gamma/gps/histpoint 0.19822999 0.0
/gate/source/VS_gamma/gps/histpoint 0.19823 0.0033
/gate/source/VS_gamma/gps/histpoint 0.19823001 0.0

/gate/source/VS_gamma/gps/histpoint 0.206799 0.0
/gate/source/VS_gamma/gps/histpoint 0.2068 0.0033
/gate/source/VS_gamma/gps/histpoint 0.2068001 0.0

/gate/source/VS_gamma/gps/histpoint 0.20779999 0.0
/gate/source/VS_gamma/gps/histpoint 0.2078 0.0011
/gate/source/VS_gamma/gps/histpoint 0.2078001 0.0

/gate/source/VS_gamma/gps/histpoint 0.247969999 0.0
/gate/source/VS_gamma/gps/histpoint 0.24797 0.0693
/gate/source/VS_gamma/gps/histpoint 0.247970001 0.0

/gate/source/VS_gamma/gps/histpoint 0.25750999 0.0
/gate/source/VS_gamma/gps/histpoint 0.25751 0.0015
/gate/source/VS_gamma/gps/histpoint 0.257510001 0.0

/gate/source/VS_gamma/gps/histpoint 0.2589999 0.0
/gate/source/VS_gamma/gps/histpoint 0.259 0.0009
/gate/source/VS_gamma/gps/histpoint 0.2590001 0.0

/gate/source/VS_gamma/gps/histpoint 0.27835999 0.0
/gate/source/VS_gamma/gps/histpoint 0.27836 0.0023
/gate/source/VS_gamma/gps/histpoint 0.2783001 0.0

/gate/source/VS_gamma/gps/histpoint 0.28102999 0.0
/gate/source/VS_gamma/gps/histpoint 0.28103 0.072
/gate/source/VS_gamma/gps/histpoint 0.28103001 0.0

/gate/source/VS_gamma/gps/histpoint 0.28531999 0.0
/gate/source/VS_gamma/gps/histpoint 0.28532 0.0043
/gate/source/VS_gamma/gps/histpoint 0.28533001 0.0

/gate/source/VS_gamma/gps/histpoint 0.29518999 0.0
/gate/source/VS_gamma/gps/histpoint 0.29519 0.001588
/gate/source/VS_gamma/gps/histpoint 0.29519001 0.0

/gate/source/VS_gamma/gps/histpoint 0.3293799 0.0
/gate/source/VS_gamma/gps/histpoint 0.32938 0.0026
/gate/source/VS_gamma/gps/histpoint 0.32938001 0.0

/gate/source/VS_gamma/gps/histpoint 0.33069999 0.0
/gate/source/VS_gamma/gps/histpoint 0.3307 0.0116
/gate/source/VS_gamma/gps/histpoint 0.3307001 0.0

/gate/source/VS_gamma/gps/histpoint 0.34372999 0.0
/gate/source/VS_gamma/gps/histpoint 0.34373 0.0043
/gate/source/VS_gamma/gps/histpoint 0.34373001 0.0

/gate/source/VS_gamma/gps/histpoint 0.34635999 0.0
/gate/source/VS_gamma/gps/histpoint 0.34636 0.12
/gate/source/VS_gamma/gps/histpoint 0.34636001 0.0

/gate/source/VS_gamma/gps/histpoint 0.40501999 0.0
/gate/source/VS_gamma/gps/histpoint 0.40502 0.0027
/gate/source/VS_gamma/gps/histpoint 0.40502001 0.0

/gate/source/VS_gamma/gps/histpoint 0.43749999 0.0
/gate/source/VS_gamma/gps/histpoint 0.4375 0.0008
/gate/source/VS_gamma/gps/histpoint 0.43750001 0.0

/gate/source/VS_gamma/gps/histpoint 0.44001999 0.0
/gate/source/VS_gamma/gps/histpoint 0.44002 0.388
/gate/source/VS_gamma/gps/histpoint 0.44002001 0.0

/gate/source/VS_gamma/gps/histpoint 0.45475999 0.0
/gate/source/VS_gamma/gps/histpoint 0.45476 0.0034
/gate/source/VS_gamma/gps/histpoint 0.45476001 0.0

/gate/source/VS_gamma/gps/histpoint 0.50532999 0.0
/gate/source/VS_gamma/gps/histpoint 0.50533 0.288
/gate/source/VS_gamma/gps/histpoint 0.50533001 0.0

/gate/source/VS_gamma/gps/histpoint 0.52896999 0.0
/gate/source/VS_gamma/gps/histpoint 0.52897 1.27
/gate/source/VS_gamma/gps/histpoint 0.52897001 0.0

/gate/source/VS_gamma/gps/histpoint 0.53853999 0.0
/gate/source/VS_gamma/gps/histpoint 0.53854 0.31
/gate/source/VS_gamma/gps/histpoint 0.53854001 0.0

/gate/source/VS_gamma/gps/histpoint 0.55604999 0.0
/gate/source/VS_gamma/gps/histpoint 0.55605 0.0025
/gate/source/VS_gamma/gps/histpoint 0.55605001 0.0

/gate/source/VS_gamma/gps/histpoint 0.56278999 0.0
/gate/source/VS_gamma/gps/histpoint 0.56279 0.0009
/gate/source/VS_gamma/gps/histpoint 0.56279001 0.0

/gate/source/VS_gamma/gps/histpoint 0.57399 0.0
/gate/source/VS_gamma/gps/histpoint 0.574 0.005852
/gate/source/VS_gamma/gps/histpoint 0.57401 0.0

/gate/source/VS_gamma/gps/histpoint 0.57825999 0.0
/gate/source/VS_gamma/gps/histpoint 0.57826 0.0016
/gate/source/VS_gamma/gps/histpoint 0.57826001 0.0

/gate/source/VS_gamma/gps/histpoint 0.59968999 0.0
/gate/source/VS_gamma/gps/histpoint 0.59969 0.0026
/gate/source/VS_gamma/gps/histpoint 0.59969001 0.0

/gate/source/VS_gamma/gps/histpoint 0.61004999 0.0
/gate/source/VS_gamma/gps/histpoint 0.61005 0.0011
/gate/source/VS_gamma/gps/histpoint 0.61005001 0.0

/gate/source/VS_gamma/gps/histpoint 0.62457999 0.0
/gate/source/VS_gamma/gps/histpoint 0.62458 0.078
/gate/source/VS_gamma/gps/histpoint 0.62458001 0.0

/gate/source/VS_gamma/gps/histpoint 0.62825999 0.0
/gate/source/VS_gamma/gps/histpoint 0.62826 0.0016
/gate/source/VS_gamma/gps/histpoint 0.62826001 0.0

/gate/source/VS_gamma/gps/histpoint 0.68793999 0.0
/gate/source/VS_gamma/gps/histpoint 0.68794 0.0268
/gate/source/VS_gamma/gps/histpoint 0.68794001 0.0

/gate/source/VS_gamma/gps/histpoint 0.73586999 0.0
/gate/source/VS_gamma/gps/histpoint 0.73587 0.047
/gate/source/VS_gamma/gps/histpoint 0.73587001 0.0

/gate/source/VS_gamma/gps/histpoint 0.76084999 0.0
/gate/source/VS_gamma/gps/histpoint 0.76085 0.00063
/gate/source/VS_gamma/gps/histpoint 0.76085001 0.0

/gate/source/VS_gamma/gps/histpoint 0.78359999 0.0
/gate/source/VS_gamma/gps/histpoint 0.7836 0.053
/gate/source/VS_gamma/gps/histpoint 0.7836001 0.0

/gate/source/VS_gamma/gps/histpoint 0.83709999 0.0
/gate/source/VS_gamma/gps/histpoint 0.8371 0.00047
/gate/source/VS_gamma/gps/histpoint 0.8371001 0.0

/gate/source/VS_gamma/gps/histpoint 0.87751999 0.0
/gate/source/VS_gamma/gps/histpoint 0.87752 0.00072
/gate/source/VS_gamma/gps/histpoint 0.87752001 0.0

/gate/source/VS_gamma/gps/histpoint 0.89479999 0.0
/gate/source/VS_gamma/gps/histpoint 0.8948 0.0007
/gate/source/VS_gamma/gps/histpoint 0.89480001 0.0

/gate/source/VS_gamma/gps/histpoint 0.89819999 0.0
/gate/source/VS_gamma/gps/histpoint 0.8982 0.0006
/gate/source/VS_gamma/gps/histpoint 0.8982001 0.0

/gate/source/VS_gamma/gps/histpoint 0.90911999 0.0
/gate/source/VS_gamma/gps/histpoint 0.90912 0.0013
/gate/source/VS_gamma/gps/histpoint 0.909120001 0.0

/gate/source/VS_gamma/gps/histpoint 1.03662999 0.0
/gate/source/VS_gamma/gps/histpoint 1.03663 0.00077
/gate/source/VS_gamma/gps/histpoint 1.03663001 0.0

# THE DEFAULT POSITION OF THE VOXELIZED SOURCE IS IN THE 1ST QUARTER
# SO THE VOXELIZED SOURCE HAS TO BE SHIFTED OVER HALF ITS DIMENSION IN THE NEGATIVE DIRECTION ON EACH AXIS
/gate/source/VS_gamma/setPosition -111.794907 -111.3251805 -83.75 mm
/gate/source/VS_gamma/dump 1
```

The source image `patient15_LuDOTATATE_src.h33` allows specific activities to be defined separately for the liver, spleen, left and right kidneys and the three tumours. The source model defines the following values: 
|     Region        |     Voxel value    |     Number   of source voxels    |
|-------------------|--------------------|----------------------------------|
|     Liver         |     15000          |     15381                        |
|     Spleen        |     16000          |     1535                         |
|     L   Kidney    |     17000          |     2052                         |
|     R   Kidney    |     18000          |     2493                         |
|     Tumour1       |     19000          |     93                           |
|     Tumour2       |     20000          |     55                           |
|     Tumour3       |     21000          |     90                           |


See https://github.com/BenAuer2021/Phantoms-For-Nuclear-Medicine-Imaging-Simulation#readme for more information on this source. 

The following example shows how to define a source of <sup>177</sup>Lu gammas for each region. The gamma emission data is from the IAEA database: https://www-nds.iaea.org/relnsd/vcharthtml/VChartHTML.html.
Note that this example shows the gamma emissions only,  so the simulation output will be missing the X-ray peaks and Bremsstrahlung from the betas seen in true <sup>177</sup>Lu spectra. 

`VS_gamma` is the name of the voxelised gamma source 
```ruby

#  Voxelised gamma source of 177Lu
#  For 1MBq of 17Lu,
#  Gamma activity = 0.172688 MBq (scale activity file accordingly)

# Add new voxelised source
/gate/source/addSource VS_gamma voxel
/gate/source/VS_gamma/reader/insert image
/gate/source/VS_gamma/imageReader/translator/insert range
/gate/source/VS_gamma/imageReader/rangeTranslator/readTable patient15_activity_235MBq.dat
/gate/source/VS_gamma/imageReader/rangeTranslator/describe 1
/gate/source/VS_gamma/imageReader/readFile patient15_LuDOTATATE_src.h33

# THE DEFAULT POSITION OF THE VOXELIZED SOURCE IS IN THE 1ST QUARTER
# SO THE VOXELIZED SOURCE HAS TO BE SHIFTED OVER HALF ITS DIMENSION IN THE NEGATIVE DIRECTION ON EACH AXIS
/gate/source/VS_gamma/setPosition -210.9402 -136.7205 -198.882 mm

# Attach to voxel phantom
/gate/source/VS_gamma/attachTo VoxAttn

/gate/source/VS_gamma/gps/ang/type iso

# Set verbosity (2 = every event)
# Good to set to 2 initially to check output is as expected
/gate/source/VS_gamma/gps/verbose 0

# Force unstable
/gate/source/VS_gamma/setForcedUnstableFlag  true
# Half life is 6.647 days
/gate/source/VS_gamma/setForcedHalfLife 574067.52 s

/gate/source/VS_gamma/gps/particle    gamma
/gate/source/VS_gamma/gps/ene/type  User
/gate/source/VS_gamma/gps/hist/type    energy

/gate/source/VS_gamma/gps/ene/min    0.0716419 MeV
/gate/source/VS_gamma/gps/ene/max    0.321315901 MeV

# ------------------hist of emissions----------------------------- #
/gate/source/VS_gamma/gps/hist/point 0.0716419999 0.0
/gate/source/VS_gamma/gps/hist/point 0.071642 0.164
/gate/source/VS_gamma/gps/hist/point 0.0716420001 0.0

/gate/source/VS_gamma/gps/hist/point 0.1129497999 0.0
/gate/source/VS_gamma/gps/hist/point 0.1129498 6.23
/gate/source/VS_gamma/gps/hist/point 0.11294980001 0.0

/gate/source/VS_gamma/gps/hist/point 0.1367244999 0.0
/gate/source/VS_gamma/gps/hist/point 0.1367245     0.0465
/gate/source/VS_gamma/gps/hist/point 0.13672450001 0.0

/gate/source/VS_gamma/gps/hist/point 0.2083661999 0.0
/gate/source/VS_gamma/gps/hist/point 0.2083662     10.41
/gate/source/VS_gamma/gps/hist/point 0.20836620001 0.0

/gate/source/VS_gamma/gps/hist/point 0.2496741999 0.0
/gate/source/VS_gamma/gps/hist/point 0.2496742     0.1997
/gate/source/VS_gamma/gps/hist/point 0.24967420001 0.0

/gate/source/VS_gamma/gps/hist/point 0.3213158999 0.0
/gate/source/VS_gamma/gps/hist/point 0.3213159     0.2186
/gate/source/VS_gamma/gps/hist/point 0.3213159001 0.0
###################################################

/gate/source/list
/gate/source/VS_gamma/dump 1

```
The file `patient15_activity_235MBq.dat` permits the definition of activity in each region of the source image, based on its voxel values. 
The following example sets a total activity of 235 MBq in the phantom.

``` ruby
7
15000.0 15000.0 1167.645
16000.0 16000.0 3847.511
17000.0 17000.0 2625.665
18000.0 18000.0 2576.813
19000.0 19000.0 28595.65
20000.0 20000.0 25746.21
21000.0 21000.0 9210.026667
```
The first row specifies the number of active regions we want to set. The following rows specify a voxel range to set to an activity (Bq/voxel). For example, here we have 3847.511 Bq/voxel in the spleen which has 1535 source voxels. Therefore we define a total gamma activity of 5.91 MBq. Note that <sup>177</sup>Lu has a total gamma decay branching ratio of  0.172688, so this corresponds to a total activity of <sup>177</sup>Lu  of 34.2 MBq. 

### Output

The ROOT output can be defined as follows 

```ruby
/gate/output/root/enable
/gate/output/root/setFileName ./PathTo/outputFileName
/gate/output/root/setRootSinglesFlag 1
```
where `./PathTo/outputFileName` gives the path and base name for the output root file to be written to (no extension is given). Here we have specified that the ROOT Singles tree should be recorded. Singles are the raw Hits data after the Digitizer modules have been applied.  We can turn off output of other trees to reduce the filesize of our output e.g: 

```ruby
/gate/output/root/setRootHitFlag 0
/gate/output/root/setRootNtupleFlag 0
/gate/output/root/setRootSinglesAdderFlag 0
/gate/output/root/setRootSinglesBlurringFlag 0
/gate/output/root/setRootSinglesSpblurringFlag 0
```

We can also write projections directly from the simulation as interfile images (*.sin *.hdr). The energy window for the projection should have been already specified as a thresholder digitizer module: 
```ruby
/gate/output/projection/enable
/gate/output/projection/setInputDataName Photopeak
/gate/output/projection/setFileName ./PathTo/outputFileName
/gate/output/projection/projectionPlane YZ
/gate/output/projection/pixelSizeX 0.234 cm
/gate/output/projection/pixelSizeY 0.234 cm
/gate/output/projection/pixelNumberX 230
/gate/output/projection/pixelNumberY 170
```
The number and size of pixels in the x and y directions are specified. This will create an interfile projection image of the specified dimensions with 32 bit float data. 

### Running the simulation

First we must set the random engine and its seed. `auto` automatically sets a random seed for each job or it can be specified. If running multiple simulation jobs and combining to improve statistics, make sure they have different specified random seeds or use `auto`. 

```ruby
############################### RANDOM ###############################
/gate/random/setEngineName MersenneTwister
/gate/random/setEngineSeed auto
/gate/random/verbose 1
```

Now we start the simulation. We can specify a time slice, this will be the time for  each head position in step-and-shoot acquitions. In this example we define 60 projection of 1 second per head (120 projections in total over 360 degree). To be used with the first orbiting definition above (3 deg/s with head replication along z).
```ruby
############################### START ################################
/gate/application/setTimeSlice      1.0  s
/gate/application/setTimeStart      0.0    s
/gate/application/setTimeStop       60.0  s #180 degree 2 heads
```

In this other example we use 40 seconds per projection. We then set a start and stop time. Here we used 32 projections per head so 1280 seconds. To be used with the first orbiting definition above (0.140625 deg/s with head replication along z).

```ruby
############################### START ################################
/gate/application/setTimeSlice      40.0  s
/gate/application/setTimeStart      0.0  s
/gate/application/setTimeStop       1280 s # 180 degree for each head
/gate/application/startDAQ
```

### Visualization

The following commands can be added to a GATE macro to permit visualization in OpenGL. This function requires installation of X11 and Qt. 

```ruby
/vis/open                             OGLI # or OGLS (OGLI better for voxelized phantom)
/vis/drawVolume
/vis/viewer/flush
/tracking/storeTrajectory             1
/vis/scene/add/trajectories
/vis/scene/endOfEventAction           accumulate

/vis/scene/add/axes            0 0 0 500 mm # Option to add axes
/vis/scene/add/text            10 0 0 cm  20 0 0   X # Option to add labels for axes
/vis/scene/add/text            0 10 0 cm  20 0 0   Y
/vis/scene/add/text            0 0 10 cm  20 0 0   Z
/vis/viewer/set/viewpointThetaPhi 0 0 
/vis/viewer/set/auxiliaryEdge true

#/vis/viewer/set/viewpointThetaPhi 90 90 # View along Oy
#/vis/viewer/set/viewpointThetaPhi 90 360 # View along Ox
/vis/viewer/set/viewpointThetaPhi 0 0 # View along Oz

/vis/viewer/zoom 1.2 # zoom into the scene

/vis/viewer/set/background white # background can be set to white. it is black by default.

```

### Run the simulation

The last command in the .mac file executes the simulation 
```ruby
 /gate/application/startDAQ
```
To run the simulation, open a terminal prompt, and type `path_to/Gate macro.mac`. To visualize and manipulate the geometry, run via `path_to/Gate --qt macro.mac`.

# 3. Simulating the BrightView system equiped with LEHR, LEHR-VXHR, MEGP and HEGP collimator in GATE

We also provide the GATE macros to simulate the BrightView system equiped with other collimators (LEHR, LEHR-VXHR, and HEGP) introduced above and available for download : https://github.com/BenAuer2021/Simulation-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/blob/main/GATE_MACROS_%20Jaszczak.zip 

The LEHR and LEHR-VXHR were built by folding lead alloy foils, forming double septa on two opposing sides and single septa on the other four sides of the hexagonal holes. The MEGP and HEGP collimators are constructed by casting lead, where all six walls are formed by single septa. The LEHR and LEHR-VXHR hexagon holes are oriented in 90° with respect to the holes of the MEGP and HEGP collimators. As shown on the figure below, the LEHR, LEHR-VXHR, MEGP, and HEGP collimators consist of 354 × 350, 230 x 216, 146 × 93 and 112 × 72 holes, respectively. The NaI(T1) detector surface area is 540 × 400 mm<sup>2</sup>.

<p align="center">
<img width="900" alt="Screen Shot 2023-06-21 at 9 42 31 PM" src="https://github.com/BenAuer2021/Simulation-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/assets/84809217/3bb590cd-5220-49d3-a80a-6a8d6ff20dca">
</p>

<p align="center">
<img width="900" alt="Screen Shot 2023-06-21 at 10 15 22 PM" src="https://github.com/BenAuer2021/Simulation-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/assets/84809217/d9a4a560-72f8-4146-909d-538c5c759d83">
</p>


# 4. Simulating the BrightView system equiped with Single-Pinhole collimator in GATE

We provide one example of planar imaging with the BrightView system in the context of I-123 thyroid scintigraphy with single pinhole collimator - https://github.com/BenAuer2021/Simulation-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/blob/main/GATE_MACROS_%20PINHOLE_Thyroid.zip .

The single pinhole collimator was designed with primitive objects available by default in GATE, and with a combination of primitive and STL-based objects. We provide the GATE macros and required files for both models.

The STL files for the crystal, collimator, and aluminium housing were designed in [Solidworks]<sup>R</sup> {https://www.solidworks.com/}, and then exported into the STL format (triangular surface meshes).

The figure below illustrate the differences between primitive and STL-based modeling of the collimator.

<p align="center">
<img width="900" alt="Screen Shot 2023-06-21 at 10 55 43 PM" src="https://github.com/BenAuer2021/Phantoms-For-Nuclear-Medicine-Imaging-Simulation/assets/84809217/fc97d513-6f32-4f29-8228-78fd53da3793">
</p>

# 5. Simulating bone imaging with the BrightView system in GATE

We provide an example of multi-bed skeletal Tc-99m MDP imaging with the BrightView system equipped with LEHR collimator - https://github.com/BenAuer2021/Simulation-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/blob/main/GATE_MACROS_BoneImaging.zip .
The attenuation and activity phantoms were divided into 3 sub phantoms each 40 cm long axially to fit within the axial field of view of the imaging system. This resulted in an improvement in computation efficiency, as simulating the HeadTorsoAbd region of the mesh50 phantom as a whole would not have been efficient as the vast majority of the gammas would be emitted outside of the system field of view. The source and attenuation phantoms were derived from the whole-body skeletal mesh50_XCAT phantom described here: https://github.com/BenAuer2021/Mesh-based-Human-Phantom-for-Simulation . The number of projections was set to 64 over 360 degree, resulting in 32 views per head.

<p align="center">
<img width="900" alt="Screen Shot 2023-06-22 at 12 59 52 AM" src="https://github.com/BenAuer2021/Simulation-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/assets/84809217/24485a2a-77b5-414d-a498-3ae77bc0260c">
</p>

# 6. Simulating brain perfusion and DaT imaging with the BrightView system in GATE

We provide an example of Tc-99m HMPAO brain perfusion imaging with the BrightView system equipped with LEHR collimator and I-123 IMP brain perfusion and I-123 DaT with the BrightView system equipped with MEGP collimator - https://github.com/BenAuer2021/Simulation-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/blob/main/GATE_MACROS_BrainImaging.zip .
The source and attenuation phantoms are described in details here: https://github.com/BenAuer2021/Phantoms-For-Nuclear-Medicine-Imaging-Simulation .

<p align="center">
<img width="900" alt="image" src="https://github.com/BenAuer2021/Simulation-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/assets/84809217/d1942447-1817-4ef0-ad1b-8b80dc65c254">
</p>

# 7. Simulating glioblastoma imaging with the BrightView system in GATE

We provide an example of I-131 glioblastoma imaging with the BrightView system equipped with HEGP collimator - https://github.com/BenAuer2021/Simulation-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/blob/main/GATE_MACROS_GlioBlastomaImaging.zip .
The source and attenuation phantoms are described in details here: https://github.com/BenAuer2021/Phantoms-For-Nuclear-Medicine-Imaging-Simulation .

# 8. Simulating brain perfusion and DaT imaging with the BrightView system incorporating the STL-based mesh50 attenuation phantom in GATE

We provide an example of Tc-99m HMPAO brain perfusion and Tc-99m TRODAT-1 DaT imaging with the BrightView system equipped with LEHR collimator and incorporating the open-source XCAT mesh50 phantom - https://github.com/BenAuer2021/Simulation-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/blob/main/GATE_MACROS_BrainImaging_STL.zip .
The source and attenuation phantoms based on the mesh50 phantom are described in details here: https://github.com/BenAuer2021/Mesh-based-Human-Phantom-for-Simulation .

<p align="center">
<img width="900" alt="Screen Shot 2023-06-22 at 1 03 32 AM" src="https://github.com/BenAuer2021/Simulation-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/assets/84809217/73dc9400-b228-4522-b72e-28ce383a4ff9">
</p>

# 9. Simulating a Lu-177 DOTATATE post-therapy SPECT with the BrightView system in GATE

We provide an example of a post Lu-177 DOTATATE therapy SPECT  with the BrightView system equipped with MEGP collimators - https://github.com/BenAuer2021/Simulation-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/blob/main/GATE_MACROS_LuDOTATE_SPECT.zip. The source and attenuation phantoms are described in details here: https://github.com/BenAuer2021/Phantoms-For-Nuclear-Medicine-Imaging-Simulation.

![LuPatientSPECT_xsec](https://github.com/BenAuer2021/Simulation-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/assets/55833314/c85de4e9-b81c-429e-bdf5-d3ec43f48185)





