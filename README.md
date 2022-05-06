# bioinformatics-singularity
The software we use to run things in our lab. Some of the software is older than what is available to be consistent with other publications.


# Building the Image
Make sure `/tmp` has enough space! Why wouldn't it? I'm not sure, but one of the nice build environments I had only allocated 350M to `/tmp` and that broke things in confusing and unexpected ways. I'm used to singularity just saying no when there's a trouble, but since it happned while R was doing installs, it snuck by. 2GB should be plenty of room, but I've had to be cautious, including making sure I had enough RAM (more weirdness with R, 16GB is *safe*). I am now building with Singularity 3.x.

The current simple image build I'm using is on a vm. I install Centos 9 (Stream), then run:
````sh
yum -y install epel-release
yum -y install singularity git wget debootstrap
yum -y install nano
````

`singularity build --sandbox <PathToBasedir>/image Singularity`
This builds the image as a directory structure that you can go into. You can work in this in writable mode if you need to tweak (or even from outside singularity). 

`singularity build <PathToBasedir>/bioinformatics-singularity.sif <PathToBasedir>/image`
This builds the image as a squashfs formatted image, suitable for putting on environments where people will/run use it in a fixed form.

# Running the Image
To run it with our pre-built image, you just call:

```singularity shell https://tootsuite.encs.concordia.ca/singularity-images/bioinformatics-singularity.sif```

I reccomend running it with an overlay as some of our tools have the bad habit of trying to write into their own temporary space:
````sh
mkdir /<wherever>/overlay
singularity shell --overlay /<wherever>/overlay https://tootsuite.encs.concordia.ca/singularity-images/bioinformatics-singularity.sif
````

Eventually, intermediate_files used by our tools won't need such a workaround.

Binaries are made available in ```/usr/bin``` so you can just run things like ```R``` or ```t_coffee```

Among other things, we use:
 * R 3.5
 * BLAST 2.6.0+
 * tcoffee (current version from the science packages in Debian Buster)
 * eggnog-mapper (current version from git)
 
## eggnog-mapper
We have not included the contents of the `data` dir for `eggnog-mapper` and thus any commands using it should use `--data_dir` to specify where those very large files are. IF you need those files, you can use the download script provided by eggnog-mapper `/usr/bin/eggnog-mapper/download_eggnog_data.py`, but you'll still need to pass the directory where those files will end up via `--data_dir`. You'll want to set this up outside the singularity image. 

While eggnog-mapper can be found in `/usr/bin/eggnog-mapper'`, we have included a script in the image `emapper.sh` that can be run which is already in the standard path and which will pass arguments as appropriate.

### eggnog-mapper
Are you running this image at Concordia on speed? We suggest the following example call, which executes the test from their installation guide but maps to the locally stored version of that database:

````
singularity exec -B `pwd`:$PWD -B /speed-scratch/bioinformatics-group/datasets/eggnog-mapper:datasets /speed-scratch/bioinformatics-group/bioinformatics-singularity.sif emapper.sh --data_dir /datasets -i /usr/bin/eggnog-mapper/test/p53.fa --output p53_maNOG -m diamond
````

I suggest making the data files (approximately 40G) available to your images on as fast a disk as possible (I've seen putting it in `/dev/shm` suggested).
=======

## Running our Tools
While a variety of tools are available in this image, we have included a number using the [SCI-F](https://sci-f.github.io/) approach advocated with Singularity. Namely, one can view our apps in the singularity image via:
```
singularity inspect --list-apps <yourimage>
TooT-P
TooT-SC
TooT-T
TranCEP
```

Help is available for each image, e.g.:
```
singularity run-help --app TooT-P <youimage>
    Usage: TooT-P.py [-h] -query QUERY [-work WORK] [-out OUT] [-db DB]
                 [-TooTT TOOTT] [-TooTSC TOOTSC]
```

Scripts can be run via standard execution as described in the help, or via the app interface, e.g.:
```
singularity run --app TooT-P <yourimage> -query=<yourfasta>
Executing: /usr/local/bin/TooT-T.R -query=<$CWD>/work/TooT-P/P37327/P37327.fasta -db=<$CWD>/db -out=<$CWD>/work/TooT-P/P37327 -work=<$CWD>
Executing: /usr/local/bin/TooT-SC.R -query=<$CWD>/work/TooT-P/P37327/P37327.fasta -db=<$CWD>/db -out=<$CWD>/work/TooT-P/P37327 -work=<$CWD>
```
...you know, if `<yourfasta>` just contains the one sequence `P37327`

 
## Running on Concordia's HPC infrastructure
If you're at Concordia and have requested access to speed (rt-ex-hpc), then you may want to be running jobs here. You can readily use this image, as we keep a local copy in ```/speed-scratch/bioinformatics-group/bioinformatics-singularity.sif```. In that case you can go to your working directory where you have your expected script and just run it. Keep in mind that speed likes you to use tcsh, but you're running bash from within the image.

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
singularity exec -B `pwd`:$PWD /speed-scratch/bioinformatics-group/bioinformatics-singularity.sif ./test.sh
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

The above was done via qlogin, but of course you would call things in the same manner using qsub when submitting a job. You'll notice that a specific binding is required when using speed-scratch (or any of the nfs-mounted directories, I suspect) as your working directory: ```-B `pwd`:$PWD```. It gets all weird on you if you skip that. If you have enough space in your home directory, that's not needed, but I need ```/speed-scratch``` to do anything these days.

