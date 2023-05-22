# Workflow for single pulse pulsar data analysis

This flowchart is written for single pulse processing using the uGMRT 8-bit (or lower) pulsar data. 

The uGMRT pulsar data (PA: phased array) files are usually in *.raw* or *.gmrt_raw* format which basically has recorded voltages across frequency and time. With this the user also gets a *.log* and *.gmrt_hdr* (or *.raw.hdr*) file. The **.hdr* file contains the observation timestamp which is necessary for the primary data reduction.

## Primary Data Reduction
* **Step 1**: 


The DSPSR package does not work on 16-bit data