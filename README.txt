
#SPDK PMDK & Intel(r) VTune(tm) Amplifier Developer Summit
# April 17, 2019
# 1:00pm - 3:00pm
# Hayes Mansion, San Jose
#
# These are the materials used during the hackathon:
#	slides_2019_04_17.pdf contains the slides shown
#	Commmands demonstrated during the hackathon are listed below.
#	Source files for programming examnples are in this repo.
#
# These instructions are designed to work on in your lab session
# running Ubuntu 18.04.1 (bionic)
# libpmemobj-cpp workshop
#
# These are the materials used during the workshop:
#	slides.pdf contains the slides shown
#	Commands demonstrated during the workshop are listed below.
#	Source files for programming examples are in this repo.
#
# These instructions are designed to work on the workshop guest VMs.
# Start by cloning this repo, so you can easily cut and paste commands from
# this README into your shell.
#
# Many of the sys admin steps described here can be found in the
# Getting Started Guide on pmem.io:
# https://docs.pmem.io/getting-started-guide

# HW configuration: 
# We have already created regions and namespaces and and mounted DAX-capable file system. 
# Go to /mnt/pmem-fsdax0/pmdkuser<x>

# Software 
# All the following required libraries are already installed 
# libmemkind, PMDK and libpmemobj-cpp
#
# Start by making a clone of this repo...
#
git clone https://github.com/pmemhackathon/2019-04-17
cd 2019-04-17

#
# Compile examples.
#
make

#
# Warmup - a simple persistent counter.
#
# pmempool create obj may fail since the SDS feature see:
# https://github.com/pmem/issues/issues/1039
# when you create the pool by the pmempool create, you may need use the command:
# PMEMOBJ_CONF="sds.at_create=0" pmempool create obj --layout=queue -s 100M /mnt/pmem-fsdax/queue
#
pmempool create obj --layout=warmup -s 100M /mnt/pmem-fsdax0/${USER}/warmup
./warmup /mnt/pmem-fsdax0/${USER}/warmup

#
# find_bugs.cpp
#
# Program which contains few bugs. Can you find them?
#
pmempool create obj --layout=find_bugs -s 100M /mnt/pmem-fsdax0/${USER}/find_bugs
./find_bugs /mnt/pmem-fsdax0/${USER}/find_bugs

# run find-bugs under pmemcheck
valgrind --tool=pmemcheck ./find_bugs /mnt/pmem-fsdax0/${USER}/find_bugs

#
# queue.cpp
#
# Simple implementation of a volatile queue.
#
./queue
push 1
push 2
push 3
pop
show

#
# queue_pmemobj.cpp
#
# Simple implementation of a persistent queue.
#
pmempool create obj --layout=queue -s 100M /mnt/pmem-fsdax0/${USER}/queue
pmempool info /mnt/pmem-fsdax0/${USER}/queue
./queue_pmemobj /mnt/pmem-fsdax0/${USER}/queue
push 1
push 2
push 3
pop
show

#
# simplekv_simple.cpp
#
# Hashmap test program.
#
pmempool create obj --layout=simplekv -s 100M /mnt/pmem-fsdax0/${USER}/simplekv-simple
pmempool info /mnt/pmem-fsdax0/${USER}/simplekv-simple
./simplekv_simple /mnt/pmem-fsdax0/${USER}/simplekv-simple

#
# simplekv_word_count.cpp
#
# A C++ program which reads words to a simplekv hashtable and uses MapReduce
# to count words in specified text files.
#
pmempool create obj --layout=simplekv -s 100M /mnt/pmem-fsdax0/${USER}/simplekv-words
pmempool info /mnt/pmem-fsdax0/${USER}/simplekv-words
./simplekv_word_count /mnt/pmem-fsdax0/${USER}/simplekv-words words1.txt words2.txt
