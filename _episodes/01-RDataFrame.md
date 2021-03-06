---
title: "Higgs to Tau Tau analysis"
teaching: 5
exercises: 10
questions:
- "How is the Higgs -> Tau Tau analysis example set up?"
objectives:
- "Checkout Higgs -> Tau Tau code for the workshop"
- "Review the basic RDataFrame commands for filtering and defining variables"
keypoints:
- "The NanoAOD samples required for this analysis include data, simulated signals, and several simulated backgrounds"
- "RDataFrame tools are useful for efficiently manipulating data and plotting histograms."
---

>## Setup
>To begin, check out the workshop's version of this analysis code if you have not done so already:
>~~~
>$ cd ~/CMSSW_5_3_32/src/
>$ git clone git://github.com/cms-opendata-workshop/HiggsTauTauNanoAODOutreachAnalysis Htautau
>$ cd Htautau
>~~~
>{: .language-bash}
{: .prereq}

## Data and simulation files

You learned in the Data Scouting lesson to search for CMS Open Data samples, and in the previous lesson we discussed how to
run the AOD2NanoAOD tool over multiple files to incorporate your changes to `AOD2NanoAOD.cc`. For the sake of time in the
workshop, data and simulation NanoAOD samples have already been produced for you. 

Data samples to be used in the analysis:

 * Run2012B_TauPlusX
 * Run2012C_TauPlusX

Signal simulations to be used in the analysis:

 * GluGluToHToTauTau (gluon-gluon fusion Higgs production)
 * VBF_HToTauTau (vector boson fusion Higgs production)

Background processes can produce very similar signatures in the detector which have to be considered in the anaylsis.
The most prominent background processes with a similar signature include:

 * DYJetsToLL (gamma*/Z production with decays to leptons)
 * TTbar (top quark pair production)
 * W1JetsToLNu (W boson produced with 1 jet)
 * W2JetsToLNu (W boson produced with 2 jets)
 * W3JetsToLNu (W boson produced with 3 jets)

All of these files can be accessed from the "eospublic" realm:

~~~
$ root -l root://eospublic.cern.ch//eos/opendata/cms/derived-data/AOD2NanoAODOutreachTool/GluGluToHToTauTau.root
[0] TBrowser b
~~~
{: .language-bash}

## Review of TTree::Draw and RDataFrame

>Before moving on, please follow the pre-exercises on ROOT (especially TTree::Draw) and RDataFrame if you did not do so earlier.
{: .callout}

To review: the Events tree inside the NanoAOD file can be used to draw histograms of branches within the tree, and cuts can be
performed using any branch in the tree. 

~~~
[0] TTree *Events = (TTree*)_file0->Get("Events")
[1] Events->Draw("Muon_pt")  // draws a histogram of muon momentum
[2] Events->Draw("Muon_pt","Muon_pt > 17") // draws a histogram of muon momentum for muons with pT > 17 GeV
[3] Events->Draw("Muon_pt","value_jet_n > 10") // draws a histogram of muon momentum in events with more than 10 jets
~~~
{: .language-bash}

The RDataFrame classes in ROOT offer column-based processing that is useful for speeding up analyses using ROOT files.
RDataFrame also allows for multi-threading. To draw the same set of 3 histograms from the review example above, we
will need to use some "filters".

```cpp
[0] ROOT::EnableImplicitMT() // Tell ROOT you want to go parallel
[1] ROOT::RDataFrame df("Events", "root://eospublic.cern.ch//eos/opendata/cms/derived-data/AOD2NanoAODOutreachTool/GluGluToHToTauTau.root"); //Interface to TTree

[2] auto myHisto = df.Histo1D("Muon_pt"); // This happens in parallel!

[3] auto df2 = df.Define("highPtMuons","Muon_pt > 17");
[4] auto df3 = df2.Define("Muon_highpt","Muon_pt[highPtMuons]");
[5] auto df4 = df3.Filter("nJet > 10");

[6] auto myHisto2 = df3.Histo1D("Muon_highpt"); // new branch in a dataframe without extra cuts
[7] auto myHisto3 = df4.Histo1D("Muon_pt"); // old branch in a dataframe with a new cut applied

[8] myHisto->Draw();
[9] myHisto2->Draw("pe same");
[10] myHisto3->Draw("pe same");
```


An RDataFrame can perform transformations (to manupulate the data) and actions (to produce a result from the data). 
The `Define` and `Filter` methods are transformations while the `Count` and `Report` methods are actions.

 * Define: Creates a new column in the datset 
 * Filter: Filters the rows of the dataset
 * Count: Returns the number of events processed
 * Report: Obtains statistics on how many entries have been accepted and rejected by the filters

~~~
std::cout << "Number of events: " << *df.Count() << std::endl;
~~~
{: .language-cpp}

>## Challenge: replicate histograms with RDataFrame
>
>Perform the examples above and confirm that you get matching histograms from either method.
>Are you able to tell a difference in speed for this small test?
>To run the RDataFrame example you will need to have cvmfs installed on your local machine
>and mounted in your container. Source the proper ROOT environment and get a ROOT command line:
>~~~
>$ source /cvmfs/sft.cern.ch/lcg/views/LCG_95/x86_64-slc6-gcc8-opt/setup.sh
>$ root -l
>~~~
>{: .language-bash}
{: .challenge}

{% include links.md %}

