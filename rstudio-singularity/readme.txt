RStudio server Singularity container for Open Ondemand.

Pulled from https://github.com/nickjer/singularity-rstudio.git

Update apr19

Use Jeremy's Singularity image

mkdir /uufs/chpc.utah.edu/sys/installdir/rstudio-singularity/1.1.463
cd /uufs/chpc.utah.edu/sys/installdir/rstudio-singularity/1.1.463
singularity pull shub://nickjer/singularity-rstudio:3.5.3

Update Nov19

Use Bob Settlage's Docker image - has bio R libs installed which was requested by user:

cd /uufs/chpc.utah.edu/sys/installdir/rstudio-singularity/3.6.1
singularity pull docker://rsettlag/ood-rstudio-bio:3.6.1

Some other useful info on Bob's setup:
his containers: https://hub.docker.com/u/rsettlag
( may try also rsettlag/ood-rstudio-basic and rsettlag/ood-rstudio-geospatial
his OOD apps: https://github.com/rsettlage/ondemand2

Update 2, Nov19
Bob has hardcoded http_proxies in the container R config files (/usr/local/lib/R/etc/Rprofile.site), rather than trying to hack this through user's ~/.Rprofile, pull sandbox container, modify and save:

$ singularity build --sandbox ood-rstudio-bio_3.6.1 docker://rsettlag/ood-rstudio-bio:3.6.1
sudo /uufs/chpc.utah.edu/sys/installdir/singularity3/std/bin/singularity shell --writable ood-rstudio-bio_3.6.1
apt-get update && apt-get install apt-file -y && apt-file update && apt-get install vim -y
vi /usr/local/lib/R/etc/Rprofile.site
- remove:
 Sys.setenv(http_proxy="http://uaserve.cc.vt.edu:8080")     
 Sys.setenv(https_proxy="http://uaserve.cc.vt.edu:8080")     
 Sys.setenv(R_ENVIRON_USER="~/.Renviron.OOD")     
 Sys.setenv(R_LIBS_USER="~/R/OOD/3.6.1") 

$ sudo singularity build ood-rstudio-bio_3.6.1.sif ood-rstudio-bio_3.6.1/

NOTE - trying to figure out how to get the http_proxy env. vars. into the RStudio Server session.
The process is as follows
OOD job (script.sh.erb) -> start rserver in container (env vars to here need to be brought in with SINGULARITYENV_) -> each new R session starts with $RSESSION_WRAPPER_FILE - which starts the R session.

Turns out that the http_* env. vars. propagate from the user host session into the container, but, rserver does not propagate them into the rsession.sh that starts the session - hand adding whatever other environment variables we need into the rsession.sh, defined in template/script.sh.erb, makes those variables to pass into the rsession inside of OOD. See https://github.com/CHPC-UofU/bc_osc_rstudio_server/blob/master/template/script.sh.erb
