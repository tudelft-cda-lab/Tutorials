# Model selection

FlexFringe is a tool for learning state machine models from input traces. 
It is based on the classic red-blue fringe state merging algorithm but uses flexible evaluation functions 
that can be used to learn a diverse set of models. 
In this post, we analyze the model selection process for FlexFringe, given the endless parameter combinations that it allows. 

In general, the model parameters depend very much on the input data. In this post, let us consider traces representing intrusion detection alerts from a penetration testing competition.

Several teams compete to compromise a fictitious automotive company's infrastructure. The intrusion alerts resulting from attacker actions are converted into input traces for 
FlexFringe by following these steps:
-	Aggregate all alerts between each pair of hosts (A <> B) in a separate sequence, while also aggregating alerts into user-actions.
-	For each item in a sequence, extract the _attack stage_ (indicating the intent of the attacker) and the _targeted service_ (indicating the target of an attack action) to form a an input trace.

The resulting [trace file](models/prefix-traces.txt "prefix-traces") for all student teams looks as follows:

```
536 112
1 3 infoD|http(s) infoD|http(s) CnC|http(s)
1 6 serD|unassigned vulnD|storage serD|unassigned vulnD|storage serD|unassigned vulnD|storage
1 13 serD|unassigned vulnD|storage serD|unassigned vulnD|storage hostD|remoteAccess surf|remoteAccess infoD|remoteAccess serD|unassigned vulnD|storage hostD|http(s) surf|http(s) infoD|http(s) netDOS|http(s)
1 8 surf|remoteAccess infoD|remoteAccess surf|remoteAccess serD|unassigned vulnD|storage infoD|remoteAccess hostD|remoteAccess serD|unassigned
1 6 serD|unassigned vulnD|storage serD|unassigned vulnD|storage serD|unassigned vulnD|storage
1 6 serD|unassigned vulnD|storage serD|unassigned vulnD|storage serD|unassigned vulnD|storage
1 8 serD|unassigned vulnD|storage serD|unassigned vulnD|storage serD|unassigned vulnD|storage infoD|broadcast surf|broadcast
1 6 serD|unassigned vulnD|storage serD|unassigned vulnD|storage serD|unassigned vulnD|storage
1 9 serD|unassigned vulnD|storage serD|unassigned vulnD|storage serD|unassigned vulnD|storage surf|http(s) infoD|http(s) netDOS|http(s)
1 3 hostD|unassigned serD|unassigned vulnD|storage
1 6 hostD|wireless serD|unassigned vulnD|storage surf|wireless infoD|wireless netDOS|http(s)
1 13 surf|http(s) serD|unassigned vulnD|storage infoD|http(s) rPrivEsc|http(s) acctManip|http(s) remoteexp|http(s) CnC|http(s) ACE|http(s) resHJ|http(s) dManip|http(s) exfil|http(s) delivery|http(s)
1 15 serD|unassigned serD|unassigned vulnD|storage surf|http(s) vulnD|http(s) infoD|http(s) rPrivEsc|http(s) acctManip|http(s) remoteexp|http(s) CnC|http(s) ACE|http(s) resHJ|http(s) dManip|http(s) exfil|http(s) delivery|http(s)
1 14 serD|ATCcomm vulnD|storage surf|http(s) vulnD|wireless infoD|http(s) rPrivEsc|wireless acctManip|wireless remoteexp|wireless CnC|http(s) ACE|http(s) resHJ|wireless dManip|http(s) exfil|http(s) delivery|http(s)
1 4 serD|unassigned vulnD|storage serD|unassigned vulnD|storage
...
```

The objective is to learn a state machine that summarizes the activities (strategies) of all attacker teams in the dataset. There are two key challenges in learning a suitable model:
-	There exists a huge imbalance between low-severity symbols, i.e. those referring to the reconnaissance attack stage, and high-severity symbols, i.e. those referring to exploitative attack stages. Ideally, the model should bring the infrequent high-severity symbols into the spotlight without discarding frequent low-severity symbols.
-	The symbols have a contextual meaning as well, i.e. scanning symbols occurring at the start of the competition may indicate reconnaissance, while scanning symbols occurring mid-way through the competition may indicate attack progression. Ideally, we want the model to capture this context, and model different paths for identical symbols that have a different contextual meaning. 

With the goals set in mind, we learn the first DFA as per default settings. 
We color the high-severity states as red and medium-severity states as blue to make the analysis easier.

![Default PDFA model](models/tut4-prefix-default.png)

We immediately see the impact of the frequency imbalance between high- and low-severity symbols. Since high-severity actions usually occur towards the end, the traces reflect that. Consequently, the high-severity symbols lie towards the bottom of the model. 

Additionally, due to their low frequency, a significant portion of the _Sink states_ (27.31%) is composed of severe symbols, since there is not much that can be computed statistically over these symbols. 

First, we experiment with the trace and symbol counts. 

The first setting we use limits which states are used to compute statistics. The learning algorithm merges two states if it does not find sufficient evidence that the states are different. A lower bound on the data required for this evidence is controlled by the state_count and symbol_count parameters.

Since the overall ratio between symbols (112)  and traces (536) is not too high, the default parameters are set too high. 
The default values of 50 and 25 are much too large for the limited amount of high-severity states in the dataset. We set both to 5, implying that a state in the suffix tree that occurs only 5 times in total can provide sufficient evidence to prevent a merge from happening.

![PDFA with state and symbol count set](models/tut4-prefix-with-sscount.png)

The core algorithm continues merging until all states have either been merged or added to the model. For states that occur infrequently, there is typically insufficient evidence to prevent a merge and they can therefore be merged with any of the states added in previous iterations. The sink_count parameter avoids this by disallowing merges that occur sink_count times or less, which we set to 5.

![PDFA with sink count set](models/tut4-prefix-with-sinks.png)
 

Next, in order to address the symbol imbalance problem, we learn a suffix-based model instead of a prefix-based one. To this end, we [reverse all the input traces](models/suffix-traces.txt "suffic-traces") before giving it to FlexFringe.
First, a suffix tree is learnt, which can be interpreted as follows:

![Suffix tree example](img/suffix-tree-example.png)

After merging, the following model is generated:

![Suffix-based PDFA model](models/tut4-suffix-model)

The interpretation of this model is tricky. In essence, this model predicts the past symbols based on the future symbols. It seems a little absurd, but the idea is that given a particular high-severity symbol _x_, we want to predict all paths that lead to _x_. 

As the high-severity symbols now occur at the beginning of the traces, and the appropriately set symbol count, we achieve the first goal of bringing infrequent severe symbols into the spotlight. 

Next, the merge criteria is important to incorporate contextual modelling. The context of a symbol in a trace is determined by its future and past states. As such, a merge criteria determines which states are equivalent to be merged. The default merge criteria in this set up is _alergia_, which is a purely probabilistic merge criteria, i.e. it computes statistics over future states (i.e., chronological past) in order to decide whether to merge two states. This results in states that have incoming transitions with different symbols, which makes interpretability difficult. 

Instead, we choose to use a mixture of _Markovian_ and probabilistic merge criteria. 
This property dictates that for any given states _q1_ and _q2_, the label _a_ and _b_ of the immediate previous transitions _<q1’,q1,a> and _<q2’,q2,b> have to be identical _a_ = _b_. It enforces that the incoming transition label for states is unique. This makes the model, especially states with high-severity incoming transitions, easier to interpret.

The resulting model looks as follows:

![Suffix-based PDFA with markovian properties](models/tut4-suffix-markovian.png)

This model fulfills both the criteria that we set upon at the start: highlights severe symbols and models context. Hence, this is the model we use in the next steps.
