
FlexFringe is a tool for learning state machine models from input traces. It is based on the classic red-blue fringe state merging algorithm but uses flexible evaluation functions that can be used to learn a diverse set of models. In this post, we use FlexFringe to learn one of the simplest state machines: a deterministic finite state machine (DFA). A DFA is basically a classifier for sequences. Given a sample input file [traces](models/tutorial1.traces "traces") with 2000 traces:

```
2000 5
0 9 1 1 1 3 0 0 2 3 1
0 3 1 1 3
0 0
0 13 1 3 0 3 0 1 1 1 1 3 0 3 3
0 9 1 3 0 1 1 3 0 1 3
0 0
1 4 4 2 2 0
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
x1345  x603  x266  x195  x152  x145  m127  m146  m97  m70  m36  m25  m31  m19  m19  m18  m17  m6  m5  x9  m63  m27  m4  m4  m4  m3  m1  m1  no more possible merges
```

As well as a learned model in both .dot and .json format, running:

```
dot -Tpdf tutorial1.traces.ff.final.dot -o tutorial1.png
```
gives

![image of learned state machine](models/tutorial1.png)

Every step made by the algorithm is output to the console. An x means that the algorithm identifies a new state in the model. The algorithm tried all possible merges between this state and the existing ones, and all failed to meet the consistency criterion set in the EDSM evaluation function (inherited from the count_driven class):

```c++
bool count_driven::consistent(state_merger *merger, apta_node* left, apta_node* right){
    if(inconsistency_found) return false;
  
    count_data* l = (count_data*)left->data;
    count_data* r = (count_data*)right->data;

    if(l->pos_final() != 0 && r->neg_final() != 0){ inconsistency_found = true; return false; }
    if(l->neg_final() != 0 && r->pos_final() != 0){ inconsistency_found = true; return false; }
    
    return true;
};
```

This code first casts the data type in the states (apta_node*) to count_data, giving access to the data and functions required for the test, and then checks a pair of merged states during determinization, whether one is positive (l->pos_final() != 0) and the other negative (r->neg_final() != 0). If so, the resulting model is considered inconsistent because there exists a state that both a positive and a negative trace end in. The method sets found_inconsistency to true and returns false. If not, the method returns true, indicating that the pair of states can be merged. This method is called for ever pair of states that are merged during determinization.

The number next to the x is the number of occurrences of the extended state. It also prints the performed merges m. Every m means a merge between an identified and not yet identified state has been performed, identifying a new transition in the final model. The number next to the m is the score of the EDSM evaluation function:

```c++
void evidence_driven::update_score(state_merger *merger, apta_node* left, apta_node* right){
    edsm_data* l = (edsm_data*) left->data;
    edsm_data* r = (edsm_data*) right->data;

    if(l->pos_final() > 0 && r->pos_final() > 0) num_pos += 1;
    if(l->neg_final() > 0 && r->neg_final() > 0) num_neg += 1;
};
```

This counts, over all pairs of states merged during determinization, the number of merdeg positive-positive and negative-negative state pairs. The method adds 1 to a counter for all such merges and returns the total count in the compute_score function:

```c++
double evidence_driven::compute_score(state_merger *merger, apta_node* left, apta_node* right){
    return num_pos + num_neg;
};
```

Every pair of positive-positive and negative-negative states is considered evidence that the merge is correct. The core state merging algorithm is greedy and in every iteration tries all possible merges between identified states and their children. From all consistent merges, it then selects and performs the one with largest evidence score. When there are no consistent merges, it extends the identified states by adding the not yet identified state with the largest number of occurences. The evidence (and occurrence) values are larger at the start of the merging process because the algorithms initially perform merges at the root of a prefix tree:

![image of prefix tree](models/tutorial1-1.png)

All input traces go through the root, resulting in high initial ocurrences and evidence values. Later in the process, the merge scores become smaller and smaller because the lower levels of model are reached by fewer and fewer input traces. When learning models, it is important to monitor the merge scores and make sure they do not drop become too small (meaning merges are performed based on very little evidence). This can be controlled using the lowerbound parameter and by using sink states. In the run above, the final merge scores are low and we might consider putting a lowerbound of 10 to avoid making those merges. We discuss these and other parameter settings in another post. The early large merge scores give us confidence however that most of the identified states in the learned state machine are correct:

![image of learned state machine](models/tutorial1.png)

The root state is indicated by the rectangle. The information printed in a state, i.e:

```
1:#4826
0:0 - 1:116 -
```

means that this is state number 1 (a simple identifier), it occurs 4826 times in total in the training data (a state can occur multiple times in one trace), 0 negative traces (0:0) end in this state, and 116 positive traces end in it (1:116). For a transition, i.e.:

```
0
0:2604 - 1:2106 -
```

The first symbol is the label of the transition (the event triggering this transition to fire), it occurs 2604 times in negative traces, and 2106 times in positive traces. Note that the information under the first line is printed by the evaluation function, thus using a different function will likely result in different information being printed. We also provide a python notebook that can be used to run traces through the learned model (using the .json file as input) for instance to compute the accuracy on a test set. This will be described in a later tutorial. For now, it is most important to understand the basic steps, including consistency checks, and the output of the learning algorithm. Here we show several steps of FlexFringe on the tutorial input file, resulting in smaller and smaller models. The dashed states are not yet identified parts of the prefix tree, the solid parts are the identified ones:

![image of learned state machine](models/tutorial1-2.png)
![image of learned state machine](models/tutorial1-3.png)
![image of learned state machine](models/tutorial1-4.png)
![image of learned state machine](models/tutorial1-5.png)
![image of learned state machine](models/tutorial1-6.png)
![image of learned state machine](models/tutorial1-7.png)
![image of learned state machine](models/tutorial1-8.png)
![image of learned state machine](models/tutorial1-9.png)
![image of learned state machine](models/tutorial1-10.png)

If you use FlexFringe in your research paper, please consider citing the following reference:

> @inproceedings{verwer2017flexfringe,
>  title={Flexfringe: a passive automaton learning package},
>  author={Verwer, Sicco and Hammerschmidt, Christian A},
>  booktitle={2017 IEEE International Conference on Software Maintenance and Evolution (ICSME)},
>  pages={638--642},
>  year={2017},
>  organization={IEEE}
>}


