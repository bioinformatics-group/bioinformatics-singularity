# bioinformatics-singularity
The software we use to run things in our lab. Some of the software is older than what is available to be consistent with other publications.

Among other things, we use:
 * R 3.5
 * BLAST 2.3.0+
 * tcoffee (current version from git)
 * eggnog-mapper (current version from git)
 
 
## eggnog-mapper
We have not included the contents of the `data` dir for `eggnog-mapper` and thus any commands using it should use `--data_dir` to specify where those very large files are. IF you need those files, you can use the download script provided by eggnog-mapper `/usr/bin/eggnog-mapper/download_eggnog_data.py`, but you'll still need to pass the directory where those files will end up via `--data_dir`. You'll want to set this up outside the singularity image. 

While eggnog-mapper can be found in `/usr/bin/eggnog-mapper'`, we have included a script in the image `emapper.sh` that can be run which is already in the standard path and which will pass arguments as appropriate.

### eggnog-mapper
Are you running this image at Concordia on speed? We suggest the following example call, which executes the test from their installation guide but maps to the locally stored version of that database:

````
singularity exec -B `pwd`:$PWD -B /speed-scratch/bioinformatics-group/datasets/eggnog-mapper:datasets /speed-scratch/bioinformatics-group/bioinformatics.simg emapper.sh --data_dir /datasets -i /usr/bin/eggnog-mapper/test/p53.fa --output p53_maNOG -m diamond
````

I suggest making the data files (approximately 40G) available to your images on as fast a disk as possible (I've seen putting it in `/dev/shm` suggested).