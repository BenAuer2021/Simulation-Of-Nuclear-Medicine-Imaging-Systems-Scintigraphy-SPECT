# Simulation-And-Reconstruction-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT

Contacts: bauer@bwh.harvard.edu, sophia.pells@umassmed.edu

Table of contents:

## 1. Philips BrightView system

### 1.1 Collimator models 

## 2. SPECT simulation of the Philips BrightView in GATE - <sup>177</sup>Lu-DOTATATE patient 

This section provides an example for setting up a GATE simulation of a SPECT scan for a <sup>177</sup>Lu-DOTATATE patient. It uses the phantoms available here https://github.com/BenAuer2021/Phantoms-For-Nuclear-Medicine-Imaging-Simulation/blob/main/patient_LuDOTATATE_phantoms.zip

The Philips BrightView model with MEGP collimators is used. 

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
The world defines the experimental framework of the simulation. It must be large enough to contain the entire SPECT system and phantom. The tracking of particles is stopped once they leave the `world`. The `world` is shown as a green wire frame in the visulazation of the simulation shown in Figure 1. `GateMaterials.db' must contain all elemental compositions and densities of materials which are defined later in the macro. 

![Vis_BV_LuPatient](https://github.com/BenAuer2021/Simulation-And-Reconstruction-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/assets/55833314/0995922f-eeda-4069-9f25-926ca7a8e76c)
<br /> Figure 1: Visualisation of the simulation discussed in this Section. 


### Define the detector

Next we must define our detector and its components. For SPECT, we must define a volume of type `SPECThead`. This volume will contain all components of the SPECT camera detectors. 

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
The `SPECThead` volume **must** be large enough to include all components of the detector (including collimators or shielding), but not cover any part of the source or phantom. Any shielding material outside the `SPECThead` volume will be ignored in the simulation (i.e. photons will pass straight through). The `SPECThead` is shown as a cyan wire frame in Figure 1. Visualization commands e.g. `/vis/setColor' can be set for all volumes, but visualization will only occur when a viewer is specified (see Visualization section below). 

The positioning of the SPECT head will depend on the acquisition and the phantom that is used. The hierarchical structure of GATE means that any phantom volume will overwrite any SPECThead volume in the same position, this includes any air around the corners of a voxelised phantom. 

All detector components must be defined relative to the **center** of the `SPECThead` volume. Any translation set in `/gate/SPECThead/placement/setTranslation` will be applied to all components. Therefore, `/gate/SPECThead/placement/setTranslation` is a good way to set the radius of the detector during the acquisition. This is the radius to the center of the `SPECThead` volume, not to the front collimator or touch plate.

Due to the phantom we are using here, a radius of 45 cm was set to avoid overlap of the phantom volume. Using many repetitions  of the `SPECThead` (`/gate/SPECThead/ring/setRepeatNumber` below)  and running  the visualization can be useful to check for overlap at all rotation angles:

![Vis_BV_LuPatient_TestOverlap](https://github.com/BenAuer2021/Simulation-And-Reconstruction-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/assets/55833314/79ec0c40-9a79-4654-9a50-7dcc04983fc7)

Now we define the components within the `SPECThead`. We will start with the NaI crystal and its cover.

```ruby 
# Crystal cover
/gate/lead_casing/daughters/name cry_cover
/gate/lead_casing/daughters/insert box
/gate/cry_cover/setMaterial Lead
/gate/cry_cover/geometry/setXLength 1.5525 cm
/gate/cry_cover/geometry/setYLength 54 cm
/gate/cry_cover/geometry/setZLength 40 cm
/gate/cry_cover/placement/setTranslation 7.47625 0. 0 cm # relative to center of SPECThead
/gate/cry_cover/vis/setColor yellow
/gate/cry_cover/attachPhantomSD

# Air Gap
/gate/cry_cover/daughters/name Air_gap
/gate/cry_cover/daughters/insert box
/gate/Air_gap/setMaterial Air
/gate/Air_gap/geometry/setXLength 0.5 cm
/gate/Air_gap/geometry/setYLength 54 cm
/gate/Air_gap/geometry/setZLength 40 cm
/gate/Air_gap/placement/setTranslation 0.52625 0. 0. cm # relative to center of cry_cover
/gate/Air_gap/vis/setColor red
/gate/Air_gap/vis/forceSolid

# Aluminium box
/gate/cry_cover/daughters/name Al_sheet
/gate/cry_cover/daughters/insert box
/gate/Al_sheet/setMaterial Aluminium
/gate/Al_sheet/geometry/setXLength 0.1 cm
/gate/Al_sheet/geometry/setYLength 54 cm
/gate/Al_sheet/geometry/setZLength 40 cm
/gate/Al_sheet/placement/setTranslation 0.22625 0. 0. cm # relative to center of cry_cover
/gate/Al_sheet/vis/setColor white
/gate/Al_sheet/vis/forceWireframe
/gate/Al_sheet/attachPhantomSD

# Crystal
/gate/cry_cover/daughters/name crystal
/gate/cry_cover/daughters/insert box
/gate/crystal/setMaterial NaI
/gate/crystal/geometry/setXLength 0.9525 cm
/gate/crystal/geometry/setYLength 54 cm
/gate/crystal/geometry/setZLength 40 cm
/gate/crystal/placement/setTranslation -0.3 0. 0. cm # relative to center of cry_cover
/gate/crystal/vis/setColor yellow
/gate/crystal/vis/forceSolid
```
Next, we will define the collimator. These will be the MEGP collimators of the Philips BrightView. This is defined as a block of lead with an array of hexagonal holes. First we define the lead block

```ruby 
/gate/SPECThead/daughters/name collimator
/gate/SPECThead/daughters/insert box
/gate/collimator/geometry/setXLength 53 cm   
/gate/collimator/geometry/setYLength 64 cm 
/gate/collimator/geometry/setZLength 5.84 cm 
/gate/collimator/placement/alignToX
/gate/collimator/placement/setTranslation 15.63 0.0 0.0 cm 
/gate/collimator/setMaterial CollLead
/gate/collimator/vis/setColor grey
/gate/collimator/vis/forceSolid
/gate/collimator/attachPhantomSD
```
`alignToX` reorients the block along the x-axis (such that the patient bed is on the z axis). 

Now we insert a hexagonal hole of air into the collimator block and repeat it to create the array. Note that we have to rotate the hole by 90 degrees to orientate it correctly with the block. 

```ruby
# Insert the first hole of air in the collimator
/gate/collimator/daughters/name hole
/gate/collimator/daughters/insert hexagone
/gate/hole/geometry/setHeight 5.84 cm
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
/gate/hole/linear/setRepeatVector 2.13 3.6893 0.0 mm # (0.152 mm thick) 
```

The figure below shows a close-up of the initial hexagonal hole, the hexagonal array after the first repeater and then after the second. 
![coll_hole_array](https://github.com/BenAuer2021/Simulation-And-Reconstruction-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/assets/55833314/09f44bf2-6f94-4138-b0d1-8474dc31d173)

Other detector components, e.g. shielding and electronics, are defined in the same way. After the components of the `SPECThead` have been defined, we can repeat the detector for dual- (or more) head acquisition. 
In this case we create two SPECTheads. They will automatically be positoned 180 degrees apart. 

```ruby
/gate/SPECThead/repeaters/insert ring
/gate/SPECThead/ring/setRepeatNumber 2
/gate/SPECThead/ring/setPoint1 0. 0. 0. cm
/gate/SPECThead/ring/setPoint2 0. 0. 1. cm
```

We now set the orbit speed for SPECT acquisitions. Here we consider 32 projections for each head with an acquisition time of 40 seconds per projection. The orbitting speed here must correspond to the frame time slice set later (see Running the Simulation). Here we set the orbitting about the z axis (where our patient bed will be). 

```ruby
/gate/SPECThead/moves/insert orbiting
# Want each head to move 180 degrees in 32 projections
# 5.625 deg/frame movement.
# Setting 40 seconds per frame so 0.140625 deg/s
/gate/SPECThead/orbiting/setSpeed 0.140625 deg/s
/gate/SPECThead/orbiting/setPoint1 0. 0. 0. cm
/gate/SPECThead/orbiting/setPoint2 0. 0. 1. cm
```

### Phantom definition

Now we define the phantom. Here the phantom is defined at the center of the world.
```ruby
/gate/world/daughters/name voxelPhantom
/gate/world/daughters/insert ImageNestedParametrisedVolume 
/gate/voxelPhantom/geometry/setImage ./PathTo/patient15_LuDOTATATE_attn.h33
/gate/voxelPhantom/geometry/setRangeToMaterialFile ./PathTo/patient15_ID2mat.txt
/gate/voxelPhantom/placement/setTranslation  0. 0. 0. cm
/gate/voxelPhantom/attachPhantomSD
```
The file `patient15_ID2mat.txt` specifies the materials to be assigned to each voxel of the image. The materials in this file must correspond to ones defined a file set with the `/gate/geometry/setMaterialDatabase` command. See https://github.com/BenAuer2021/Phantoms-For-Nuclear-Medicine-Imaging-Simulation#readme for the materials used in this image. Be sure to include `/gate/voxelPhantom/attachPhantomSD` so that all scatter interactions within the phantom are recorded. The number of Compton and Rayleigh events will be recorded throughout all volumes assigned `attachPhantomSD` for any photons absorbed in the crystal. 

### Sensitive detectors
We must now set our crystal as a sensitive volume, so that photon interactions within it will be stored as _hits_ and processed according to the Digitizer set below. 

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
/gate/digitizer/Singles/blurring/linear/setEnergyOfReference 159 keV

/gate/digitizer/Singles/insert spblurring
/gate/digitizer/Singles/spblurring/setSpresolution 3. mm       
```
Other Digitizer modules can be set for e.g. deadtime and thresholding.  Energy windows can also be set to be used for projection output later:
```ruby
/gate/digitizer/name WindowPhotopeak
/gate/digitizer/insert singleChain
/gate/digitizer/WindowPhotopeak/setInputName Singles
/gate/digitizer/WindowPhotopeak/insert thresholder
/gate/digitizer/WindowPhotopeak/thresholder/setThreshold 187.2 keV
/gate/digitizer/WindowPhotopeak/insert upholder
/gate/digitizer/WindowPhotopeak/upholder/setUphold 228.8 keV
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

We can also write projections directly from the simulation. The energy window for the projection should have been already specified as a thresholder digitizer module: 
```ruby
/gate/output/projection/enable
/gate/output/projection/setInputDataName WindowPhotopeak
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

Now we start the simulation. We can specify a time slice, this will be the time for  each head position in step-and-shoot acquitions. In this example we use 40 seconds per projection. We then set a start and stop time. Here we used 32 projections per head so 1280 seconds. 

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
```




## 3. Reconstruction in CASToR

CASToR provides a tool to create a CASToR datafile directly from a GATE macro and root file: `castor-GATErootToCastor`. To use: 

```castor-GATErootToCastor -i path/to/ifile.root -o path/to/outfile -m path/to/macrofile.mac -s scanner_alias -sp_bins bins_x,bins_y``` <br />
where <br />
`path/to/ifile.root` is the root output file from Gate <br />
`path/to/outfile` is the base name to save the CASToR datafile to <br />
`path/to/macrofile.mac` is the Gate macro file used to generate the root file <br />
`scanner_alias` corresponds to a `scanner_alias.geom` file in your `castor\config\scanner\` directory.  <br />
`bins_x,bins_y` are the transaxial and axial number of bins for projections, separated by a comma.

Note that CASToR expects the macro to have units of cm. Comments after commands can also cause issues so make sure all comments are on their own new line. 

The `scanner_alias.geom` file defines the components of your detector. Below is an example for a SPECT system: 

```ruby
modality: SPECT_CONVERGENT
scanner name: SPECT_BRIGHTVIEW
description: This scanner description is based on an actual scanner, however, this implementation is not supported nor validated by its manufacturer.

number of detector heads: 2

trans number of pixels: 1 # 1 in case of monolythic
trans pixel size: 540.0 # Given in mm
trans gap size: 0 # Given in mm

axial number of pixels: 1
axial pixel size: 400.0 # Given in mm
axial gap size: 0 # Given in mm

detector depth: 20

# Distance between the center of rotation (COR) of the scanner and the surface of a detection head in mm
scanner radius: 264.5,264.5 # Head 1 and 2 ROR for patient

# Collimator configuration

head1:
trans focal model: constant
trans number of coef model: 1
trans parameters: 0 # focal distance in mm, 0 for parallel
axial focal model: constant
axial number of coef model: 1
axial parameters: 0

head2:
trans focal model: constant
trans number of coef model: 1
trans parameters: 0 # focal distance in mm, 0 for parallel
axial focal model: constant
axial number of coef model: 1
axial parameters: 0

voxels number transaxial        : 128 # optional (default is the half of the scanner radius)
voxels number axial                : 128 # optional (default is length of the scanner computed from the given parameters)

field of view transaxial        : 613.7856 # optional (default is the half of the scanner radius)
field of view axial                : 613.7856 # optional (default is length of the scanner computed from the given parameters)
```

For the patient SPECT simulation, we set a translation of the `SPECThead` of 450 mm. The center of the SPECThead to center of the colimator is 158.5 mm and the collimator length is 54 mm. Therefore, the radius from COR to the front of collimator is 264.5 mm. 

Another argument `-t` can be provided to use only the true photons (i.e. unscattered), this will give a perfect scatter-corrected image to reconstruct. 

Running this command will generate a CASToR datafile (.Cdf) and header (.Cdh). 

The CASToR datafile can then be reconstructed with the `castor-recon` executable. 
