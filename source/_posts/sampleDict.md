---
title: sampleDict
date: 2018-11-06 23:02:17
tags:
---

# use python to write sampleDict

```python
# file writeSampleDict_2Diagonals.py

import numpy as np
import os

def writeStartEnd(whichLineGroup,f,z,nPoints):
    if whichLineGroup == 0:
        f.write("\t\t"+ "type\t"  + "uniform;" +"\r\n")
        f.write("\t\t"+ "axis\t"  + "distance;" +"\r\n")
        f.write("\t\t"+ "start\t" + "("+z+ " 0.004 0.004);" +"\r\n")
        f.write("\t\t"+ "end\t" + "("+z+ " -0.004 -0.004);" +"\r\n")
        f.write("\t\t"+ "nPoints\t" + str(nPoints) + ";" +"\r\n")
    elif whichLineGroup == 1:
        f.write("\t\t"+ "type\t"  + "uniform;" +"\r\n")
        f.write("\t\t"+ "axis\t"  + "distance;" +"\r\n")
        f.write("\t\t"+ "start\t" + "("+z+ " -0.004 0.004);" +"\r\n")
        f.write("\t\t"+ "end\t" + "("+z+ " 0.004 -0.004);" +"\r\n")
        f.write("\t\t"+ "nPoints\t" + str(nPoints) + ";" +"\r\n")
    else:
        print "big problem"

def writeLines(f,z_value_List,N,Nb_lineGroup,resolution,dictName):
    for i in range(Nb_lineGroup*N):
        whichLineGroup = i // N  # quotient  0, 1 ,2... Nb_lineGroup
        j = i % N                # remainder 0, 1, 2... Ns
        
        f.write("\t"+dictName+'-'+str(i)+"\r\n")
        f.write("\t{\r\n")
        writeStartEnd(whichLineGroup,f,z_value_List[j],resolution)
        f.write("\t}\r\n")
        f.write("\r\n")        
    
def write_LineSet(f,z_value_List,Nb_slices_along_z,Nb_lineGroup,resolution,dictName):
    f.write("sets\r\n")
    f.write("(\r\n")
    writeLines(f,z_value_List,Nb_slices_along_z,Nb_lineGroup,resolution,dictName)
    f.write(");\r\n")

def write_headerSetting_sampleLines(f,fieldName,interpolationScheme):
    f.write("setFormat raw;\r\n")
    f.write("\r\n")
    f.write("interpolationScheme "+interpolationScheme+";\r\n")
    f.write("\r\n")
    f.write("fields\r\n")
    f.write("(\r\n")
    f.write("\t"+fieldName+"\r\n")
    f.write(");\r\n")

def write_sampleLines(dictName,fieldName,interpolationScheme,z_value_List,Nb_lineDisplacement,Nb_lineGroup,resolution):
    f= open(dictName,"w")
    write_headerSetting_sampleLines(f,fieldName,interpolationScheme)
    write_LineSet(f,z_value_List,Nb_lineDisplacement,Nb_lineGroup,resolution,dictName)
    f.close()
    
def NLines2Dict(Nb_lineDisplacement):
    l = np.linspace(-0.08,-0.04,num=Nb_lineDisplacement)
    listStr = [str(x) for x in l]
    print "A list of all line displacements :\n"
    print listStr
    
    dictName="lineOn2Diagonals"
    fieldName="U_mean"
    interpolationScheme="cell"
    write_sampleLines(dictName,fieldName,interpolationScheme,listStr,Nb_lineDisplacement,2,resolution=200)
    
    os.system('dos2unix '+dictName)
    print "\n"
    print "sampleDict written. Think to add a file header!"

NLines2Dict(64)

```

1. configure python script for sampleDict : dictName, fieldName, interpolationScheme... Sampled data will be named as `postProcessing/sets/time/dictName-n_fieldName. "dictName" may also includes information like "interpolationScheme".

2. run python script

3. add header to sampleDict

4. run sample utility

5. verify generated files in `postProcessing`

6. begin plotting using python

7. edit plot parameters : a spatial statistic will require "time", "number of samples" etc. This file is the footprint of the statistics.

8. configure plot script : ajust "dataShape" or likewise. In other word, not all data are eligable because not all data are of the same shape. We can interpolate but I d rather not.

9. run plot script

10. ajust plot and save fig to `figure`

11. write plot data (x,y) to txt file in foler `data` for reporducing use or later ajustement


