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
    cd /Volumes/data/images/MIOS/$i/
    pwd
    unzip -o DICOMs/other.zip '*spgr*' -d DICOMs/
    unzip -o DICOMs/other/zip '*t2*' -d DICOMs/
    unzip -o DICOMs/other.zip '*flair*' -d DICOMs/
    unzip -o DICOMs/epi.zip '*dti*' -d DICOMs/
  done

DICOM to NiFTI
--------------

.. code-block:: bash

  ~/Applications/dcm2niix/bin/dcm2niix \
  -o ${subjDir}/t1/ \
  -x y \
  -f t1 \
  ${subjDir}/DICOMs/
