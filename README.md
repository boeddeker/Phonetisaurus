## README.md ##
=========

Phonetisaurus G2P

This repository contains scripts suitable for training, evaluating and using grapheme-to-phoneme
models for speech recognition using the OpenFst framework.  The current build requires OpenFst
version 1.6.0 or later, and the examples below use version 1.6.2.

The repository includes C++ binaries suitable for training, compiling, and evaluating G2P models.
It also some simple python bindings which may be used to extract individual
multigram scores, alignments, and to dump the raw lattices in .fst format for each word.

Standalone distributions related to previous INTERSPEECH papers, as well as the complete, exported
final version of the old google-code repository are available via ```git-lfs``` in a separate
repository:
  * https://github.com/AdolfVonKleist/phonetisaurus-downloads

#### Contact: ####
  * phonetisaurus@gmail.com

#### Scratch Build for OpenFst v1.6.2 and Ubuntu 14.04/16.04 ####
This build was tested via AWS EC2 with a fresh Ubuntu 14.04 and 16.04 base, and m4.large instance.

```
$ sudo apt-get update
# Basics
$ sudo apt-get install git g++ autoconf-archive make libtool
# Python bindings
$ sudo apt-get install python-setuptools python-dev
# mitlm (to build a quick play model)
$ sudo apt-get install gfortran
```
Next grab and install OpenFst-1.6.2 (10m-15m):
```
$ wget http://www.openfst.org/twiki/pub/FST/FstDownload/openfst-1.6.2.tar.gz
$ tar -xvzf openfst-1.6.1.tar.gz
$ cd openfst-1.6.1
# Minimal configure, compatible with current defaults for Kaldi
$ ./configure --enable-static --enable-shared --enable-far --enable-ngram-fsts
$ make -j 4
# Now wait a while...
$ sudo make install
$ cd
# Extend your LD_LIBRARY_PATH .bashrc:
$ echo 'export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/usr/local/lib:/usr/local/lib/fst' \
     >> ~/.bashrc
$ source ~/.bashrc
```

Checkout the latest Phonetisaurus from master
```
$ git clone https://github.com/AdolfVonKleist/Phonetisaurus.git
$ cd Phonetisaurus/src
$ ./configure
$ make
$ sudo make install
```

Compile the python bindings if you want to:
```
$ make phonetisaurus-binding
$ sudo make install
$ cd ..
$ sudo python setup.py install
$ cd
```

Grab and install mitlm to build a quick test model with the cmudict (5m):
```
$ git clone https://github.com/mitlm/mitlm.git
$ cd mitlm/
$ ./autogen.sh
$ make
$ sudo make install
$ cd
```

Train a quick toy model with the cmudict:
```
$ mkdir example
$ cd example
$ wget https://raw.githubusercontent.com/cmusphinx/cmudict/master/cmudict.dict
# Clean it up a bit and reformat:
$ cat cmudict.dict \
  | perl -pe 's/\([0-9]+\)//; 
              s/\s+/ /g; s/^\s+//; 
              s/\s+$//; @_ = split (/\s+/); 
              $w = shift (@_); 
              $_ = $w."\t".join (" ", @_)."\n";' \
  > cmudict.formatted.dict
# Align the dictionary (5m-10m)
$ phonetisaurus-align --input=cmudict.formatted.dict \
  --ofile=cmudict.formatted.corpus --seq1_del=false
# Train an n-gram model (5s-10s):
$ estimate-ngram -o 8 -t cmudict.formatted.corpus \
  -wl cmudict.o8.arpa
# Convert to OpenFst format (10s-20s):
$ phonetisaurus-arpa2wfst --lm=cmudict.o8.arpa --ofile=cmudict.o8.fst
$ cd
```

Test the model with the wrapper script:
```
$ cd Phonetisaurus/script
$ ./phoneticize.py -m ~/example/cmudict.o8.fst -w testing
  11.24   T EH1 S T IH0 NG
  -------
  t:T:3.31
  e:EH1:2.26
  s:S:2.61
  t:T:0.21
  i:IH0:2.66
  n|g:NG:0.16
  <eps>:<eps>:0.01
```

Use a special location for OpenFst, parallel build with 2 cores
```
 $ ./configure --with-openfst-libs=/home/ubuntu/openfst-1.6.2/lib \
          --with-openfst-includes=/home/ubuntu/openfst-1.6.2/include
 $ make -j 2 all
```

Use custom g++ under OSX (Note: OpenFst must also be compiled with this
custom g++ alternative [untested with v1.6.2])
```
 $ ./configure --with-openfst-libs=/home/osx/openfst-1.6.2gcc/lib \
          --with-openfst-includes=/home/osx/openfst-1.6.2gcc/include \
	  CXX=g++-4.9
 $ make -j 2 all
```

#### Rebuild configure ####
If you need to rebuild the configure script you can do so:
```
 $ cd .autoconf
 $ autoconf -o ../configure
 $ cd ../
 $ make -j2 all
```

### Install [Linux]: ###
```
 $ sudo make install
```

### Uninstall [Linux]: ###
```
 $ sudo make uninstall
```

### Usage: ###
#### phonetisaurus-align ####
```
 $ bin/phonetisaurus-align --help
```
#### phonetisaurus-arpa2wfst ####
```
 $ bin/phonetisaurus-arpa2wfst --help
```
#### phonetisaurus-g2prnn ####
```
 $ bin/phonetisaurus-g2prnn --help
```
#### phonetisaurus-g2pfst ####
```
 $ bin/phonetisaurus-g2pfst --help
```
