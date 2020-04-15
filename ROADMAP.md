# slicereg roadmap

## Outline

### Load data
##### Load image data
* Could use [brainio](https://github.com/adamltyson/brainio), similarly to 
[amap](https://github.com/SainsburyWellcomeCentre/amap-python).
* Any other formats to support?
*  Considering these images could be very large 
(for e.g. spatial transcriptomics), we should consider memmap/virtual stack 
or [dask](https://dask.org/) which works well with 
[napari](https://github.com/napari/napari).

##### Load metadata
* No support for this yet in brainio.
* Need pixel spacing (x/y), and z spacing (not necessarily uniform)
* The rough volume of the brain imaged should be defined in atlas coordinates.
 Alternatively, brain regions could be supplied, and a 
bounding box could be generated based on these.


### Prepare registration
* Using metadata, load a corresponding volume of the average brain into memory. 
This volume should encompass the every 2D section.
* Display (with `napari`) a single 2D image (e.g. first) along with the 3D 
volume and a "best guess" 2D slice through the volume that corresponds to the 
2D image.
* Use sliders/keybindings to adjust the 2D slice to match the original 2D 
image (scale, shear, rotations etc.).
* Repeat above for N sections.
* Interpolate any initial transformations, and apply to all sections.
* Resample images to match the template
* Preprocess 2D images to best match the template brain, e.g. 
`amap.register.brain_processor.filter`.


### Register images
* Algorithm TBC. I don't know whether a feature-based or a mutual-information
 type approach would be the best. There seems to be lots to try in openCV, but 
 I would favour scikit-image if at all possible for ease of installation, 
 although their registration options are less mature, and some still a work 
 in progress. 
 
* Loop through each 2D image. (multiprocessing?)


### Feature detection
* Sparse cell detection should be (relatively) straightforward with standard 
approaches (ideally with [scikit-image](https://scikit-image.org/)).
* More complex cell detection could use a cellfinder-type approach (see 
[issue](https://github.com/SainsburyWellcomeCentre/cellfinder/issues/79)).
* Regions could be detected using 2D versions of approaches in `neuro`.
* Loop through each 2D image. (multiprocessing?)

### Transformation into standard space
Once features are detected, they should be transformed into standard space. 
 This will probably be two-step process:
* The transform from the registration step should be inverted and applied to 
 the features, transforming them into the space of the template subvolume.
* The coordinates of the features should then be converted to the space of 
the whole brain, by adding the offsets of the subvolume from the origin.

### Visualisation
* `napari` for detailed 2D orthogonal views, in the style of 
`cellfinder.visualisation.two_dimensional` or `neuro.visualise.amap_vis`. All 
2D figures 
[on the cellfinder github page](https://github.com/SainsburyWellcomeCentre/cellfinder)
 were generated with napari.
 * `brainrender` for 3D interactive visualisation. There is code to convert 
 the `cellfinder` format to be imported directly into `brainrender` in 
 `neuro.points.points_to_brainrender`, but this should happen automatically 
 in `slicereg`.


## Questions
* What atlas do we want to support? Currently the 10um ARA is the default for
[cellfinder](https://github.com/SainsburyWellcomeCentre/cellfinder) and
[amap](https://github.com/SainsburyWellcomeCentre/amap-python). `amap` has code 
to download this (running `amap_download` will fetch it). 
To simplify dependencies, this might move to 
[neuro](https://github.com/SainsburyWellcomeCentre/neuro). Only the ARA (but 
for any resolution) is currently supported by `brainrender`.

* How do we want to support full datasets (i.e. multiple samples)? Should there
 be a batch mode that allows multiple datasets to be run in series, or a 
 separate function to collate them into a single file that can be imported 
 into the visualisation, or downstream analysis?
 
* Do we want `slicereg` to have any analysis functionality (i.e. to analyse
the spatial distribution of detected features)? `cellfinder` will have a
 companion software 
 [cellfinder-analyst](https://github.com/adamltyson/cellfinder-analyst) for 
 this, but nothing much has been done yet.
 



