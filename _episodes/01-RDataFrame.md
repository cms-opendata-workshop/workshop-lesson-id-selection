---
title: "Higgs to Tau Tau analysis and RDataFrame"
teaching: 20
exercises: 10
questions:
- "How is the Higgs -> Tau Tau analysis example set up?"
objectives:
- "Checkout Higgs -> Tau Tau code for the workshop"
- "Understand basic RDataFrame commands for filtering and defining variables"
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---

## 

CMS NanoAOD files store per-event information that is needed in analyses. 
NanoAOD files are stored in Ntuple format and contains a main TTree named Events and some additional TTrees for run, lumi and metadata. 

In the previous exercises, you learned how to access and store object information from an AOD file and convert the AOD file to NanoAOD using the [AOD2NanoAODOutreachTool](https://github.com/cms-opendata-analyses/AOD2NanoAODOutreachTool). 
[HiggsTauTauNanoAODOutreachAnalysis] (https://github.com/cms-opendata-analyses/HiggsTauTauNanoAODOutreachAnalysis) repository  contains an analysis that reduces the NanoAOD files to study the decay of a Higgs boson to two tau leptons.
This exercise uses this repository as the example we will use for selecting events in the NanoAOD based on requirements on muons and taus.  

Data samples to be used in the analysis:

*Run2012B_TauPlusX
*Run2012C_TauPlusX


Simulations to be used in the analysis:

*GluGluToHToTauTau
*VBF_HToTauTau
Background processes can produce very similar signatures in the detector which have to be considered in the anaylsis.
The most prominent background processes with a similar signature:
*DYJetsToLL
*TTbar
*W1JetsToLNu
*W2JetsToLNu
*W3JetsToLNu

We have to take all these background processes into account when analyzing our data and drawing conclusions.
In event selection, our goal is to keep the signal coming from Higss to tau tau decay and suppress the backgrounds.
These background processes can be suppressed by introducing kinematic limits on the physics objects.
For instance, the W boson can decay into a lepton. The leptons can be misidentified as coming from a signal. 
A cut in the event selection on the transverse mass of the muon and the missing energy can strongly suppress this background coming from W+ jets. 

#Viewing NanoAOD files

~~~
root -l root://eospublic.cern.ch//eos/opendata/cms/derived-data/AOD2NanoAODOutreachTool/GluGluToHToTauTau.root
new TBrowser;
~~~
{: .source}

[Insert picture of Tree here]

{: .output}

You can now view the Event branches by selecting the root file GluGluToHToTauTau.root and selecting the main TTree named Events.
You can make cuts interactively on event branches in root.
As an example there is a branch named Muon_pt and we can make the event selection for Muon pt > 17 

~~~
TTree *Events = (TTree*)_file0->Get("Events")
Events->Draw("Muon_pt","Muon_pt>17")
~~~
{: .source}

[Insert output]
{: .output}

The RDataFrame in ROOT offers a high level interface for anlyses of data stored in TTree, 
~~~
#include "ROOT/RDataFrame.hxx"
ROOT::EnableImplicitMT(); // Tell ROOT you want to go parallel
ROOT::RDataFrame df("Events", "root://eospublic.cern.ch//eos/opendata/cms/derived-data/AOD2NanoAODOutreachTool/GluGluToHToTauTau.root"); //Interface to TTree
auto myHisto = df.Histo1D("Muon_pt"); // This happens in parallel!
myHisto->Draw();
~~~
{: .source}

[Insert output]
{: .output}

The RDataFrame can perform transformations (to manupulate the data) and actions (to produce a result from the data). 
The Define and Filter methods are transformations while the Count and Report methods are actions.

Define - Creates a new column in the datset 
Filter - Filters the rows of the dataset
Count  - Returns the number of events processed
Report - Obtains statistics on how many entries have been accepted and rejected by the filters

~~~
std::cout << "Number of events: " << *df.Count() << std::endl;
~~~

Source: (https://root.cern/doc/master/classROOT_1_1RDataFrame.html)

MORE MORE MORE!

{% include links.md %}

