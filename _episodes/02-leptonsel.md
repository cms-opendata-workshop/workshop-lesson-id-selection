---
title: "Event selection"
teaching: 10
exercises: 10
questions:
- "How can I select events in which to search for Higgs -> tau tau?"
objectives:
- "Understand typical object selection criteria"
- "Implement selections for the muon and tau examples"
- "Begin the workshop's physics analysis example"
.keypoints:
- "Selection criteria include kinematic limits (momentum and angle), identification, and isolation."
- "Kinematic limits are usually based on detector ranges and the physics process being studied."
- "Identification and isolation criteria depend mostly on your physics analysis goals."
---

Event selection for the Higgs -> tau tau search is performed in the file called `skim.cxx`. As we select events, our goal is to keep the signal
events from Higgs to tau tau decay and suppress the background events from other sources (Drell-Yan, W boson, top quarks, etc).
Many background processes can be suppressed by introducing kinematic limits on the physics objects.
For instance, a W boson could decay to a muon and neutrino, similar to a tau decay, but only the W boson background processes should have
the transverse mass of the muon and neutrino (modeled by the missing transverse energy) near 80 GeV. Cuts on key distinguishing variables
can strongly suppress background.

## Trigger and final state definition

The first event selection requirement should be a set of triggers. In fact, we have made a hidden trigger selection by analyzing the
"TauPlusX" datasets! Only events passing some tau-related trigger will enter our data files.

In the `MinimalSelection` function we will filter events by requiring they fire a tau + muon trigger -- this means we are choosing
to search for events in which one of the tau leptons decayed to a muon. We will certainly reject signal events based on this
requirement, but narrowing the final state makes the relevant background processes more clear and simplifies the search procedure. 

~~~
template <typename T>
auto MinimalSelection(T &df) {
    return df.Filter("HLT_IsoMu17_eta2p1_LooseIsoPFTau20 == true", "Passes trigger")
}
~~~
{: .source}

>## Challenge: require muons and taus
>
>In the same function, require that each event have at least one muon and at least one tau lepton. (hint: .Filter() commands can be chained together in the same statement)
>Open a VBF signal ROOT file and use TTree::Draw to plot a histogram of the number of tau leptons in each event, and then the number of muons in each event.
>What do the distributions tell you about the sources of these leptons?
>TESTME
{: .challenge}

## Lepton selection criteria

We clearly need to slim down the number of muons! The typical event has far more than 2 muons from tau decays. The `FindGoodMuons` function will add a new column
to the dataframe that defines a "good muon" for the purpose of our analysis.

>What constitutes a good muon?
{: .discussion}

For any physics object, the selection criteria typically include:
 * kinematic constraints (momentum, pseudorapidity, masses of object pairs, etc)
 * identification requirements (loose, medium, tight quality levels)
 * isolations requirements (loose, medium, tight isolation levels)

Which kinematic constraints should be applied? The first place to look is the trigger path: "HLT_IsoMu17_eta2p1_LooseIsoPFTau20".
This trigger path selects events with an isolated muon that has pT > 17 GeV and |eta| < 2.1; along with a loose, isolation tau lepton with pT > 20 GeV.
In an ideal case, "offline" (post-trigger) selection criteria are universally more restrictive than "online" (trigger) selection criteria, especially
in terms of transverse momentum. This allows analysts to avoid the murky kinematic regions of "trigger turn-ons" in which data and simulation might respond
differently to the application of the trigger because of small differences in the input variables.

>## Challenge: define good muons
>
>Complete the `FindGoodMuons` function with selection criteria on muon momentum, pseudorapidity, identification level, and isolation level.
>
>~~~
>template <typename T>
>auto FindGoodMuons(T &df) {
>    return df.Define("goodMuons", "abs(Muon_eta) < 2.1 && Muon_pt > 17 && Muon_tightId == true");
>}
>~~~
>{: .source}
{: .challenge}

We use this function to find the interesting taus in the tau collection. These tau candidates represent hadronic decays of taus which means that
the tau decays to combinations of pions and neutrinos in the final state. Add your cuts on tau eta and pt similar to how it was done for muon.

Also add requirements for the tau charge, Tau_idDecayMode, Tau_idIsoTight, Tau_idAntiEleTight, Tau_idAntiMuTight.

Notice that we have added an isolation requirement for our tau. 
Hard processes produce large angles between the final state partons. The final object of interest will be separated from 
the other objects in the event or be isolated. For instance an isolated muon from a W. In contrast, a non-isolated muon can come from
a weak decay inside a jet. Isolation variables are ET and pT sums in cones (drawn around the object) in eta-phi space. 
Isolation of a muon is done relative to detector objects such as detector hits and tracks.
~~~
template <typename T>
auto FindGoodTaus(T &df) {
    return df.Define("goodTaus", "PUT YOUR SELECTIONS HERE");
}
~~~
{: .source}

Implement the selections on muon and tau in a copy of the HiggsTauTauNanoAODOutreachAnalysis repository provided for you.

 
We can reduce the dataset to the interesting events containing at least one interesting
muon and tau candidate.

~~~
template <typename T>
auto FilterGoodEvents(T &df) {
    return df.Filter("Sum(goodTaus) > 0", "Event has good taus")
             .Filter("Sum(goodMuons) > 0", "Event has good muons");
}
~~~
{: .source}

We can then use the functions we have created to perform event selections on the GluGluToHToTauTau sample.

~~~
ROOT::EnableImplicitMT();
ROOT::RDataFrame df("Events", "root://eospublic.cern.ch//eos/opendata/cms/derived-data/AOD2NanoAODOutreachTool/GluGluToHToTauTau.root");
auto df2 = MinimalSelection(df);
auto df3 = FindGoodMuons(df2);
auto df4 = FindGoodTaus(df3);
auto df5 = FilterGoodEvents(df4);
~~~

~~~
auto dfFinal = df5;
auto report = dfFinal.Report();
dfFinal.Snapshot("Events", sample + "Skim.root", finalVariables);
report->Print();
~~~
{: .source}


{% include links.md %}

