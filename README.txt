# SPDK PMDK & Intel(r) VTune(tm) Amplifier Developer Summit
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
#
# Many of the sys admin steps described here can be found in the
# Getting Started Guide on pmem.io:
https://docs.pmem.io/getting-started-guide


#
# making sure your platform provides an NFIT table
ndctl list -BN     # check the "provider" field

#
# Checking to make sure your kernel supports pmem:
#
uname -r	# see kernel currently running
grep -i pmem /boot/config-`uname -r`
grep -i nvdimm /boot/config-`uname -r`

#
# Use ipmctl to show that you have pmem installed
#

ipmctl show -topology
ipmctl show -memoryresources
ipmctl show -dimm

#
# Use ndctl to show that you have pmem installed:
#
ndctl list -RuN

# HW configuration 
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
# raw.c contains the simplest possible memory-mapped pmem example
# this illustrates how to use mmap().  nothing transactional here!
#
dd if=/dev/zero of=/mnt/pmem-fsdax/daxfile bs=4k count=1
od -c /mnt/pmem-fsdax/daxfile
./raw /mnt/pmem-fsdax/daxfile
od -c /mnt/pmem-fsdax/daxfile

#
# volatile_pmem.c demonstrates using libmemkind to allocate both
# normal memory and pmem for volatile use.  nothing persistent here!
#
./volatile_pmem /mnt/pmem-fsdax 100

#
# freq.c
#
# Simple C program for counting word frequency on a list on text files.
# It uses a hash table with linked lists in each bucket.
#
# This just sets the stage with a simple programming example.  Nothing
# related to pmem yet.
#
./freq -p words.txt

#
# freq_mt.c
#
# Convert freq.c to be multi-threaded.  A separate thread is created
# for each text file.
#
./freq_mt -p words.txt words.txt words.txt

#
# freq_pmem.c
#
# Convert freq_mt.c to mode the hash table to persistent memory and use
# libpmemobj for pmem allocation, locking, and transactions.
#
# We also use freq_pmem_print.c to display the currrent hash table contents.
#
pmempool create obj --layout=freq -s 100M /mnt/pmem-fsdax/freqcount
pmempool info /mnt/pmem-fsdax/freqcount
./freq_pmem_print /mnt/pmem-fsdax/freqcount
./freq_pmem /mnt/pmem-fsdax/freqcount words.txt words.txt words.txt
./freq_pmem_print /mnt/pmem-fsdax/freqcount

#
# freq_pmem_cpp.c
#
# Similar to freq_pmem.c but uses the C++ bindings to libpmemobj.
#
./freq_pmem_cpp /mnt/pmem-fsdax/freqcount words.txt words.txt words.txt

#
# For more examples when you cloned the PMDK tree:
#
cd
cd pmdk/src/examples  
  

