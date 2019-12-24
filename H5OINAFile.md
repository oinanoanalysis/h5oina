# Oxford Instruments NanoAnalysis HDF5 File Specification

Version | Release date
--- | ---
1.0 | AZtec 4.2

This document details the specification for the Oxford Instruments NanoAnalysis HDF5 file format (_.h5oina_).
This file format can be used to export EBSD acquisitions and combined EBSD/EDS acquisitions.
The file format is largely influenced by the H5EBSD file format developed by Jackson et al. (2014) [[doi](http://dx.doi.org/10.1186/2193-9772-3-4)].
It is using the [Hierarchical Data Format 5](http://www.hdfgroup.org) file format library, which has several implementations in different programming languages.

## Table of Content

- [General notes](#general-notes)
- [AZtec project data tree](#data-tree)
- File Layout
  - [Root level](#root-level)
  - [Slice](#slice)
  - [Technique](#technique)
  - [Common header](#common-header)
  - [EBSD](#ebsd)
    - [Data](#ebsd-data)
    - [Header](#ebsd-header)
  - [EDS](#eds)
    - [Data](#eds-data)
    - [Header](#eds-header)

## <a name="general-notes"></a> General Notes

- All datasets containing physical quantities have an attribute *Unit* specifying the unit of the values. Note that this does not imply that different units might be use for a given dataset. The *Unit* attribute is only a hint to allow users to know the unit used in the dataset.
- All angles are expressed in _Radians_.
- Euler angles follow the Bunge convention ZXZ, i.e. +Z  +X'  +Z''.
- Each dataset defining Euler angles contains three columns, one for each Euler angle.
- All datasets and attributes of type H5T_STRING are encoded as UTF8 (see [H5T_CSET_UTF8](https://confluence.hdfgroup.org/display/HDF5/H5T_SET_CSET)).
- An _.h5oina_ file may not contain all the datasets specified in this specification. Different hardware and acquisition conditions mean that some parameters are not available, and therefore cannot be exported. The mandatory datasets are indicated below.

## <a name="data-tree"></a> AZtec Project Data Tree

Here is how the AZtec data are exported to _.h5oina_.
An AZtec project is normally structured using _Specimen_ and _Site_.
A _Site_ may contain one or more acquisitions like ED spectra, ED mappings, EDS mappings, EBSD mappings, electron images, etc.
If the user decides to export the whole project to _.h5oina_, an _.h5oina_ file would be created for each acquisition.
For instance, the following project would be exported as two _.h5oina_ files.
Note that the EDS and EBSD data of the _Map Analysis 1_ are stored in the same _.h5oina_ file and that _Electron Image 1_ is currently not exported.

AZtec project:

- Project 1
  - Specimen 1
    - Site 1
      - Electron Image 1
      - Map Analysis 1
        - EBSD Data
        - EDS Data
      - Map Analysis 2
        - EBSD Data

_.h5oina_ files:

- Project 1 - Specimen 1 - Site 1 - Map Analysis 1.h5oina
- Project 1 - Specimen 1 - Site 1 - Map Analysis 2.h5oina

## File Layout

### <a name="root-level"></a> Root Level Specification

Each file has the following datasets under the root level.

**Dataset Name** | **Mandatory** | **HDF5 Type** | **Dimension (row, column)** | **Comment**
--- | --- | --- | --- | ---
Manufacturer | | H5T_STRING | (1, 1) | i.e. _Oxford Instruments_
Software Version | | H5T_STRING | (1, 1) | Version of software used to create this file
Format Version | yes | H5T_STRING | (1, 1) | Version of this file format
Index | yes | H5T_STRING | (number of slices, 1) | List of the name of the slices (i.e. acquisitions) contained in this file.

### <a name="slice"></a> Slice Group Specification

Each Slice (i.e. acquisition) has its own H5G_GROUP with the Name of the group as the index of the slice.
The name of all the slices is given in the Index dataset.
At the moment, _.h5oina_ only supports one acquisition per file, so the file only contains one Slice, labelled _1_.

Within each Slice group there is at least one Technique group, representing the technique used for the acquisition.
A Slide group can contain several Technique groups.
The techniques can be, but not restrictive to:

**Group Name** | **Mandatory** | **Comment**
--- | --- | ---
[EBSD](#ebsd) | | Contains one EBSD acquisition
[EDS](#eds) | | Contains one EDS acquisition

### <a name="technique"></a> Technique Group Specification

Each technique group contains two groups with names **Data** and **Header**.

**Group Name** | **Mandatory** | **Comment**
--- | --- | ---
Data | yes | Contains all the data columns
Header | yes | Contains all the header entries

### <a name="common-header"></a> Common Header Group Specification

The following groups are common to the header group of the EBSD and EDS techniques.

**Group Name** | **Mandatory** | **Comment**
--- | --- | ---
[Stage Position](#stage-position) | | Contains datasets about the stage position of this acquisition

The following datasets are common to the header group of the EBSD and EDS techniques.

**Dataset Name** | **Mandatory** | **HDF5 Type** | **Dimension (row, column)** | **Comment**
--- | --- | --- | --- | ---
Acquisition Date | | H5T_STRING | (1, 1) | Acquisition date/time as ISO8601, yyyy-MM-ddTHH:mm:ss
Project Label | yes | H5T_STRING | (1, 1) | AZtec project name
Project Notes | | H5T_STRING | (1, 1) |
Project File | | H5T_STRING | (1, 1) |
Specimen Label | | H5T_STRING | (1, 1) | Label of specimen containing this acquisition
Specimen Notes | | H5T_STRING | (1, 1) |
Site Label | | H5T_STRING | (1, 1) | Label of site containing this acquisition
Site Notes | | H5T_STRING | (1, 1) |
Analysis Label | yes | H5T_STRING | (1, 1) | Label of this acquisition
Analysis Notes | | H5T_STRING | (1, 1) |
Analysis Unique Identifier | | H5T_STRING | (1, 1) | Unique identifier of this acquisition. It is unique across any AZtec project.
Magnification | | H5T_NATIVE_FLOAT | (1, 1) |
Beam Voltage | | H5T_NATIVE_FLOAT | (1, 1) | In kilovolts
Working Distance | | H5T_NATIVE_FLOAT | (1, 1) | Working distance of microscope (in millimeters)
Tilt Angle | | H5T_NATIVE_FLOAT | (1, 1) | Tilt angle of sample (either from stage tilt or pre-tilted holder)
Tilt Axis | | H5T_NATIVE_FLOAT | (1, 1) | 0 for x-axis, &pi;/2 for y-axis
X Cells | yes | H5T_NATIVE_INT32 | (1, 1) | Width of map in pixels
Y Cells | yes | H5T_NATIVE_INT32 | (1, 1) | Height of map in pixels
X Step | yes | H5T_NATIVE_FLOAT | (1, 1) | Step size along x-axis in micrometers
Y Step | yes | H5T_NATIVE_FLOAT | (1, 1) | Step size along y-axis in micrometers
Drift Correction | | H5T_NATIVE_HBOOL | (1, 1) | Whether drift correction was used during this acquisition

#### <a name="stage-position"></a> Stage Group Specification

The Stage Group contains the following datasets.

**Dataset Name** | **Mandatory** | **HDF5 Type** | **Dimension (row, column)** | **Comment**
--- | --- | --- | --- | ---
X | | H5T_NATIVE_FLOAT | (1, 1) | In millimeters
Y | | H5T_NATIVE_FLOAT | (1, 1) | In millimeters
Z | | H5T_NATIVE_FLOAT | (1, 1) | In millimeters
Tilt | | H5T_NATIVE_FLOAT | (1, 1) | Tilt angle of the stage in radians
Rotation | | H5T_NATIVE_FLOAT | (1, 1) | Rotation angle of the stage in radians

### <a name="ebsd"></a> EBSD Technique

#### <a name="ebsd-data"></a> Data Group Specification
The number of rows (first dimension of array) of all datasets is equal to the size of the acquisition (i.e. width x height).
In other words, it is equal to the total number of pixels in the acquisition.

The EBSD Data Group contains the following datasets.

**Dataset Name** | **Mandatory** | **HDF5 Type** | **Dimension (row, column)** | **Comment**
--- | --- | --- | --- | ---
Phase | yes | H5T_NATIVE_INT32 | (size, 1) | Index of phase, 0 if not indexed
X | | H5T_NATIVE_FLOAT | (size, 1) | X position in micrometers
Y | | H5T_NATIVE_FLOAT | (size, 1) | Y position in micrometers
Bands | | H5T_NATIVE_INT32 | (size, 1) | Number of bands positively indexed
Error | | H5T_NATIVE_INT32 | (size, 1) | Error code. Some of these codes are historical and no longer apply. NotAnalyzed=0, Success=1, NoSolution=2, LowBandContrast=3, LowBandSlope=4, HighMAD=5, UnexpectedError=6, Replaced=7
Euler | yes | H5T_NATIVE_FLOAT | (size, 3) | Orientation of Crystal (CS2) to Sample-Surface (CS1). See [Definition of Coordinate Systems](#coordinate-system) for more information.
Mean Angular Deviation | | H5T_NATIVE_FLOAT | (size, 1) | In radians
Band Contrast | | H5T_NATIVE_INT32 | (size, 1) |
Band Slope | | H5T_NATIVE_INT32 | (size, 1) |
Pattern Quality | | H5T_NATIVE_FLOAT | (size, 1) |
Pattern Center X | | H5T_NATIVE_FLOAT | (size, 1) | Pattern center X position scaled to the width of the image. This means that an X value of 0.5 is in the middle on the horizontal axis of the image. The origin is in the bottom left corner.
Pattern Center Y | | H5T_NATIVE_FLOAT | (size, 1) | Pattern center Y position scaled to the width of the image. Note that for a non-square image a Y value of 0.5 is _not_ in the center of the vertical axis of the image. The origin is in the bottom left corner.
Detector Distance | | H5T_NATIVE_FLOAT | (size, 1) | Detector distance scaled to the width of the image.
Beam Position X | | H5T_NATIVE_FLOAT | (size, 1) | X position of the beam in the real-world (in micrometers). The origin is in the center of the image, and a mathematical Y axis that is positive when going from bottom to top
Beam Position Y | | H5T_NATIVE_FLOAT | (size, 1) | Y position of the beam in the real-world (in micrometers). The origin is in the center of the image, and a mathematical Y axis that is positive when going from bottom to top

#### <a name="ebsd-header"></a> Header Group Specification

Apart from the [common header specification](#common-header), the EBSD Header Group contains the following groups.

**Group Name** | **Mandatory** | **Comment**
--- | --- | ---
Phases | yes | Contains a subgroup for each [phase](#ebsd-phase) where the name of each subgroup is the index of the phase starting at **1**.

Apart from the [common header specification](#common-header), the EBSD Header Group contains the following datasets.

**Dataset Name** | **Mandatory** | **HDF5 Type** | **Dimension (row, column)** | **Comment**
--- | --- | --- | --- | ---
Detector Orientation Euler | | H5T_NATIVE_FLOAT | (1, 3) | Orientation of Detector (CS3) to Microscope (CSm). See [Definition of Coordinate Systems](#coordinate-system) for more information.
Detector Insertion Distance | | H5T_NATIVE_FLOAT | (1, 1) | Insertion distance of EBSD detector in millimeters
Lens Distortion | | H5T_NATIVE_FLOAT | (1, 1) |
Lens Field View | | H5T_NATIVE_FLOAT | (1, 1) | In millimeters
Camera Binning Mode | | H5T_STRING | (1, 1) | For example, "4x4"
Camera Exposure Time | | H5T_NATIVE_FLOAT | (1, 1) | In milliseconds
Camera Gain | | H5T_NATIVE_FLOAT | (1, 1) |
Number Frames Averaged | | H5T_NATIVE_INT32 | (1, 1) |
Pattern Width | | H5T_NATIVE_INT32 | (1, 1) | Width of diffraction pattern images in pixels
Pattern Height | | H5T_NATIVE_INT32 | (1, 1) | Height of diffraction pattern images in pixels
Static Background Correction | | H5T_NATIVE_HBOOL | (1, 1) | Whether a static background correction was applied
Auto Background Correction | | H5T_NATIVE_HBOOL | (1, 1) | Whether an automatic background correction was applied
Hough Resolution | | H5T_NATIVE_INT32 | (1, 1) | &Delta;&rho; is equal to 1 / (2 * Hough Resolution + 1) and &Delta;&theta; is equal to &pi; / (2 * Hough Resolution + 1)
Band Detection Mode | | H5T_STRING | (1, 1) | Either _Center_ or _Edge_
Number Bands Detected | | H5T_NATIVE_INT32 | (1, 1) |
Indexing Mode | | H5T_STRING | (1, 1) | Either _Optimized - EBSD_, _Optimized - TKD_ or _Refined Accuracy_
Hit Rate | | H5T_NATIVE_FLOAT | (1, 1) | Hit rate, percentage of indexed pixels
Acquisition Time | | H5T_NATIVE_FLOAT | (1, 1) | In seconds
Specimen Orientation Euler | yes | H5T_NATIVE_FLOAT | (1, 3) | Orientation of Sample-Surface (CS1) to Sample-Primary (CS0). See [Definition of Coordinate Systems](#coordinate-system) for more information.
Scanning Rotation Angle | yes | H5T_NATIVE_FLOAT | (1, 1) | Angle between the specimen tilt axis and the scanning tilt axis in radians. If NaN, the angle is unknown.

##### <a name="ebsd-phase"></a> Phase Group Specification

Each phase group is defined by the following datasets.

**Dataset Name** | **Mandatory** | **HDF5 Type** | **Dimension (row, column)** | **Comment**
--- | --- | --- | --- | ---
Phase Name | yes | H5T_STRING | (1, 1) |
Reference | | H5T_STRING | (1, 1) |
Lattice Angles | yes | H5T_NATIVE_FLOAT | (1, 3) | Three columns for the alpha, beta and gamma angles in radians
Lattice Dimensions | yes | H5T_NATIVE_FLOAT | (1, 3) | Three columns for a, b and c dimensions in Angstroms
Laue Group | yes | H5T_NATIVE_INT32 | (1, 1) | Laue group index. The attribute **Symbol** contains the string representation, for example _m-3m_.
Space Group | | H5T_NATIVE_INT32 | (1, 1) | Space group index. The attribute **Symbol** contains the string representation, for example _P m -3 m_.
Number Reflectors | | H5T_NATIVE_INT32 | (1, 1) | Number of reflectors
Color | | H5T_NATIVE_UINT8 | (1, 3) | Three columns for the RGB values

#### <a name="coordinate-systems"></a> Definition of Coordinate Systems
An orientation is quantified by a set of rotations whereby one coordinate system (CS) is rotated to coincidence with another CS.
Depending on the application, it might be desirable to express an orientation relative to a specific CS.
Five coordinate systems are defined:

**CS** | **Name** | **Description**
--- | --- | ---
CSm | Microscope stage | Follows the naming convention from the stage (XY)
CS0 | Sample primary | For a rolled sheet of metal, this would be the rolling, tranverse, normal (RTN) system.In geology this could be the foliation, lineation system seen in some layered rocks.
CS1 | Sample surface | Surface from where we acquire the EBSD measurements
CS2 | Crystal | Follows the convention: Z parallel to c axis of the unit cell, X perpendicular to b and c axes of the unit cell and Y perpendicular to X and Z.
CS3 | EBSD detector | The EBSD detector is position sensitive, in the way that the EBSP changes as we change the position of the detector in the SEM.

The absolute crystal orientation is given by the orientation of the crystal (CS2) to the sample surface (CS1).

### <a name="eds"></a> EDS Technique ###

#### <a name="eds-data"></a> Data Group Specification ####
The number of rows (first dimension of array) of all datasets is equal to the size of the acquisition (i.e. width x height).
In other words, it is equal to the total number of pixels in the acquisition.

The EDS Data Group contains at least one of the following groups, but may also contain two or all three.

**Group Name** | **Mandatory** | **Comment**
--- | --- | ---
Window Integral | | Contains one dataset for each element and X-ray line analysed (e.g. Al Ka1). Each value (stored as H5T_NATIVE_FLOAT) corresponds to the integral of the raw X-ray counts over an energy window divided by the live time. The units are counts per second. The dataset has the attributes **Atomic Number** (H5T_NATIVE_INT32) and **X-ray Line** (H5T_STRING)
Peak Area | | Contains one dataset for each element and X-ray line analysed (e.g. Al K series). Each value (stored as H5T_NATIVE_FLOAT) corresponds to the fitted peak area divided by the live time. The units are counts per second. The dataset has the attributes **Atomic Number** (H5T_NATIVE_INT32) and **X-ray Line** (H5T_STRING)
Composition | | Contains one dataset for each element analysed (e.g. Al). Each value (stored as H5T_NATIVE_FLOAT) corresponds to the concentration, expressed in wt%. The dataset has the attribute **Atomic Number** (H5T_NATIVE_INT32).

The EDS Data Group contains the following datasets.

**Dataset Name** | **Mandatory** | **HDF5 Type** | **Dimension (row, column)** | **Comment**
--- | --- | --- | --- | ---
X | | H5T_NATIVE_FLOAT | (size, 1) | X position in micrometers
Y | | H5T_NATIVE_FLOAT | (size, 1) | Y position in micrometers
Live Time | yes | H5T_NATIVE_FLOAT | (size, 1) | In seconds
Real Time | | H5T_NATIVE_FLOAT | (size, 1) | In seconds

#### <a name="eds-header"></a> Header Group Specification ####

Apart from the [common header specification](#common-header), the EDS Header Group contains the following datasets.

**Dataset Name** | **Mandatory** | **HDF5 Type** | **Dimension (row, column)** | **Comment**
--- | --- | --- | --- | ---
Channel Width | yes | H5T_NATIVE_FLOAT | (1, 1) | Width of channel in electronvolt
Start Channel | yes | H5T_NATIVE_FLOAT | (1, 1) | Value of channel zero in electronvolt
Process Time | | H5T_NATIVE_INT32 | (1, 1) | Process time used
Number Frames | | H5T_NATIVE_INT32 | (1, 1) | Number of frames acquired during the acquisition
Energy Range | | H5T_NATIVE_FLOAT | (1, 1) | Energy range used in kiloelectronvolt
Number Channels | | H5T_NATIVE_INT32 | (1, 1) | Number of channels in spectrum
Detector Elevation | | H5T_NATIVE_FLOAT | (1, 1) | Angle of ED detector from XY plane in radians
Detector Azimuth | | H5T_NATIVE_FLOAT | (1, 1) | Angle between the tilt direction (90deg from tilt axis) and detector plane in radians
Detector Serial Number | | H5T_STRING | (1, 1) |
Detector Type Id | | H5T_NATIVE_INT32 | (1, 1) | Attribute **Name** gives friendly name of detector
Processor Type | | H5T_STRING | (1, 1) | Name of pulse processor
Window Type | | H5T_STRING | (1, 1) | Description of the window
Strobe FWHM | | H5T_NATIVE_FLOAT | (1, 1) | Full width half maximum of strobe in electronvolts
Strobe Area | | H5T_NATIVE_INT32 | (1, 1) | Number of counts in the strobe peak
Binning | | H5T_NATIVE_INT32 | (1, 1) | Binning factor from the original data
