# bioinformatics-singularity
The software we use to run things in our lab. Some of the software is older than what is available to be consistent with other publications.

To run it with out pre-built image, you just call:

```singularity shell https://tootsuite.encs.concordia.ca/singularity-images/bioinformatics-singularity.simg```

Binaries are made available in ```/usr/bin``` so you can just run things like ```R``` or ```t_coffee```

Among other things, we use:
 * R 3.5
 * BLAST 2.3.0+
 * tcoffee (current version from git)


## Running on Concordia's HPC infrastructure
If you're at Concordia and have requested access to speed (rt-ex-hpc), then you may want to be running jobs here. You can readily use this image, as we keep a local copy in ```/speed-scratch/bioinformatics-group/bioinformatics-singularity.simg```. In that case you can go to your working directory where you have your expected script and just run it. Keep in mind that speed likes you to use tcsh, but you're running bash from within the image.

For example, I can make/go to my working directory
```bash
mkdir -p /speed-scratch/{$uid}/test3
cd /speed-scratch/{$uid}/test3
```
Then I can create a file test.sh:
```bash
#!/bin/bash
ls -latr
makeblastdb -version
```

and give it appropraite permissions to run:
```bash
chmod 700 test.sh
```

Finally, I run the image with singularity:
```bash
singularity exec -B /nfs/speed-scratch/{$uid}:/speed-scratch/{$uid} /speed-scratch/bioinformatics-group/bioinformatics-singularity.simg ./test.sh
```

I get the expected output that shows my directory contents and the version of ```makeblastdb```
```
total 8
drwxrwxr-x 6 sthiel sthiel 4096 Oct 17 10:54 ..
-rwx------ 1 sthiel sthiel   43 Oct 17 11:41 test.sh
drwxrwx--- 2 sthiel sthiel 4096 Oct 17 11:42 .
makeblastdb: 2.3.0+
Package: blast 2.3.0, build Nov 30 2015 13:32:08
```

The above was done via qlogin, but of course you would call things in the same manner using qsub when submitting a job. You'll notice that a specific binding is required currently on speed ```-B /nfs/speed-scratch/{$uid}:/speed-scratch/{$uid}``` because it gets all weird on you if you skip that. If you have enough space in your home directory, that's not needed, but I need ```/speed-scratch``` to do anything these days.


