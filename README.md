# cactus-guide
A guide of setting up and operating cactus on cluster 


Installation on Zeus HPC Cluster:
```
cd $MYGROUP

git clone https://github.com/nemequ/lzo.git
cd lzo
 ./configure --enable-shared --prefix=$PWD
make -j10
make install -- this will give an error, but you can ignore

cd $MYGROUP
wget https://github.com/alticelabs/kyoto/archive/stable-20170410.tar.gz
tar -xvf stable-20170410.tar.gz
cd kyoto-stable-20170410/
export LIBRARY_PATH=$MYGROUP/lzo/lib
export CPLUS_INCLUDE_PATH=$MYGROUP/lzo/include:$MYGROUP/kyoto-stable-20170410/kyotocabinet

cd kyotocabinet 
./configure --prefix="$MYGROUP/kyoto-stable-20170410" --enable-lzo 
make -j10
make install

cd ../kyototycoon/
PKG_CONFIG_PATH="../kyotocabinet" CPPFLAGS="-I../kyotocabinet" LDFLAGS="-L../kyotocabinet" ./configure --prefix="$MYGROUP/kyoto-stable-20170410" --with-kc="$MYGROUP/kyoto-stable-20170410"
make -j10
make install

Install Toil required to compile cactus
cp /group/pawsey0263/ddeeptimahanti/toil.cyg $HOME/.maali/sles12sp3/cygnet_files #requested to be copied to my group
module load maali
maali -t toil -v git -d
module use /group/pawsey0263/ashling_charles/software/sles12sp3/modulefiles
module load toil

cd $MYGROUP
git clone https://github.com/ComparativeGenomicsToolkit/cactus.git
cd cactus/
export ttPrefix=$MYGROUP/kyoto-stable-20170410
export kyotoTycoonIncl="-I${ttPrefix}/include -DHAVE_KYOTO_TYCOON=1"
export kyotoTycoonLib="-L${ttPrefix}/lib -Wl,-rpath,${ttPrefix}/lib -lkyototycoon -lkyotocabinet -lz -lbz2 -lpthread -lm -lstdc++"
git submodule update --init
make

module load python cython
mkdir -p $PWD//lib/python${MAALI_PYTHON_LIB_VERSION}/site-packages
export PYTHONPATH=$PWD/lib/python${MAALI_PYTHON_LIB_VERSION}/site-packages:$PYTHONPATH

python setup.py build
module load networkx
python setup.py install --prefix=$PWD


export PATH=$MYGROUP/cactus/bin:$PATH
```


SBATCH script successful for evolverMammals.txt example: 
```
#!/bin/bash -l
#SBATCH --job-name="myjob"
#SBATCH --nodes=1
#SBATCH -M zeus
#SBATCH --account=pawsey0149
#SBATCH --export=NONE
#SBATCH --ntasks=16
#SBATCH --ntasks-per-node=16
#SBATCH --output=cactusexample.%j.o
#SBATCH --error=cactusexample.%j.e

export PATH=/group/pawsey0149/ashling_charles/kyoto-stable-20170410/bin:$PATH
export PATH=$MYGROUP/cactus/bin:$PATH
export LD_LIBRARY_PATH=/group/pawsey0149/ashling_charles/kyoto-stable-20170410/lib:$LD_LIBRARY_PATH


module use /group/pawsey0149/ashling_charles/software/sles12sp3/modulefiles
module load python toil
export PYTHONPATH=/group/pawsey0149/ashling_charles/cactus/lib/python2.7/site-packages:$PYTHONPATH

cactus --maxCores 16 --binariesMode local jobStore $MYGROUP/cactus/examples/evolverMammals.txt $MYGROUP/cactus/examples/evolverMammals.hal

```
Time taken: 2386.51343918 seconds ie. 40 minutes 
Output tree: ((simHuman_chr6:0.144018,(simMouse_chr6:0.084509,simRat_chr6:0.091589)mr:0.271974)Anc1:0.020593,(simCow_chr6:0.18908,simDog_chr6:0.16303)Anc2:0.032898)Anc0;

Setting up HAL: 
```
module load hdf5
cd $MYGROUP
git clone https://github.com/ucscGenomeBrowser/kent.git
cd kent (compile the packages as required)

cd $MYGROUP
git clone https://github.com/ComparativeGenomicsToolkit/hal.git

export  ENABLE_UDC=1
export KENTSRC=$MYGROUP/kent/src (i've cloned kent in my group directory)
git clone https://github.com/benedictpaten/sonLib.git
pushd sonLib && make && popd

cd hal
make
```

Using Hal commands on .hal output of cactus 
eg. outputting newick tree
```
halStats --tree /group/pawsey0149/ashling_charles/cactus/examples/evolverMammals.hal
```
note if make does not work in installation - can execute commands in bin directory eg. ./halStats <halfile> 
 
 
 
 Installing RepeatMasker:
```

wget http://www.repeatmasker.org/rmblast-2.9.0+-p2-x64-macosx.tar.gz
tar zxvf rmblast-2.9.0-p2-x64-linux.tar.gz

module load trf
mkdir -p $PWD//lib/trf${MAALI_PYTHON_LIB_VERSION}/site-packages
export TRFPATH=$PWD/lib/trf${MAALI_PYTHON_LIB_VERSION}/site-packages:$PATH

wget http://www.repeatmasker.org/RepeatMasker-4.1.0.tar.gz
gunzip RepeatMasker-4.1.0.tar.gz
tar xvf RepeatMasker-4.1.0.tar

cd RepeatMasker
perl ./configure
```
 
 
 
