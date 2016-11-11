Mindboggle
==========

Installing Mindboggle
---------------------

1. Install and run Docker on your (macOS, Linux, or Windows) host machine:

   https://docs.docker.com/engine/installation/


Initial Template
----------------

Create a job script

.. code-block:: bash

	vi /fslhome/intj5/scripts/MIOS/template/initial.sh

Copy and paste code::

	#!/bin/bash

	#SBATCH --time=03:00:00   # walltime
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

Submit job script::

	var=`date +"%Y%m%d-%H%M%S"`
	mkdir -p ~/logfiles/${var}
	sbatch \
	-o ~/logfiles/${var}/output-initial.txt \
	-e ~/logfiles/${var}/error-initial.txt \
	/fslhome/intj5/scripts/MIOS/template/initial.sh

Template
--------

Create a job script::

	vi /fslhome/intj5/scripts/MIOS/template/template.sh

Copy and paste code::

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

Submit job script::

	var=`date +"%Y%m%d-%H%M%S"`
	mkdir -p ~/logfiles/${var}
	sbatch \
	-o ~/logfiles/${var}/output-initial.txt \
	-e ~/logfiles/${var}/error-initial.txt \
	/fslhome/intj5/scripts/MIOS/template/template.sh

