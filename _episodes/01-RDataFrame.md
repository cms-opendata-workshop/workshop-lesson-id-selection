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

To begin, check out the workshop's version of this analysis code:
~~~
$ git clone git://github.com/cms-opendata-workshop/HiggsTauTauNanoAODOutreachAnalysis Htautau
$ cd Htautau
~~~
{: .source}

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
{: .source}

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
{: .source}

The RDataFrame classes in ROOT offer column-based processing that is useful for speeding up analyses using ROOT files.
RDataFrame also allows for multi-threading. To draw the same set of 3 histograms from the review example above, we
will need to use some "filters".

~~~
#include "ROOT/RDataFrame.hxx"
ROOT::EnableImplicitMT(); // Tell ROOT you want to go parallel
ROOT::RDataFrame df("Events", "root://eospublic.cern.ch//eos/opendata/cms/derived-data/AOD2NanoAODOutreachTool/GluGluToHToTauTau.root"); //Interface to TTree

auto myHisto = df.Histo1D("Muon_pt"); // This happens in parallel!
myHisto->Draw();

auto df2 = df.Define("Muon_pt_highPt","Muon_pt[Muon_pt > 17]")
auto df3 = df2.Filter("value_jet_n > 10")

auto myHisto2 = df2.Histo1D("Muon_pt_highPt"); // new branch in a dataframe without extra cuts
auto myHisto3 = df3.Histo1D("Muon_pt"); // old branch in a dataframe with a new cut applied

myHisto2->Draw("pe same");
myHisto3->Draw("hist same");
FIXME
~~~
{: .source}

An RDataFrame can perform transformations (to manupulate the data) and actions (to produce a result from the data). 
The `Define` and `Filter` methods are transformations while the `Count` and `Report` methods are actions.

 * Define: Creates a new column in the datset 
 * Filter: Filters the rows of the dataset
 * Count: Returns the number of events processed
 * Report: Obtains statistics on how many entries have been accepted and rejected by the filters

~~~
std::cout << "Number of events: " << *df.Count() << std::endl;
~~~
{: .source}

>## Challenge: replicate histograms with RDataFrame
>
>Perform the examples above and confirm that you get matching histograms from either method.
>Are you able to tell a difference in speed for this small test?
{: .challenge}

{% include links.md %}

