Template
========

Copy images for template construction

.. code-block:: bash
	mkdir -p ~/templates/MIOS
	for i in $(ls ~/compute/images/MIOS/); do
	cp -v ~/compute/images/MIOS/$i/t1/t1.nii.gz ~/templates/MIOS/img_${i}.nii.gz;
	done

Build initial template