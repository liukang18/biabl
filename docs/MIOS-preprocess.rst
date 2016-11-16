Preprocess
==========

Download Files
--------------

Download the latest files using sftp:

.. code-block:: bash

  sftp njhunsak@128.187.76.2
  get -r /00-mios/scans/archive/2154--2016-10-24/

Rename directories:

.. code-block:: bash

  for i in $(ls /Volumes/data/images/MIOS/); do
    mv $i/ ${i:0:4}/;
  done

Convert zip to tar
------------------

If you don't have the program installed on your computer:

.. code-block:: bash

    pip install ruamel.zip2tar

The zip2tar program converts zipped files into tar files with no intermediate files created:

.. code-block:: bash

  for i in $(find /Volumes/data/images/MIOS/ -type f -name "*.zip"); do
    cd $(dirname $i)
    pwd
    zip2tar --gz $(basename $i)
    rm $(basename $i)
  done

Extract DICOMs
--------------

.. code-block:: bash

  for i in $(ls /Volumes/data/images/MIOS/); do
    cd /Volumes/data/images/MIOS/$i/DICOMs
    pwd
    tar -zxvf other.tar.gz other/*spgr*
    tar -zxvf other.tar.gz other/*t2*
    tar -zxvf other.tar.gz other/*flair*
    tar -zxvf epi.tar.gz epi/*dti*
  done

DICOM to NiFTI
--------------

Convert T1 image:

.. code-block:: bash

  for i in $(find /Volumes/data/images/MIOS/ -type d -name "*spgr*"); do
    cd $(dirname $(dirname $(dirname $i)))
    mkdir str
    ~/Applications/dcm2niix/bin/dcm2niix -o str/ -x y -f t1 -z n $i
  done

Convert T2 image:

.. code-block:: bash

  for i in $(find /Volumes/data/images/MIOS/ -type d -name "*t2*"); do
    cd $(dirname $(dirname $(dirname $i)))
    ~/Applications/dcm2niix/bin/dcm2niix -o str/ -x y -f t2 -z y $i
  done

Convert FLAIR image:

.. code-block:: bash

  for i in $(find /Volumes/data/images/MIOS/ -type d -name "*flair*"); do
    cd $(dirname $(dirname $(dirname $i)))
    ~/Applications/dcm2niix/bin/dcm2niix -o str/ -x y -f flair -z y $i
  done

Convert DW image:

.. code-block:: bash

  for i in $(find /Volumes/data/images/MIOS/ -type d -mindepth 3 -name "dti"); do
    cd $(dirname $(dirname $(dirname $i)))
    mkdir dti
    /Applications/MRIcron/dcm2nii64 -a y -g y -n y -x y -o dti/ $i/*
    cd dti/
    mv *.bval dwi.bval
    mv *.bvec dwi.bvec
    rm x*.nii.gz
    mv *.nii.gz dwi.nii.gz
  done

Sync Data
---------

.. code-block:: bash

  rsync -rauv \
  --exclude=".*" \
  --exclude="DICOMs" \
  /Volumes/data/images/MIOS/ \
  intj5@ssh.fsl.byu.edu:~/compute/images/MIOS/

Job Script
----------

Create script:

.. code-block:: bash

  vi ~/scripts/MIOS/preprocess_job.sh

Copy and paste code:

.. code-block:: bash

  #!/bin/bash

  #SBATCH --time=00:15:00   # walltime
  #SBATCH --ntasks=1   # number of processor cores (i.e. tasks)
  #SBATCH --nodes=1   # number of nodes
  #SBATCH --mem-per-cpu=32768M  # memory per CPU core

  # COMPATABILITY VARIABLES FOR PBS. DO NO DELETE.
  export PBS_NODEFILE=`/fslapps/fslutils/generate_pbs_nodefile`
  export PBS_JOBID=$SLURM_JOB_ID
  export PBS_O_WORKDIR="$SLURM_SUBMIT_DIR"
  export PBS_QUEUE=batch
  export OMP_NUM_THREADS=$SLURM_CPUS_ON_NODE

  # LOAD ENVIRONMENTAL VARIABLES
  var=`id -un`
  export ANTSPATH=/fslhome/${var}/apps/ants/bin/
  PATH=${ANTSPATH}:${PATH}

  # INSERT CODE, AND RUN YOUR PROGRAMS HERE
  DATA_DIR=~/compute/images/MIOS/${1}/
  ~/apps/art/acpcdetect -M -o ${DATA_DIR}/str/acpc.nii -i ${DATA_DIR}/str/t1_Crop_1.nii
  ~/apps/ants/bin/N4BiasFieldCorrection -v -d 3 -i  ${DATA_DIR}/str/acpc.nii -o ${DATA_DIR}/str/n4.nii.gz -s 4 -b [200] -c [50x50x50x50,0.000001]
  ~/apps/c3d/bin/c3d ${DATA_DIR}/str/n4.nii.gz -resample-mm 1x1x1mm -o ${DATA_DIR}/str/resampled.nii.gz

Batch Script
------------

Create script:

.. code-block:: bash

  vi ~/scripts/MIOS/preprocess_batch.sh

Copy and paste code:

.. code-block:: bash

  #!/bin/bash

  for subj in $(ls ~/compute/images/MIOS/); do
  sbatch \
  -o ~/logfiles/${1}/output_${subj}.txt \
  -e ~/logfiles/${1}/error_${subj}.txt \
  ~/scripts/MIOS/preprocess_job.sh \
  ${subj}
  sleep 1
  done

Submit Job
----------

.. code-block:: bash

  var=`date +"%Y%m%d-%H%M%S"`
  mkdir -p ~/logfiles/$var
  sh ~/scripts/MIOS/preprocess_batch.sh $var

Sync Data
---------

.. code-block:: bash

  rsync \
  -rauv \
  intj5@ssh.fsl.byu.edu:~/compute/images/MIOS/ \
  /Volumes/data/images/MIOS/
