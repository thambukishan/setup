If you want to use anexVis, a web service is provided at https://anexvis.chpc.utah.edu

If you want to provide your own host, contribute to the code, or adopt it for your research, it is recommended to set it up on a server with Ubuntu 16.04 or higher.

This repo provides an installation script example that can be used to set up your own instance of anexVis web application. 
The application involves several components that need to be stitched together. In principle, the server components can be implemented separately.
The example below only shows how to implement them in one single machine.

# Preparation

## OS
The application makes use of OpenCPU server runs on **Ubuntu 18.04 LTS**. Please make sure you have the appropriate OS before moving forward.

## Data file

NOT WORK YET

## Give a path on your local machine to download and install the packages
```bash
program_dir=$HOME"/Programs"
mkdir -p $program_dir
```
# Installation

## [Optional] Ubuntu upgrade

```bash
sudo apt-get update
sudo apt-get dist-upgrade -y
sudo apt-get install -y software-properties-common
sudo add-apt-repository -y ppa:opencpu/opencpu-2.0
sudo apt-get update
```

OpenCPU-2.0 is used for compatibility with Ubuntu 18.04 LTS. OpenCPU-1.6 is not compatible with this Operating System.

## Install an up-to-date R version
You may want to revise the mirror link and/or key depending on your location and your Ubuntu version. Modify arch to either i386 if OS is 32-bit or amd64 if OS is 64-bit.
For Operating System of Ubuntu 18.04 LTS, use bionic repository for compatibility. Since there is no repository for R version 3.4, make sure to use the bionic cran-35 for R version 3.5.

```bash
sudo add-apt-repository -y "deb [arch=amd64] https://cran.cnr.berkeley.edu/bin/linux/ubuntu bionic-cran35/"
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E084DAB9
sudo apt-get update
sudo apt-get install -y r-base
```
## OpenCPU

Install using package manager

```bash
sudo apt-get install opencpu
```

OpenCPU 2.0 has a different way to start. Refer to the documentation here [https://opencpu.github.io/server-manual/opencpu-server.pdf](https://opencpu.github.io/server-manual/opencpu-server.pdf)

```bash
......................
```

And make sure it works
```bash
curl http://localhost/ocpu/info
```

## Redis server

Download, extract, compile and install
```bash
cd $program_dir
wget http://download.redis.io/releases/redis-3.2.8.tar.gz
tar -xzf  redis-3.2.8.tar.gz
cd redis-3.2.8/
make
sudo make install
```

## R packages

### Developer tools

```bash
sudo apt-get install git
sudo Rscript -e "install.packages(c('futile.logger','testthat', 'roxygen2'), repos='https://cran.cnr.berkeley.edu/',Ncpus=2)"
sudo Rscript -e "devtools::install_version('devtools', version='1.11.1', repos='https://cran.cnr.berkeley.edu/',Ncpus=2,quiet=TRUE)"
```
### bioconductor dependencies

```bash
sudo Rscript -e 'source("https://bioconductor.org/biocLite.R");biocLite("rhdf5");'
```
### rvislib - a dependency of ranexvis
```bash
cd $program_dir
git clone https://github.com/anexVis/rvislib.git
cd rvislib
sudo Rscript -e "devtools::install('.')"
```
### rredis

At the time of this writing, rredis on CRAN has an unfixed bug which we don't want. The code below will install rredis from github.
```bash
sudo Rscript -e 'devtools::install_github("bwlewis/rredis"); devtools::install_dev_deps(".")'
redis-server &
disown
```


### ranexvis
As `ranexvis` depends on all the components above, make sure OpenCPU and redis-server are started, and your data file is in place.
```bash
cd $program_dir
git clone https://github.com/anexVis/ranexvis.git
```

Replace the custom data path and install the package

```bash
Rscript -e `printf 'load("data/sysdata.rda");dbpath=list(gtex="%s");save(list=ls(),file="data/sysdata.rda")' $DATAPATH`
sudo Rscript -e "devtools::install('.', quick=TRUE, force_deps=FALSE, upgrade_dependencies=FALSE)"
```

Add `ranexvis` to the list of packages to pre-load by OpenCPU server. You can also do this manually by modifying the line `preload` in `/etc/opencpu/server.conf`

```bash
sudo sed -i '/preload/ c\    "preload": ["ranexvis", "ggplot2"]' /etc/opencpu/server.conf
```
Restart OpenCPU
```bash
sudo service opencpu restart
```

From the time OpenCPU server is up, it generally takes about 30 seconds to finish loading the whole data set. Until then, the gene list and other inputs won't be available.

### Install anexvis-app

```bash
cd $program_dir
git clone https://github.com/anexVis/anexvis-app.git
sudo mv anexvis-app /var/www/html/
```
The app is accessible at http://url-to-your-apache-server/anexvis-app
