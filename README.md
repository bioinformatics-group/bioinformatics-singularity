# bioinformatics-singularity
The software we use to run things in our lab. Some of the software is older than what is available to be consistent with other publications.

To run it with out pre-built image, you just call:

```singularity shell https://tootsuite.encs.concordia.ca/singularity-images/bioinformatics-singularity.simg```

Binaries are made available in ```/usr/bin``` so you can just run things like ```R``` or ```t_coffee```

Among other things, we use:
 * R 3.5
 * BLAST 2.3.0+
 * tcoffee (current version from git)
