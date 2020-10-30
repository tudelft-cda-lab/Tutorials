
FlexFringe is a tool for learning state machine models from input traces. It is based on the classic red-blue fringe state merging algorithm but uses flexible evaluation functions that can be used to learn a diverse set of models. In this post, we use FlexFringe to learn one of the simplest state machines: a deterministic finite state machine (DFA). A DFA is basically a classifier for sequences. Given a sample input file with 20000 traces:

```
20000 8
1 0
1 6 5 0 2 5 4 0
0 9 5 2 1 1 2 7 1 2 1
1 6 5 2 7 7 0 2
0 1 3
1 8 5 2 0 0 5 5 5 5
0 13 2 7 0 2 4 1 1 2 1 1 0 2 4
1 3 5 0 2
…
```

in Abbadingo format:

```
num_traces alphabet_size
class length sequence
class length sequence
class length sequence
…
```

Flexfringe can be called to learn DFA using the evidence-driven state-merging evaluation (EDSM) function with the command:

```
./flexfringe tutorial1.traces --ini ini/edsm.ini
```

Resulting in the following output:

```
x  x  x  x  x  x  m127  m146  m97  m70  m36  m25  m31  m19  m19  m18  m17  m6  m5  x  m63  m27  m4  m4  m4  m3  m1  m1  no more possible merges
```

Every step made by the algorithm is output to the console. An x means that the algorithm extends the red core with another state by coloring a blue state red, identifying a new state in the final model. The algorithm tried all possible merges between this state and the existing red core, and all failed to meet the consistency criterion set in the EDSM evaluation function:


This code checks for every pair of merged states (apta_node) during determinization, whether one is positive and the other negative. If so, the resulting model is considered inconsistent because it is no longer a perfect classifier. The method sets found_inconsistency to true and returns false. If not, the method returns true, indicating that the pair of states can be merged.

The number next to the x is the number of occurrences of the extended state. It also prints the performed merges m. Every m means a merge between a red and a blue state has been performed, identifying a new transition in the final model. The number next to the m is the score of the EDSM evaluation function:


This counts, over all pairs of states merged during determinization, the number of positive-positive and negative-negative pairs. The method adds 1 to a counter for all such merges and returns the total count in the compute_score function, :

Every pair of positive-positive and negative-negative states is considered evidence that the merge is correct. The values are larger at the start of the merging process because the algorithms initially perform merges at the root of a prefix tree:




All input traces go through the root, resulting in high initial evidence values. Later in the process, the merge scores become smaller and smaller because the lower levels of model are reached by fewer and fewer input traces. When learning models, it is important to monitor the merge scores and make sure they do not drop become too small (meaning merges are performed based on very little evidence). This can be controlled using the lowerbound parameter and by using sink states. We discuss these and other parameter settings in another post. In the run above, the scores stay sufficiently high, giving us confidence that the state machine model provided in the form of a Graphviz dot file is (mostly) correct:


The root state is indicated by the rectangle, final states (in which positive traces end) are indicated by double circles, and all the counts of transitions and state occurrence are printed after the # character. We provide a python notebook that can be used to run traces through the learned model for instance to compute the accuracy on a test set:

