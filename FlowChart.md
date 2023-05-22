# Workflow for single pulse pulsar data analysis

This flowchart is written for single pulse analysis using the uGMRT 8-bit (or lower) pulsar data. 

The uGMRT pulsar data (PA: phased array) files are usually in *.raw* or *.gmrt_raw* format which basically has recorded voltages across frequency and time. With this the user also gets a *.log* and *.gmrt_hdr* (or *.raw.hdr*) file. The **.hdr* file contains the observation timestamp which is necessary for the primary data reduction.

## Primary Data Reduction
* **Step 1**: Oftentimes the *.raw* or *.gmrt_raw* extension creates problems, so it's best to just create a symbolic link with a *.dat* extension.

    `ln -s your_file_name.raw your_file_name.dat`

* **Step 2**: Create a header file. This is different from *.gmrt_hdr* or *.raw.hdr* file. Write the following information - 

        **DATA FILE HEADER**
        Site            : GMRT 
        Observer        : GMRT
        Proposal        : 38_002
        Array Mode      : PA
        Observing Mode  : Search
        Date            : 09/06/2020
        Num Antennas    : 13
        Antenna List    : C00 C01 C03 C04 C05 C06 C08 C09 C10 C11 C12 C13 C14
        Num Channels    : 4096
        Channel width   : -0.048828
        Frequency Ch.1  : 500
        Sampling Time   : 327.68
        Num bits/sample : 8
        Data Format     : integer binary, little endian
        Polarizations   : Total I
        MJD             : 59009
        UTC             : 16:48:33.0.812421000
        Source          : J1820-0427
        Coordinates     : 18:20:52.5934, -04:27:37.712
        Coordinate Sys  : J2000
        Drift Rate      : 0.0, 0.0
        Obs. Length     : 9830400
        Bad Channels    : 3:1-650,2400-2800,4000-4096
        Bit shift value : 1

    Most of these values can stay same, but be careful to change the following - 
    * Num Channels: This should be the number of channels in the observation. Eg. 2k (2048), 4k (4096), 8k (8192), 16k (16,384) etc. It basically is the total number of frequency chunks in the observation bandwidth.
    * Channel Width: This is the width of each frequency channel in MHz. Multiplying this width with the number of channels will give the bandwidth of observations. The negative sign accounts for the negative bandwidth for Band 3 of uGMRT where the first channel is 500 MHz and last channel is 300 MHz. For Band-  data the first channel is at 550 MHz and the last at 750 MHz, so the sign of channel width will be positive in that case. 
    * Frequency Ch.1: This would be the first frequency channel in the data bandwidth. This would be 550 (MHz) for Band 4 data.
    * Sampling time: While making the observations, this must be set at some value (in mirco-seconds).
    * Num bits/sample: Some new uGMRT data is recorded with 16-bit as well. Change accordingly. 
    * MJD: This is the rough value, no need to be too precise since it doesn't change too much within a day, which is less than a typical observation length. Find the MJD value using the date and time given in *.gmrt_hdr* file through the `cal2mjd` command of PRESTO.\
    PS: In the *.gmrt_hdr* file the time and date are in IST (Indian Standard Time). For using `cal2mjd` convert from IST to UTC.
    * UTC: Convert the IST time from the *.gmrt_hdr* file to UTC.
    * Coordinates: Change according to the source. 
    * Bad Channels: Generally the known RFI channels are removed beforehand. If the full data is needed then keep all the channels.

    Save this file as `your_file_name.hdr`. 

* **Step 3**: Run `filterbank` command from PSRCHIVE.

    `filterbank `

The DSPSR package does not work on 16-bit data