# How To Read fMRIPrep visual Report

fMRIPrep generates HTML visual reports that allow you to check the results of the preprocessing steps that were applied to the data. 
Note: this is not a replacement for running quality control of your images before running fMRIPrep (FAQ link: https://fmriprep.readthedocs.io/en/stable/faq.html) 
Below is a guide to interpreting the information contained in the visual reports, organized by subsection of the report.

If you have examples of “bad” preprocessing results that you’re willing to share, please [submit them somewhere?] with a short description of the problem (for example, “bad skull stripping”).

When you open a report, it may be helpful to first jump to the [Errors subsection](LINK) to see whether fMRIPrep encountered any errors during preprocessing. Note that “errors” refers only to problems in running fMRIPrep, and NOT problems with the quality of the resulting images. This section won’t flag participants in which fMRIPrep was able to successfully run to completion but yielded poor results. 

## **Summary**

The first section of the report is a text summary, containing the subject ID, number of structural and functional images, task name, output spaces in which fMRIPrep generated your preprocessed data, and whether surface reconstruction with Freesurfer was run. 
It is good to check that all of this information is as expected.

## **Anatomical**

The reports first present results of anatomical preprocessing. 

### Anatomical conformation

* This section provides information about the structural images.
* Check for obvious failures, such as missing images or implausible values for voxel size, orientation, or dimensions.
    * For example: you’re expecting two input T1w images but it says “Input T1w images: 1”

### Brain mask and brain tissue segmentation of the T1w

The brain mask report shows the quality of intensity non-uniformity (INU) correction, skull stripping, and tissue segmentation.
* Good:
    * The brain mask (red) covers the whole brain and only the brain - not the dura or anything else outside the brain. It should closely follow the contour of the brain.
    * The outlines of the gray matter (magenta) and white matter (blue) segmentation are correctly drawn. The blue line should follow the boundary between gray and white matter, while the magenta line should outline ventricles.

* Bad:
    *  Intensity non-uniformity artifacts (looks like the “original image” panel in the figure below):
 
        ![inu](../assets/fmriprep_visual_report/inu.png) 
        *from Vovk, U., Pernus, F., & Likar, B. (2007). A review of methods for correction of intensity inhomogeneity in MRI. IEEE transactions on medical imaging, 26(3), 405-421.*
 
 
    * Skull-stripping failed leading to an inaccurate brain mask
        * The brain mask cuts off part of the brain and/or contains holes surrounding signal drop-out regions.
        * The brain mask includes parts that are clearly NOT brain.
            * For example, fMRIPrep might have misclassified part of the dura, epidural space, or skull as belonging inside the brain mask; as a result the brain mask presents “bumps” surrounding high-intensity areas of signal outside of the cortex. 

        * Having an accurate brain mask makes the downstream preprocessing of an fMRI scan faster (excluding voxels of non-interest) and more accurate (less bias from voxels of non-interest). Consequently, it is important to discard subjects for which the brain mask is not well defined.
    
    * The gray matter (magenta)/white matter (blue) outlines don’t match where those tissue classes are distributed in the underlying image, so either the blue line does not follow the boundary between gray and white matter or the magenta line does not outline ventricles.
    * Inclusion of tissues other than the tissue of interest in the contour delineations should lead to exclusion of the scan.
    * T1w scans showcasing a low signal-to-noise ratio because of thermal noise will present scattered misclassified voxels within piecewise-smooth regions (generally more identifiable in the WM and inside the ventricles). 
        * These scans should be excluded except for images where these voxels are only present at subcortical structures (e.g., thalamus) or nearby tissue boundaries. In the latter case, the misclassification results from partial volume effects (i.e., indeed, such voxels contain varying fractions of two or more tissues). The figure below illustrates the difference between individual dots caused by noise versus partial volume effects.
        ![noise-in-segmentation](../assets/fmriprep_visual_report/noise-in-segmentation.svg) 
        *Fig. 2. Error in brain tissue segmentation of T1w images. (A) The presence of noise compromises the segmentation leading to single voxels being excluded from the ventricle mask. The subject has thus been excluded from further analysis. (B) A series of spots are visible at the boundary between WM and GM. Those spots are due to partial volume effect and thus is a flaw of the fMRIPrep segmentation implementation not of the image quality.*

* Common pitfalls in interpretation:
    * At the inter-hemispheric space, masks (and in particular the brain mask, despite its smooth edges) may intersect the visualization plane several times, giving the impression that the mask is cutting off brain regions. However, this is more of a visual effect on the cutting plane.

### Spatial normalization of the anatomical T1w reference

The normalization report shows how successfully your T1w image(s) were resampled into standard space, for each of the template spaces used.
* Good:
    * Your T1w image and the template image line up well when you toggle between the images (hover mouse over the panel):
        * In order of importance, the following structures should be correctly aligned : 1. ventricles, 2. subcortical regions, 3. corpus callosum, 4. cerebellum, 5. cortical gray matter.
        * The standard templates provided with fMRIPrep (such as `MNI152NLin2009cAsym`) are averaged across multiple subjects, so the template image will look blurrier than the T1w image. Because of this averaging, it is normal for sulci to be less pronounced and gyri to be wider on the template than the T1w image.

* Bad:
    * Stretching or distortion in the participant’s T1w image, indicating failed normalization.
        * A misalignment of the ventricles, the subcortical regions, or the corpus callosum should lead to immediate exclusion. You can however be more lenient with the misalignment of cortical GM because volumetric (image) registration may not resolve substantial inter-individual differences (e.g., a sulcus missing in an individual’s brain but typically present in the population of the template).

    * If skull-stripping was not successful, you might see some places where there are non-brain voxels outside of the contours of the brain. 


### Surface reconstruction

If you used the `--fs-no-reconall` flag to skip surface-based preprocessing, this section of the report will not exist.
The FreeSurfer [fischl2012] [1] subject reconstruction report shows the white matter (blue outline) and pial (red outline) surfaces overlaid on the T1w image.
* Good:
    * The white-gray boundary outlined (blue link) matches the underlying image.
    * White matter and pial surface boundary outlines do not cross or overlap each other.

* Bad:
    * The white-gray boundary outline (blue line) does not correspond well to the boundary observed in the underlying image.
    * White matter and pial surface boundaries cross or overlap each other.
    * Pial surface (red outline) extends past the actual pial boundary (see images [here](https://surfer.nmr.mgh.harvard.edu/fswiki/FsTutorial/PialEdits_tktools) for an example ; this can be a result of bad skull-stripping).

* Common pitfalls in interpretation:
    * Note that cerebellum and brainstem are excluded from the surface reconstruction in fMRIPrep; it is thus normal that the outlines do not include these areas.
 
## **Functional**

### Textual summary

* The BOLD summary describes the interpretation of the inputs by fMRIPrep, and the applied heuristics. For instance, if the dataset is multi-echo EPI and comprehends three or more echos, then fMRIPrep should indicate this in the BOLD summary.

* Other potential issues are missing images or implausible values.
    * Functional data can vary based on available data, metadata, user selections.
        * What does “Heuristics - BBR may fall back to volume-based coregistration” mean?
        In fMRIPrep’s workflow, if boundary-based registration fails (meaning that `bbr` distorts the affine registration more than 15mm), fMRIPrep reverts to using the initial affine transform from Freesurfer’s `mri_coreg` tool instead of using the bbr refinement of the initial transform ([relevant code](https://github.com/poldracklab/fmriprep/pull/694/files#diff-939042e0a10e509ce5708c3e112376ffR29)) 
        “Note on orientation: qform matrix overwritten/ this data has been copied from sform/sform matrix set” warnings
        `qform` and `sform` refer to metadata fields in the NIfTI file header, prescribing a coordinate system for the image. 
        This message warns you that information in the header was altered during the preprocessing.
        This is an advisory message and does not necessarily indicate issues in data quality or registration.
        A common scenario is that the sform code was 0, so fMRIPrep copied the code from qform in order to keep the affine matrices aligned, ensuring that the images will be treated as having the same orientation later down the pipeline.
        See Chris Markewicz’s Neurostars posts [here](https://neurostars.org/t/note-on-orientation-qform-and-sform-warning/4379/4) and [here](https://neurostars.org/t/note-on-orientation-sform-matrix-set/5939) for a longer explanation of when/why fMRIPrep produces these warnings.
        You can read more about how `qform` and `sform` work at the following links: [Recommended usage of qform and sform](https://nifti.nimh.nih.gov/nifti-1/documentation/nifti1fields/nifti1fields_pages/qsform_brief_usage) and [Nifti Qform and Sform](http://gru.stanford.edu/doku.php/mrTools/coordinateTransforms). 

### Susceptibility distortion correction

* `--use-syn-sdc`
* `--force-syn`
* `--ignore fieldmaps`

The functional images can have some warping or distortion due to inhomogeneities in the magnetic field. (fMRIPrep has several options to try to estimate the displacement map)[https://fmriprep.readthedocs.io/en/stable/sdc.html], based either on command-line arguments or automatic selection based on the available data). 
This section of the report shows the participant’s functional images before and after distortion correction, with the GM/WM boundary from the anatomical images overlaid as reference.
* Good:
    * The distortion-corrected (or “after”) image shows improved alignment between the brain and the GM/WM boundary, compared to the “before” image.
    * The shape of the brain in the distortion-corrected image is not .

* Bad:
    * Alignment between the brain and the GM/WM boundary is worse in the “after” compared to the “before” image.
    * The brain in the “after” image looks more distorted, stretched, smeared or otherwise misshapen, indicating that distortion correction was not successful. 
 
### Alignment of functional and anatomical data

The text tells you which method fMRIPrep used to align the functional and anatomical data - for example, by running `bbregis
ter`. The images show the registered BOLD reference with the white and pial surfaces overlaid (red and blue lines)
* Good:
    * The BOLD and T1w images are aligned; image boundaries and anatomical landmarks (for example, ventricles; corpus callosum) appear to be in the same place when toggling between the images.
    * White and pial surface outlines (red and blue lines) appear to correspond well to the tissue boundaries in the functional images. 

* Bad:
    * Functional and anatomical images are not aligned and clearly differ in their spatial location or orientation.
    * White and pial surface boundaries (red and blue lines) overlays correspond poorly to the tissue boundaries in the underlying images.
    * Some drop out in inferior brain regions (such as the OFC or medial temporal lobe) in functional images is somewhat inevitable. Depending on the type of analysis you’re doing and what you’re interested in, this may be more/less of an issue for you. If you see a lot of signal drop out, you will probably want to check the mask generated by fMRIPrep to see how that drop-out impacts the location and quantity of missing voxels in your mask.

* Note that the BOLD images displayed in the report may have what appears to be an “artifact” in the data, as shown below. This is because the reports use a faster but less precise type of interpolation (Nearest Neighbor interpolation) for display. In actuality, fMRIPrep uses Lanczos interpolation for resampling, so these apparent “artifacts” are not present in the actual data (you can confirm this by checking the preprocessed BOLD NIFTI file and its registration to the T1w image in your preferred software). 

    ![segmentation](../assets/fmriprep_visual_report/segmentation.png)

### Brain mask and temporal/anatomical CompCor ROIs

BOLD brain mask with the aCompCor and tCompCor masks
The aCompCor mask (magenta outline) is a conservative CSF and white-matter mask for extracting physiological and movement confounds. 
The tCompCor mask (blue outline) contains the top 5% most variable voxels within a heavily-eroded brain-mask.
* Good:

* Bad:

### Variance explained by t/a CompCor components

Explained in more depth by <https://fmriprep.readthedocs.io/en/stable/outputs.html#confounds-and-carpet-plot-on-the-visual-reports> but may be useful to duplicate some of this information here?
 
### BOLD summary

The BOLD summary report shows several characteristic statistics along with a carpetplot (Power, 2016), giving a view of the temporal characteristics of the preprocessed BOLD series.
* Good:
* Bad:
Note on how variance and scaling affects this display? (in other words, spikes are relative to other datapoints in the timeseries)
* Definitions: 
    * GS
    * GSCSF
    * GSWM
    * DVARS
        * *standardized DVARS!* <https://neurostars.org/t/fmriprep-standardised-dvars/5271> 
    * FD

### Correlations among nuisance regressors

This may also be addressed in more depth by the confounds and regressors documentation (link to page)
Can be used to diagnose partial volume effects
PVE indicated by high correlations of a confound time series (or several) with the global signal

### ICA-AROMA
What you’re looking at:
* Glass brain
* Map with time on the top and frequency on the bottom
* 1) the spatial map, 2) the time course, and 3) the power spectrum of the time c
* Artifact components (red) should have more high-frequency noise whereas signal components (green) should have their peak in the   lower-frequency range
* Artifacts you might see:
    * Multi-band artifacts in ICA-AROMA: <https://neurostars.org/t/potential-issue-in-ica-aroma-report/4231>
        * Note on ICA-AROMA and multiband data 
* Slice-timing artifacts in ICA-AROMA: <https://neurostars.org/t/potential-issue-in-ica-aroma-report/4231/6> 
* Link to documentation on confounds: <https://fmriprep.readthedocs.io/en/stable/outputs.html#confounds>

## About
 
## Methods
 
## Errors

This section tells you whether fMRIPrep encountered any problems during the preprocessing. Note that “errors” refers only to problems in running fMRIPrep, and NOT problems with the quality of the resulting images. This section won’t flag participants in which fMRIPrep was able to successfully run to completion but yielded poor results, or in which the input data was of lower-than-desirable quality.

## Things that aren’t an issue:
* “Missing” [not visualized] slices
    * In the report visualizations that display a panel of slices, sometimes it will appear as though there is a slice missing - for example, nothing is shown for x = -12. This is a problem with the visualization and does NOT indicate the slice is missing from the actual data. You can try reloading the report, or opening it with Chrome if you’re using another browser, as well as using your preferred NIFTI viewer to check the data. 