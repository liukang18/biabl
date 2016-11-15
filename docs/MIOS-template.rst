Template
========

Copy preprocessed (insert reference) images for template construction:

.. code-block:: bash

   mkdir -p ~/templates/MIOS
   for i in $(ls ~/compute/images/MIOS/); do
   cp -v ~/compute/images/MIOS/$i/t1/t1.nii.gz ~/templates/MIOS/img_${i}.nii.gz;
   done


Initial Template
----------------

Create a job script:

.. code-block:: bash

	vi /fslhome/intj5/scripts/MIOS/template/initial.sh

Copy and paste code:

.. code-block:: bash

	#!/bin/bash

	#SBATCH --time=02:00:00   # walltime
	#SBATCH --ntasks=1   # number of processor cores (i.e. tasks)
	#SBATCH --nodes=1   # number of nodes
	#SBATCH --mem-per-cpu=16384M  # memory per CPU core

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
	cd ~/templates/MIOS/
	~/apps/ants/bin/buildtemplateparallel.sh \
	-d 3 \
	-m 1x0x0 \
	-o pt1 \
	-c 5 \
	img*.nii.gz

Submit job script:

.. code-block:: bash

	var=`date +"%Y%m%d-%H%M%S"`
	mkdir -p ~/logfiles/${var}
	sbatch \
	-o ~/logfiles/${var}/output-initial.txt \
	-e ~/logfiles/${var}/error-initial.txt \
	/fslhome/intj5/scripts/MIOS/template/initial.sh

Template
--------

Create a job script:

.. code-block:: bash

	vi /fslhome/intj5/scripts/MIOS/template/template.sh

Copy and paste code:

.. code-block:: bash

	#!/bin/bash

	#SBATCH --time=10:00:00   # walltime
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
	cd ~/templates/MIOS
	~/apps/ants/bin/buildtemplateparallel.sh \
	-d 3 \
	-z ~/templates/MIOS/pt1template.nii.gz \
	-o pt2 \
	-c 5 \
	img*.nii.gz

Submit job script:

.. code-block:: bash

	var=`date +"%Y%m%d-%H%M%S"`
	mkdir -p ~/logfiles/${var}
	sbatch \
	-o ~/logfiles/${var}/output-template.txt \
	-e ~/logfiles/${var}/error-template.txt \
	/fslhome/intj5/scripts/MIOS/template/template.sh

Align Template
--------------

For whatever reason, the population template was not at all aligned when it was created, so I rigidly aligned it to the NKI template. Trying the run the image through acpcdetect absolutely didn't work.

.. code-block:: bash

  ~/apps/ants/bin/antsRegistrationSyNQuick.sh \
  -d 3 \
  -f ~/templates/NKI/T_template.nii.gz \
  -m ~/templates/MIOS/pt2template.nii.gz \
  -o ~/templates/MIOS/aligned_ \
  -t r

Clean Up Directory
------------------

.. code-block:: bash

  cd ~/templates/MIOS
  mkdir data
  mv img*.nii.gz data/
  mv aligned_Warped.nii.gz template.nii.gz
  find . \( ! -name "data" ! -name "img*.nii.gz" ! -name "template.nii.gz" \) -exec rm -rf {} \;

Tissue Segmentation
-------------------
