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
  /Volumes/data/images/MIOS \
  intj5@ssh.fsl.byu.edu:~/compute/images/MIOS/
