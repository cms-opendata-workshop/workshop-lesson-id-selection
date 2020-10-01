---
title: "Event selection"
teaching: 10
exercises: 20
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

In the `MinimalSelection` function we will filter events by requiring they fire a hadronic tau + muon trigger -- this means we are choosing
to search for events in which one of the tau leptons decayed to a muon and the other decayed to hadrons. We will certainly reject signal events based on this
requirement, but narrowing the final state makes the relevant background processes more clear and simplifies the search procedure. 

~~~
template <typename T>
auto MinimalSelection(T &df) {
    return df.Filter("HLT_IsoMu17_eta2p1_LooseIsoPFTau20 == true", "Passes trigger")
}
~~~
{: .language-cpp}

>## Challenge: require muons and taus
>
>In the same function, require that each event have at least one muon and at least one tau lepton. (hint: .Filter() commands can be chained together in the same statement)
>Open a VBF signal ROOT file and use TTree::Draw to plot a histogram of the number of tau leptons in each event, and then the number of muons in each event.
>What do the distributions tell you about the sources of these leptons?
{: .challenge}

## Lepton selection criteria

We clearly need to slim down the number of muons! The typical event has far more than 2 muons from tau decays. The `FindGoodMuons` function will add a new column
to the dataframe that defines a "good muon" for the purpose of our analysis.

>What constitutes a good muon?
{: .testimonial}

For any physics object, the selection criteria typically include:
 * kinematic constraints (momentum, pseudorapidity, masses of object pairs, etc)
 * identification requirements (loose, medium, tight quality levels). We have stored all of these labels as boolean pass/fail variables.
 * isolations requirements (loose, medium, tight isolation levels). When considering the relative isolation variables calculated in the
 previous exercise for muons, "loose" isolation would correspond to values < 0.4 while "tight" isolation would correspond to values < 0.15.

Thinking about the signal process guides our decisions here: in signal events we expect one hadronic tau, one muon, and neutrinos that
were the prompt decay products of a massive particle. The tau and the muon will likely be produced with reasonably high momentum, and
are not likely to be surrounded by energy deposits from other particles. For this physics process we should select hadronic taus and
muons that pass at least their "loose" ID criteria, and perhaps even the "tight" ID criteria. It may help reject background to require
that these leptons be isolated. Making these decisions often involves studying the signal and background efficiencies for a variety of
choices. 

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
>    return df.Define("goodMuons", "BOOLEAN CONDITIONS GO HERE");
>}
>~~~
>{: .language-cpp}
>(hint: Muon_pt, Muon_eta, Muon_tightId, Muon_looseId, etc, are branches of interest.)
{: .challenge}

>What constitutes a good hadronic tau?
{: .testimonial}

