---
title: my-2nd-blog
date: 2018-05-28 21:12:42
categories: diary
tags: [Hexo,diary]
---

# Analysis on sample utility

```cpp
// IOOutputFilter.H
template<class OutputFilter>
class IOOutputFilter
:
    public IOdictionary,
    public OutputFilter


// IOOutputFilter.C 
template<class OutputFilter>
Foam::IOOutputFilter<OutputFilter>::IOOutputFilter
(
    const word& outputFilterName,
    const objectRegistry& obr,
    const fileName& dictName,
    const IOobject::readOption rOpt,
    const bool readFromFiles
)
:
    IOdictionary
    (
        IOobject
        (
            dictName,
            obr,
            rOpt,
            IOobject::NO_WRITE
        )
    ),
    OutputFilter(outputFilterName, obr, *this, readFromFiles)
{
	Info << "IOOutputFilter 2rd constructor : " << outputFilterName << endl;
}

```
