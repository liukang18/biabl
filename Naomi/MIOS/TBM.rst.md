Template
========

Copy images for template construction

.. highlight:: bash
	mkdir -p ~/templates/MIOS
	for i in $(ls ~/compute/images/MIOS/); do
	cp -v ~/compute/images/MIOS/$i/t1/t1.nii.gz ~/templates/MIOS/img_${i}.nii.gz;
	done

Build initial template

.. highlight:: bash
	#!/bin/bash

	#SBATCH --time=09:00:00   # walltime
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

Build high-resolution template