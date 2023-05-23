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

    `filterbank your_file_name.dat > your_file_name.fil`

    This will create a *.fil* filterbank file of the data. This file will have the header that was made in the last step and the data will be organised as a dynamic spectrum data cube (intensity vs frequency vs time). This file will be used for all the further processing and can be accessed using most pulsar analysis softwares.

## Get single pulses

* **Step 1**: The filterbank file obtained in the previous step can be used to get individual pulses. Obviously this requires knowing the pulsar ephemiris. For known pulsar observations, the ephemiris is obtained from [psrcat](https://www.atnf.csiro.au/research/pulsar/psrcat/), and saved as `your_file_name.eph`.

    For blind pulsar searches the process is very different and not disussed here.

* **Step 2**: To get the single pulses `dspsr` routine of the DSPSR package is used. 

    `dspsr -E your_file_name.eph -U 600 -b 512 -s -K -A -O your_file_name your_file_name.fil`

    where
    * -E is for the ephemiris.
    * -U is the upper limit on RAM usage. If 600 is not the right value for your system it will automatically suggest the proper value once the command runs.
    * -b stands for the number of bins. This can be changed according to preference and can be reduced later on if needed. Keep it in powers of 2.
    * -K is for removing the inter-channel dispersion delays.
    * -s is to prompt `dspsr` to create single pulse sub-integrations.
    * -A asks `dspsr` to output a single archive file with all the integrations. This option still retains the single pulses, and only puts them in a order such that each pulse can be accessed later on. Without this the -s option will create single pulse archives, which can be added later on if needed.
    * -O is the output file name for the full archive file, with a `.ar` extension.

The output file `your_file_name.ar` will have the following information about every pulse - 
* MJD / pulse number
* Intensity (uncalibrated) vs phase/bins vs frequency

The DSPSR package does not yet work on 16-bit data.

* **Step 3:** For new pulsars where the period is not exactly known, the `pdmp` routine can be used.

    `pdmp -g your_file_name.png/png your_file_name.ar`

    Since the size of a typical `.ar` file is too large, a common practice is to scrunch all or some of the frequency channels, otherwise the routine could crash.

    `pam -f 16 -e arf16 your_file_name.ar`

    which makes a new file with a frequency scrunch by a factor of 16 `your_file_name.arf16`. Basically this file will have 4096/16 frequency channels. Now this file can be used in `pdmp`.

    `pdmp -g your_file_name.arf16.png/png your_file_name.arf16`

## Data Cleaning

Usually pulsar data comes with radio frequency interference (RFI), which should be removed before any further scientific analysis.
