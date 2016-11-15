Mindboggle
==========

Installation
------------

Install and run Docker on your (macOS, Linux, or Windows) host machine:

https://docs.docker.com/engine/installation/

Clone the Mindboggle Docker app:

.. code-block:: bash

  git clone https://github.com/BIDS-Apps/mindboggle;
  mindboggle;

Edit Dockerfile (starting line 40):

.. code-block:: docker

  RUN curl -sSL http://neuro.debian.net/lists/vivid.us-ca.full | tee /etc/apt/sources.list.d/neurodebian.sources.list && \
  apt-key adv --recv-keys --keyserver hkp://pgp.mit.edu:80 0xA5D32F012649A5A9 && \
  apt-get update #&& \

Build mindboggle:

.. code-block:: bash

  docker build -t bids/mindboggle .;

Set the path on your host machine for the Docker container to access Mindboggle input and output directories:

.. code-block:: bash

  PATH_ON_HOST=/;
  docker run --rm -ti -v $PATH_ON_HOST:/root/data \
   --entrypoint /bin/bash bids/mindboggle;

Preprocessing
-------------

Mindboggle takes as its input preprocessed brain MR image data. Mindboggle currently takes output from FreeSurfer (v5.3 or higher recommended) and optionally from ANTs (v2.1.0rc3 or higher recommended; v2.1.0 is included in the Docker app).

**FreeSurfer** generates labeled cortical surfaces, and labeled cortical and noncortical volumes. Run recon-all on a T1-weighted image:

.. code-block:: bash

  ~/apps/freesurfer/bin/recon-all \
  -subjid 1004 \
  -i /fslhome/intj5/compute/images/MIOS/1004/t1/t1.nii.gz \
  -wsatlas \
  -all \
  -sd /fslhome/intj5/compute/analyses/MIOS/FreeSurferv530/

**ANTs** provides brain volume extraction, segmentation, and registration-based labeling. To generate the ANTs transforms and segmentation files used by Mindboggle, run the antsCorticalThickness.sh script on the same T1-weighted image:

Run Program
-----------

Set Freesurfer, ANTs, and Mindboggle paths:

.. code-block:: bash

  HOST=/root/data/Users/njhunsak/mindboggle/;
  FREESURFER_SUBJECT=$HOST/freesurfer/1004;
  ANTS_SUBJECT=$HOST/ants/1004;
  MINDBOGGLING=$HOST/mindboggling;
  MINDBOGGLED=$HOST/mindboggled;

Run mindboggle:

.. code-block:: bash

  mindboggle $FREESURFER_SUBJECT \
  --working $MINDBOGGLING \
  --out $MINDBOGGLED \
  --ants $ANTS_SUBJECT/BrainSegmentation.nii.gz