Taus that decay to hadrons and neutrinos are identified using similar logic, but with combinations of tau identification discriminants instead of the simpler
loose/medium/tight framework. The [tau ID reference page](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookPFTauTagging#Legacy_Tau_ID_Run_I) gives
a brief description of each type identification variable. They note that we always want to use the "ByDecayModeFinding" discriminant.


>## Challenge: define good taus
>
>Complete the `FindGoodTaus` function with selection criteria on the tau identification discriminants. Based on the trigger path,
>we have already placed some kinematic constraints on the tau. 
>~~~
>template <typename T>
>auto FindGoodTaus(T &df) {
>    return df.Define("goodTaus", "Tau_charge != 0 && abs(Tau_eta) < 2.3 && Tau_pt > 20 && BOOLEAN ID CONDITIONS HERE");
>}
>~~~
>{: .language-cpp}
{: .challenge}

## Selecting on jets and MET

In this analysis we will not explicitly require jets or missing transverse momentum. Analyses expecting MET will
set thresholds on `MET_pt` and might also require that MET is not close to collinear with a jet (aka, require MET
to be "significant"). 

When selecting jets, the "loose" identification criteria that you added in `AOD2NanoAOD.cc` is recommended, and the number of jets required would be
informed by your physics process (keeping in mind that jets can be lost and radiation can produce extra jets).
An extra wrinkle comes in when leptons and jets are expected in the final state: particle-flow jets are clustered
from all particle-flow candidates -- muons and electrons are not removed from this list! For many final state topologies,
it is sufficient to remove "jets" from the list if they closely overlap a tight, isolated muon or electron. If this is
expected based on the physics process (for instance, the decay of a very high momentum top quark to a lepton and b quark jet),
the four-vector of the lepton can be subtracted from the uncorrected four-vector of the jet before the corrections are applied.

## Selecting events

We can now reduce the dataset to the interesting events containing at least one good muon and good tau candidate.

~~~
template <typename T>
auto FilterGoodEvents(T &df) {
    return df.Filter("Sum(goodTaus) > 0", "Event has good taus")
             .Filter("Sum(goodMuons) > 0", "Event has good muons");
}
~~~
{: .language-cpp}

>How should we determine which muon-tau pair for the best Higgs boson candidate?
{: .testimonial}

This is not possible will the simple one-liner RDataFrame commands we have used so far. Instead, we will need a C++
function based on the `RVec` columns. The function is called `build_pair` and it takes several arguments:

 * `goodMuons` and `goodTaus`: these columns were defined in the previous functions and hold a pass/fail (1/0) value
 for each muon or tau in the event
 * momentum, eta, and phi of the muon (`pt_1`, `eta_1` and `phi_1`)
 * isolation, eta, and phi of the tau (`iso_2`, `eta_2`, `phi_2`)

The function finds all possible pairs of a good muon and a good tau, and filters out pairs in which the leptons are
too close together. Then preference is given to the pair with the highest muon pT, and then (if any options remain)
the tau with the lowest isolation. 

~~~
template <typename T>
auto FindMuonTauPair(T &df) {
    using namespace ROOT::VecOps;
    auto build_pair = [](RVec<int>& goodMuons, RVec<float>& pt_1, RVec<float>& eta_1, RVec<float>& phi_1,
                         RVec<int>& goodTaus, RVec<float>& iso_2, RVec<float>& eta_2, RVec<float>& phi_2)
            {
                 // Get indices of all possible combinations
                 auto comb = Combinations(pt_1, eta_2);
                 const auto numComb = comb[0].size();

                 // Find valid pairs based on delta r
                 std::vector<int> validPair(numComb, 0);
                 for(size_t i = 0; i < numComb; i++) {
                     const auto i1 = comb[0][i];
                     const auto i2 = comb[1][i];
                     if(goodMuons[i1] == 1 && goodTaus[i2] == 1) {
                         const auto deltar = sqrt(
                                 pow(eta_1[i1] - eta_2[i2], 2) +
                                 pow(Helper::DeltaPhi(phi_1[i1], phi_2[i2]), 2));
                         if (deltar > 0.5) {
                             validPair[i] = 1;
                         }
                     }
                 }

                 // Find best muon based on pt
                 int idx_1 = -1;
                 float maxPt = -1;
                 for(size_t i = 0; i < numComb; i++) {
                     if(validPair[i] == 0) continue;
                     const auto tmp = comb[0][i];
                     if(maxPt < pt_1[tmp]) {
                         maxPt = pt_1[tmp];
                         idx_1 = tmp;
                     }
                 }

                 // Find best tau based on iso
                 int idx_2 = -1;
                 float minIso = 999;
                 for(size_t i = 0; i < numComb; i++) {
                     if(validPair[i] == 0) continue;
                     if(int(comb[0][i]) != idx_1) continue;

		     // COMPLETE THE FUNCTION
                 }

                 return std::vector<int>({idx_1, idx_2});
            };
}
~~~
{: language-cpp}

>## Challege: select lowest-iso tau
>
>Complete the function above to select the pair with the highest muon pT (already implemented) and the most isolated tau (missing)
{: .challenge}

The rest of the `FindMuonTauPair` function shows how to call `build_pair` in RDataFrame and pass it the necessary columns.
A `Define` statement is used to store the pair index from the output of `build_pair`. This object is a vector of 2 values,
so we can also use `Define` to store the first (muon) and second (tau) index separately. These indices will show which
lepton in the column should be used to form a Higgs boson.

~~~
    return df.Define("pairIdx", build_pair,
                     {"goodMuons", "Muon_pt", "Muon_eta", "Muon_phi",
                      "goodTaus", "Tau_relIso_all", "Tau_eta", "Tau_phi"})
             .Define("idx_1", "pairIdx[0]")
             .Define("idx_2", "pairIdx[1]")
~~~
{: .language-cpp}

>##Challenge: Filter on a good muon and tau in the pair
>
>Add `Filter` statements to the end of `FindMuonTauPair` to filter out events with bad indices for the muon or tau in the pair.
{: .challenge}


Finally, to actually perform event selection we must call all these functions we have created!

~~~
ROOT::EnableImplicitMT();
ROOT::RDataFrame df("Events", "root://eospublic.cern.ch//eos/opendata/cms/derived-data/AOD2NanoAODOutreachTool/GluGluToHToTauTau.root");

auto df2 = MinimalSelection(df);
auto df3 = FindGoodMuons(df2);
auto df4 = FindGoodTaus(df3);
auto df5 = FilterGoodEvents(df4);
auto df6 = FindMuonTauPair(df5);
auto df7 = DeclareVariables(df6); // adds more column definitions
auto df8 = CheckGeneratorTaus(df7, sample); // checks that the generated particles are good
auto df9 = AddEventWeight(df8, sample);

// Final dataframe!
auto dfFinal = df9;
auto report = dfFinal.Report();
dfFinal.Snapshot("Events", sample + "Skim.root", finalVariables);
~~~
{: .language-cpp}

The last line above saves a "snapshot" of the dataframe, which is a ROOT file with a new tree (still called "Events") containing only the
branches listed in "finalVariables".

>## Challenge: go through the final variables
>
>Go through `DeclareVariables` to understand the variables being defined. **Describe in words all the masses computed**.
>If time permits, edit the definition of `goodJets` to require that they are separated from the muon (see the previous definitions in the chain!) by DR > 0.4.
>~~~
>.Define("goodJets", "Jet_puId == true && abs(Jet_eta) < 4.7 && Jet_pt > 30")
>~~~
>{: .language-cpp}
{: .challenge}

## Running the skimmer

With your edits in place, run `skim.cxx` to process all the data and simulation samples. Great time for a break!

~~~
$ source /cvmfs/sft.cern.ch/lcg/views/LCG_95/x86_64-slc6-gcc8-opt/setup.sh
$ g++ -g -O3 -Wall -Wextra -Wpedantic -o skim skim.cxx $(root-config --cflags --libs)
$ ./skim
~~~
{: .language-bash}

{% include links.md %}

