# Simulation-And-Reconstruction-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT



## 1. Philips BrightView system

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
/gate/world/vis/setVisible 0
/gate/world/vis/setColor green
/gate/world/setMaterial Air
```
The world defines the experimental framework of the simulation. It must be large enough to contain the entire SPECT system and phantom. The tracking of particles is stopped once they leave the `world`. The `world` is shown as a green wore frame in the visulazation of th esimulation shown in Figure 1. `GateMaterials.db' must contain all elemental compositions and densities of materials which are defined later in the macro. 

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
The `SPECThead` volume **must** be large enough to include all components of the detector (including collimators or shielding), but not cover any part of the source or phantom. Any shielding material outside the `SPECThead` volume will be ignored in the simulation (i.e. photons will pass straight through). The `SPECThead` is shown as a cyan wire frame in Figure 1. 

The positioning of the SPECT head will depend on the phantom that is used. The hierarchical structure of GATE means that any phantom volume will overwrite any SPECThead volume in the same position, this includes any air around the corners of a voxelised phantom. 

All detector components must be defined relative to the **center** of the `SPECThead` volume. Any translation set in `/gate/SPECThead/placement/setTranslation` will be applied to all components. Therefore, `/gate/SPECThead/placement/setTranslation` is a good way to set the radius of the detector during the acquisition. 

Due to the phantom we are using here, a radius of 45 cm was set to avoid overlap of the phantom volume. Using many repetitions  of the `SPECThead`   in the visualization (`/gate/SPECThead/ring/setRepeatNumber` below) can be useful to check for overlap at all rotation angles:

![Vis_BV_LuPatient_TestOverlap](https://github.com/BenAuer2021/Simulation-And-Reconstruction-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/assets/55833314/79ec0c40-9a79-4654-9a50-7dcc04983fc7)

Now we define the components within the `SPECThead`. We will start with the NaI crystal and its cover.

```ruby 
# Crystal cover
/gate/cry_cover/daughters/name Al_sheet
/gate/cry_cover/daughters/insert box
/gate/Al_sheet/setMaterial Aluminium
/gate/Al_sheet/geometry/setXLength 0.1 cm
/gate/Al_sheet/geometry/setYLength 54 cm
/gate/Al_sheet/geometry/setZLength 40 cm
/gate/Al_sheet/placement/setTranslation 0.22625 0. 0. cm # relative to center of SPECThead
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

```
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
/gate/crystal/attachCrystalSD****
```
The line `/gate/crystal/attachCrystalSD` sets this crystal as a _sensitive detector_ which means that _hits_ in this volume are recorded (see the Digitizer section). 


### Digitizer 

The digitizer modules mimic the response of the detector and electronic signal chain in the physical system. The energy of all photons absorbed in the crystal volume will be recorded as an exact value, so digitiser modules must be used to simulate the finite resolution of the physical detectors. 

For SPECT, digitizers for energy and spatial blurring shoul dbe added, based on the specific system being modelled. Here, an energy blurring of 10% at 159 keV, and an intrinsic spatial resolution of 3 mm are set. 

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

Geant4 containes a library of pre-built physics list which describe all processes for photon and particle interaction with material. This can be set with 
```/gate/physics/addPhysicsList emstandard_opt4```

There appears to be an issue with scatter recording when atomic de-excitation is included. Therefore, for simulations where accurate scatter information is required, I would recommend to turn off atomic de-excitation with `/process/em/pixe false` with the physics definition and then `/process/em/fluo 0` after the simulation has been initialised. 

We also define cuts which determine a threshold below which no secondary particles will be generated. Each volume can be given specific cuts, or else it will inheret from its parent volume. The default cuts for the world are 1 mm. 

``` ruby
##############################  C U T S ##############################
/gate/physics/Gamma/SetCutInRegion      SPECThead 0.1 cm
/gate/physics/Electron/SetCutInRegion   SPECThead 0.1 cm
```




The initialisation step must be performed **after** the geometry, phantom and digitizer is set and **before** the definition of the source and root output. 

```
/gate/run/initialize
/gate/physics/processList Initialized
```


### Source definition



### ROOT output



### Running the simulation


### Visualization
