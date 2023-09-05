### Brain Network Construction (FC& SC)
---
#### Requirements
+ `Singularity`: **Image container** suitable for HPC environment, where users usually do not have *root* access.
   + Installation: 
      1. Create a new conda environment: `conda create -n singularity python=3.10`
      2. Activate the environment: `conda activate singularity`
      3. Install singularity: `conda install -c conda-forge singularity`
---
+  `smriprep`: **Preprocessing** for structural MRI data. It performs basic processing steps (subject-wise averaging, B1 field correction, spatial normalization, segmentation, skullstripping etc.) 
   + Installation:  
   +   ```
        singularity build /my_images/smriprep.simg \
                            docker://nipreps/smriprep:<version>
       ```
       where `<version>` is the version of the image to download. (version information can be found [here](https://github.com/nipreps/smriprep/releases))
 ---      
+ `fmriprep`:**Preprocessing** for functional MRI data. It performs basic processing steps (slice-timing correction, motion correction, spatial normalization, segmentation, skullstripping etc.) 
   + Installation:  
   +   ```
        singularity build /my_images/fmriprep.simg \
                            docker://nipreps/fmriprep:<version>
       ```
       where `<version>` is the version of the image to download. (version information can be found [here](https://github.com/nipreps/fmriprep/releases))
---
+ `qsiprep`:**Preprocessing** and **Reconstruction** for DWI data.
    + Installation:
      + `singularity build qsiprep.sif docker://pennbbl/qsiprep:<version>`

---
+ `MRIQC` extracts no-reference IQMs (image quality metrics) from structural (T1w and T2w) and functional MRI (magnetic resonance imaging) data.
    + Installation:
      + `singularity build mriqc.sif docker://poldracklab/mriqc:<version>`

+ `Clinica` [optional]: Converts some famous datasets (ADNI, AIBL, OASIS, etc.) to BIDS format.
    + Installation:
      + `pip install clinica`
+ `hcp2bids` [optional]: Converts HCP dataset to BIDS format.
    + Installation:
      + ```
        git clone https://github.com/niniko1997/hcp2bids.git
        ```
---
#### Usage

##### Make sure that your input data is in BIDS format. 
  + If not, check whether your data is in one of the following databases:
    + ADNI
    + AIBL
    + HABS
    + NIFD
    + OASIS3
    + OASIS
    + UK Biobank
    + HCP 
    + use `Clinica` or `hcp2bids` to convert above data to BIDS format. 
  + If the data is not in any of the above databases, use `dcm2bids` or `HeudiConv`to convert your data to BIDS format. 
    + Installation:
      + `conda install -c conda-forge dcm2bids`
      + `pip install heudiconv`(  `dcm2niix` is required for local `heudiconv`) or `singularity pull docker://nipy/heudiconv:latest`
    + Usage:
      + Please refer to [HeuDiConv](https://heudiconv.readthedocs.io/en/latest/) and [dcm2bids](https://unfmontreal.github.io/Dcm2Bids/docs/tutorial/first-steps/) because my test datasets are OASIS-1 and a public dataset avaliable on [OpenNeuro](https://openneuro.org/datasets/ds002843/versions/1.0.1) which are already in BIDS format.
  + If the data is not in any of the above databases and the data is not in dcm format, usually there are 2 situations:
    1.  The data includes nifti files and corresponding sidecar files, such as **['.json', '.tsv', '.tsv.gz', '.bval', '.bvec']**. Refer to [nibabel2bids-BIDScoin](https://bidscoin.readthedocs.io/en/stable/options.html#nibabel2bids-plugin)
    2.  The data contains only nifti files. You may need to find the protocol of data. And even then, you may need to write your own converter, which is difficult. Refer to [neurostars1](https://neurostars.org/t/reorganizing-nifti-to-bids/1147/16),[neurostars2](https://neurostars.org/t/niifti-conversion-to-bids/22909/5)

---
##### MRIQC

+ First Level QC (participant)
  + `mriqc bids-root/ output-folder/ participant` (all the subjects in the bids-root folder will be processed)
  + `mriqc bids-root/ output-folder/ participant --participant-label S01 S02 S03`(**bids-root/sub-S01**,**bids-root/sub-S02**, and **bids-root/sub-S03** will be processed)
+ Second Level QC (group)
  + `mriqc bids-root/ output-folder/ group`
+ DWI NOT SUPPORTED!  [https://github.com/nipreps/mriqc/issues/1125]
---
##### smriprep
+ ```
    #!/bin/bash
    singularity run --cleanenv /biolab/zhouzx/Code/simg/smriprep/smriprep.simg \
        /biolab/zhouzx/Code/Data/hcp-bids /biolab/zhouzx/Code/Data/hcp-bids/outputs \
        participant \
        --participant-label 996782 \
        --fs-license-file /biolab/zhouzx/freesurfer/freesurfer/license.txt \
        -w  /biolab/zhouzx/Code/Data/ds002843-processed/work \
        --write-graph \
        -vv \
        --nprocs 64 \
        --omp-nthreads 48
  ```
+ Default output space: `standard space`, use `--output-spaces` to specify other output spaces. Standard spaces will be extracted for spatial normalization.
+ **Outputs**:
  + ![Alt text](fig/t1w_flow.png)
+ **Other Options** see [here](https://www.nipreps.org/smriprep/usage.html)
---
##### fmriprep
+ ```
    #!/bin/bash
    export SINGULARITYENV_FS_LICENSE=/biolab/zhouzx/freesurfer/freesurfer/license.txt
    singularity run --cleanenv fmriprep/fmriprep-23.1.3.simg \
        /biolab/zhouzx/Code/Data/ds002843-download /biolab/zhouzx/Code/Data/ds002843-processed \
        participant \
        --participant-label dmp0001 \
        -w  /biolab/zhouzx/Code/Data/ds002843-processed/work \
        --write-graph \
        --nthreads 64 \
        --omp-nthreads 48
  ```
+ Default output space: `MNI152NLin2009cAsym`, which is different from smriprep.  use `--output-spaces` to specify other output spaces. Standard spaces will be extracted for spatial normalization.
+ If smri data has been processed by smriprep, use `--fs-subjects-dir` to specify the freesurfer output directory.
---
##### Nilearn
+ `nilearn` is a python package for statistical learning on neuroimaging data. It provides easy access to a large variety of imaging algorithms within Python. We will use it to extract the time series of each ROI and calculate the functional connectivity matrix.
+ See (https://github.com/brainhack-school2020/stong3_fMRI_processing/blob/master/fMRIPrep_tutorial/fMRIPrep%20pre-processing%20and%20post-processing%20-%20connectivity%20matrices%20extraction%20using%20a%20parcellation.ipynb)

##### QSIPrep
+ `qsiprep` is much easier to use because it has included both **preprocessing** and **reconstruction**.
+ 
    ```
        #!/bin/bash
        singularity run --cleanenv /biolab/zhouzx/Code/simg/qsiprep/qsiprep.sif \
        /biolab/zhouzx/Code/Data/ds002843-download /biolab/zhouzx/Code/Data/ds002843-processed \
        participant \
        --participant-label dmp0188 \
        --fs-license-file /biolab/zhouzx/freesurfer/freesurfer/license.txt \
        -w  /biolab/zhouzx/Code/Data/ds002843-processed/work \
        --write-graph \
        --freesurfer-input /biolab/zhouzx/Code/Data/ds002843-processed/sourcedata/freesurfer/sub-dmp0188 \
        --output-resolution 1.2 \
        -v
    ```
--- 
##### Notes and TODOs
+ the default included atlases in `qsiprep` are schaefer100x7, schaefer100x17, schaefer200x7, schaefer200x17, schaefer400x7, schaefer400x17, brainnetome246, aicha384,gordon333, aal116, power264.
+ the default included atlases in `nilearn` are aal, allen_2011, basc_multiscale_2015, craddock_2012, destrieux_2009, harvard_oxford, juelich, schaefer_2018, smith_2009, talairach, yeo_2011 and so on.
+ Therefore, if you want to use other atlases, you need to download the **nii files** and **label files**. [site1](https://www.lead-dbs.org/helpsupport/knowledge-base/atlasesresources/cortical-atlas-parcellations-mni-space/) [site2](https://github.com/neurodata/neuroparc)
+ Multiple subjects task is not tested. But it should be ok by just replacing the `--participant-label xxx` with `--participant-label ${PARTICIPANT_LABEL}`, where `PARTICIPANT_LABEL ="0001 0004 0005"` .

---
##### Useful Links
+ [Andy's Brain Book](https://andysbrainbook.readthedocs.io/en/latest/)
+ [The Princeton Handbook for Reproducible Neuroimaging](https://brainhack-princeton.github.io/handbook/index.html#the-princeton-handbook-for-reproducible-neuroimaging)
+ [NeuroStars](https://neurostars.org/)