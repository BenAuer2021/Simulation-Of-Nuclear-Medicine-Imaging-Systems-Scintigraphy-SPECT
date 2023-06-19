# Simulation-And-Reconstruction-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT



## 1. SPECT simulation of the Philips BrightView - <sup>177</sup>Lu-DOTATATE patient 

This section provides an example for setting up a GATE simulation of a SPECT scan for a <sup>177</sup>Lu-DOTATATE patient. It uses the phantoms available here https://github.com/BenAuer2021/Phantoms-For-Nuclear-Medicine-Imaging-Simulation/blob/main/patient_LuDOTATATE_phantoms.zip


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

All detector components must be defined relative to the **center** of the `SPECThead` volume. Any translation set in `/gate/SPECThead/placement/setTranslation' will be applied to all components. Therefore, `/gate/SPECThead/placement/setTranslation` is a good way to set the radius of the detector during the acquisition. 

Due to the phantom we are using here, a radius of 45 cm was set to avoid overlap of the phantom volume. Using many repetitions  of the `SPECThead`   in the visualization (`/gate/SPECThead/ring/setRepeatNumber` below) can be useful to check for overlap at all rotation angles:

![Vis_BV_LuPatient_TestOverlap](https://github.com/BenAuer2021/Simulation-And-Reconstruction-Of-Nuclear-Medicine-Imaging-Systems-Scintigraphy-SPECT/assets/55833314/79ec0c40-9a79-4654-9a50-7dcc04983fc7)

Now we define the components within the `SPECThead`. We will start with the NaI crystal

```ruby 
# Crystal
/gate/cry_cover/daughters/name crystal
/gate/cry_cover/daughters/insert box
/gate/crystal/setMaterial NaI
/gate/crystal/geometry/setXLength 0.9525 cm
/gate/crystal/geometry/setYLength 54 cm
/gate/crystal/geometry/setZLength 40 cm
/gate/crystal/placement/setTranslation -0.3 0. 0. cm # relative to center of SPECT head volume
/gate/crystal/vis/setColor yellow
/gate/crystal/attachCrystalSD
/gate/crystal/vis/forceSolid
```
The line `/gate/crystal/attachCrystalSD` sets this crystal as a _sensitive detector_ which means that _hits_ in this volume are recorded (see the Digitizer section). 


### Visualization
