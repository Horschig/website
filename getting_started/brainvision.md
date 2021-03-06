---
title: Getting started with BrainVision Analyzer and Easycap
tags: [dataformat, brainvision, eeg, layout]
---

# Getting started with BrainVision Analyzer and Easycap

### Introduction

[BrainProducts](http://www.brainproducts.com) provides technical and software solutions for neurophysiological and psychophysiological research and clinical applications. Their BrainAmp ExG amplifier allows to record signals with a sampling rate up to 5000 Hz and a broad hardware bandwidth ranging from DC to 1000 Hz. Brain Products also provides EEG caps with the electrodes distributed over the head according to some standard schemes. Most of the caps provided by BrainProducts are actually fabricated by [EasyCap](http://www.easycap.de), on whose website you can also find more information. Although it is possible to use a BrainAmp amplifier with another type of cap, or to use an Easycap with an different amplifier, the most common case is to use them together and that is why we describe them jointly on this page.


Further information on the BrainVision file format
The BrainVision file format consists of three separate files:

1. A text header file (`.vhdr`) containing meta data
2. A text marker file (`.vmrk`) containing information about events in the data
3. A binary data file (`.eeg`) containing the voltage values of the EEG

Both text files are based on the Microsoft Windows INI format consisting of:

- sections marked as `[square brackets]`
- comments marked as `; comment`
- key-value pairs marked as `key=value`

{% include markup/info %}
The BrainVision Recorded and Analyzer software packages use a well-defined file format that is supported by many software packages (e.g. FieldTrip, EEGLAB, SPM, MNE-Python). The details of the file format are available in [this specification](/assets/pdf/BrainVisionCoreFileFormat_1.0_2018-08-02.pdf).
{% include markup/end %}

For example, see this excerpt from a BrainVision header file (.vhdr):

    Brain Vision Data Exchange Header File Version 1.0
    ; Data synthesized by MNE-BIDS

    [Common Infos]
    DataFile=test.eeg
    MarkerFile=test.vmrk

In this short example we can observe a challenge that is caused by having three separate files for each dataset: It means that the single files have internal pointers to each other's locations (see the DataFile and MarkerFile keys in the example).

{% include markup/info %}
Manually renaming BrainVision datasets may lead to errors, since the .vhdr and .vmrk file headers contain the name of the linked data file. Paul Czienskowski from the MPI for Human Development in Berlin, Germany, has written a small windows program that you can use: http://code.google.com/p/eeg-renamer/. Or you can use [this MATLAB function](https://gist.github.com/CPernet/e037df46e064ca83a49fb4c595d4566a). When renaming a single or small number of datasets, you could also use a text editor to fix the header.
{% include markup/end %}

{% include markup/success %}
For validation of BrainVision file triplets, you can use the [brainvision-validator](https://github.com/sappelhoff/brainvision-validator), which is a command line tool developed in nodejs.
{% include markup/end %}

### Preprocessing of raw EEG data

FieldTrip needs the user to define what file to read in. The BrainVision Recorder software usually stores different filetypes (.vhdr, .eeg, .vmrk). For reading the data into FieldTrip you can refer to the .eeg file, for exampl

    cfg = [];
    cfg.dataset = '/users/karlheinz/EEG/myrecording.eeg';
    ...

The .eeg files are the raw data files, i.e. they contain the data as it has been stored upon acquisition.

You can subsequently epoch your data using [ft_definetrial](/reference/ft_definetrial), and you can read in the data and preprocess it using [ft_preprocessing](/reference/ft_preprocessing). Note that in FieldTrip, no unit conversion takes place.

### Exporting raw EEG data after doing processing in BVA

Sometimes users have already done some processing (e.g., rereferencing, epoching, artifact identification) in BrainVision Analyzer, and in order to avoid repeating the time consuming / subjective selection steps it might be preferable to start from the processed data. BrainVision Analyzer stores the processing steps in a so called history file, keeping the raw data unchanged, and applying the processing steps on-the-fly. This is not something that FieldTrip can work with, so you need to export your data first.

The following describes the recipe to export the processed data into a format the FieldTrip can deal with. It produces a triplet of files (.vhdr, .vmrk and .seg (instead of .eeg)), that can be imported into FieldTrip, in much the same way as described above.
You can do all the preprocessing you want to do in BrainVision Analyzer (e.g. iltering and re-referencing can be done too) and once you have the data segmented the way you want it select 'export > generic data'.  You'll get a window (maybe 2 consecutive windows) popping up asking for various settings.  Leave everything as it is, except make sure the following are set:

 1.  For the filename output you should set it to be .seg instead of .eeg (which I think is the default).
 2.  DataFormat should be 'BINARY'
 3.  DataOrientation should be 'MULTIPLEXED'
 4.  Encoding should be 'UTF-8'
 5.  DataType should be 'TIMEDOMAIN'
 6.  BinaryFormat should be 'IEEE_FLOAT_32'

 {% include markup/info %}
 When comparing your preprocessed data from FieldTrip to preprocessed data from BrainVision Analyzer, you might notice subtle differences. This might be due to two reasons: First, the filtersettings of BVA are hard to mimic using FieldTrip, because FieldTrip is using different defaults. Also, the order of preprocessing steps is fixed in FieldTrip, whereas you have to perform them manually, which makes it possible to do them in any order in BVA. The effect filters have on your data depend on the order of the preprocessing steps.
 {% include markup/end %}

### Plotting

FieldTrip provides electrode layouts for the purpose of plotting data recorded by means of an easycap. These layouts are stored in .mat files and are based on the manufacturer's original drawings, which can also be found as .gif files in the /template/layout directory.

Using FieldTrip, data recorded with Brain Vision hard- and software is readily plotted. It is important that the channel labels match that of the manufacturer specifications. Specify the layout that matches your set-up/easycap when plotting, e.g.

   cfg.layout = 'easycapM1.lay';

Examples regarding the type of plots can be observed [here](/tutorial/plotting). In the [template](/template/layout) directory you can find a collection of template layouts for plotting. If you want to create your own custom layout files, please have a look [here](/tutorial/layout).
