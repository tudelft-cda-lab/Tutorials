
FlexFringe is a tool for learning state machine models from input traces. It is based on the classic red-blue fringe state merging algorithm but uses flexible evaluation functions that can be used to learn a diverse set of models. In this post, we use FlexFringe to learn one of the simplest state machines: a deterministic finite state machine (DFA). In addition, we show the effect of several basic control variables on the learning algorithm, and how to set them on your own data. If this is your first time running FlexFringe, please run the getting_started tutorial first. A DFA is basically a classifier for sequences. Given a sample input file [traces](models/tutorial1.traces "traces") with 2000 traces in Abbadingo format:

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

We call Flexfringe using the evidence-driven state-merging evaluation (EDSM) function with the command:

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

Every step made by the algorithm is output to the console, the meaning of which is explained in the getting_started tutorial. Basically, we want to perform merges with a lot of evidence, i.e., the number next to m should be large. States are added to the model when they are inconsistent with all current states, the number next to an x gives the number of occurrences of an added state.

In the run given above, we are happy with the large sizes of added states and the large merge scores. Several low evidence merges are performed, two of which are based on the evidence from a single state pair. It is possible but not unlikely that these merges are correct. When learning a classifier or predictor, such merges are fine because the difference in (predictive) accuracy is negligible. When using a learned state machine to provide insight into the underlying system, however, performing such a merge can add several edges and states to the model. This can be problematic because they can be used to draw incorrect inferences such as the presence of loops. To overcome this, FlexFringe has a lowerbound parameter, that sets a lowerbound on the allowed merge score. Another option is to define sinks states, but these are to topic of another tutorial.

**Lower Bound.** We add the line

```
lowerbound = 10
```

to edsm.ini, resulting in radically changed algorithm behavior:

```
 x1345  x603  x266  x195  x152  x145  x78  x35  x24  x18  x16  x15  x12  x9  x8  x8  x6  x5  x5  x5  x4  x4  x4  x4  x3  x3  x3  x3  x3  x3  x3  x2  x2  x2  x2  x2  x2  x2  x2  x2  x2  x2  x2  x2  x2  x2  x2  x2  x2  x2  x2  x2  x2  x2  x2  x2  x2  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  m127  x3  x3  x3  x3  x3  x3  x2  x2  x2  x2  x2  x2  x2  x2  x2  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  m146  x3  x3  x2  x2  x2  x2  x2  x2  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  m97  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  m70  x2  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  m36  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  m35  x1  x1  x1  x1  m20  x1  x1  x1  x1  x1  x1  m18  x1
```

So, what is going on? It might look like something is wrong, but this is the intended effect of using a lowerbound value. Without a lowerbound, any state-pair that is consistent can be merged. With a lowerbound l, the state-pair in addition needs to achieve a merge score of at least l. In EDSM, the merge score is simply the number of performed merges, so at least 10 merges need to be performed during the determinization process. Let us have a look at how the state machine structure progresses during the EDSM run (red states in solid lines, blue states in dashed lines) :

![image of learned state machine](models/tutorial_lowerbound1.png)

It starts with a root (solid) and 3 children (dashed). The solid states are identified and its children are options for merges. The algorithm adds one of the dashed states to the solid core:

![image of learned state machine](models/tutorial_lowerbound2.png)

And several more, until the point where it would have performed a merge before we added the lowerbound constraint:

![image of learned state machine](models/tutorial_lowerbound3.png)

At this point it keeps adding states to the core, learning new states in the state machine model, even with single occurrences (x1 in the algorithm run output), essentially forming a tree:

![image of learned state machine](models/tutorial_lowerbound4.png)

Until finally it performs the merges it perfomed at the start of the run without the lowerbound constraint (m127, m146, m97), resulting in:

![image of learned state machine](models/tutorial_lowerbound5.png)

This gives essentially a tree, with some merges close to the root. The FlexFringe algorithm added all those states to the core because these failed the lowerbound criterion, i.e., they could not be merged consistently with any other state and obtain a merge score of at least 10. In the default setting, whenever the algorithm finds a (blue) state that cannot be merged with any other (red) state, it adds that state to the core (coloring it red). This behavior can be changed by setting the extend parameter to false.

**Extend.** We add the line

```
extend = 0
```

to edsm.ini, resulting in:

```
m64  x1475  m140  x1111  m166  x565  m33  x490  m24  m31  m17  m18  x299  m75  x236  m59  x184  m21  m20  x134  m27  x15  x13  x12  x9  x8  x5  x5  x3  x3  x3  x2  x2  x2  x2  x2  x2  x2  x2  x2  x2  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1 
 ```
FlexFringe performs a merge immediately:

![image of learned state machine](models/tutorial_lowerbound_extend0.png)
![image of learned state machine](models/tutorial_lowerbound_extend1.png)

and results in a small model where states that fail the lowerbound constraint are left as subtrees:

![image of learned state machine](models/tutorial_lowerbound_extend2.png)

This is not a bad model to have. Essentially, the merges for which the algorithm has confidence (the smallest number of merged states is 17 during determinization) are performed. The remaining part of the input data remains intact. Depending on your data, however, there can still be a problem in that is first merges a state (reached by symbol 2) that occurs only 177 times, before it decides what to do with the state reached by 1, which occurs 1345 times. Note that performing merges updates such counts for later iterations. Although this can be good for some datasets, it seems counterintuitive to first makes decisions on states that occur less frequently. To control this behavior, FlexFringe includes a largestblue parameter.



You may also notice that the algorithm starts with several extend actions before perfoming merges. Let us first investigate how to modify this initial behavior. In images, the initial part of the run looks as follows:

**Largestblue.** We add the line

```
largestblue = 1
```

to edsm.ini, resulting in:

```
x1345  x603  m127  m146  x518  x432  x266  m15  m97  x190  x162  m71  x112  m36  m15  m54  m45  m23  m25  m21  m20  x15  x13  x12  x9  x8  x5  x5  x3  x3  x3  x2  x2  x2  x2  x2  x2  x2  x2  x2  x2  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1
 ```

and the same final state machine model:

![image of learned state machine](models/tutorial_largestblue.png)

Now it does first extend more frequent states before merging less frequent ones. This has the effect that the first merge it performs has more confidence (127 merged state pairs, instead of 64). Since the merges influence each other, this can lead to better models, but also to worse ones. It all depends on the data you have available and the used heuristic functions. Importantly, since the merges influence the occurence counts, a heuristic based on statistics can be influenced by merges in such a way that consistent pairs become inconsistent and the other way around. The early implementations of state merging algorithms, such as basic Alergia and RPNI, used a shallow-first (close to the root) search order. Flexfringe includes this as an option using the shallowfirst parameter. Although we rarely use it in practice, it can be useful when experimenting with different merging strategies.

**Shallowfirst.** We add the line

```
shallowfirst = 1
```

to edsm.ini, resulting in:

```
 x1345  m127  x1003  x340  m23  x145  x2  x1  x1  x1  x1  x1  x1  x1  x11  x3  x1  x1  x1  x1  x1  x7  x4  x1  x1  x1  x1  x1  x3  x3  x1  x1  x1  x1  x1  x2  x1  x1  x1  x1  x1  x2  x1  x1  x1  x1  x1  x1  x1  x36  x20  x4  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x2  x1  x1  x1  x1  x1  x7  x3  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x2  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  m12  m160  x1  x1  x1  x1  x2  x2  x1  x1  x3  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  m11  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  m49  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x2  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x3  x1  x1  x1  x1  x1  x1  x1  x1  x1  x2  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x5  x2  x2  x1  x1  x1  x1  x1  x1  x2  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x3  x2  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x2  x2  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x2  x3  x2  x2  x1  x1  x1  x1  x1  x2  x1  x1  x1  x1  x1  x1  x1  x1  x2  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x2  x4  x3  x3  x2  x2  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x7  x1  x2  x1  x1  x1  x2  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x2  x1  x1  x1  x3  x2  x2  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  m100  x1  x1  x1  x310  m79  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x184  m20  x5  x1  x1  x1  x1  x1  x1  x2  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x12  x3  x1  x1  x1  x2  x2  x1  x2  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  m21
 ```

and a much larger state machine model, caused by the interaction between lowerbound and shallowfirst parameters (suddenly many states are considered first that occur infrequently):

![image of learned state machine](models/tutorial_largestblue.png)

Let's remove the shallowfirst parameter from edsm.ini. The last parameter we discuss in this tutorial is finalred. This parameter prevents FlexFringe from adding new transitions to red states, i.e., a new transition cannot be added by a merge (also during determinization) to a state that is already in the core. Intuitively, the states in the core are already identified (learned). Changing the already identified structure using data from later (and often less certain) merges can therefore be considered bad. Let's see what it does to our learning process.

**Finalred.** We add the line

```
finalred = 1
```

to edsm.ini, resulting in:

```
x1345  x603  m127  x459  x340  x296  x266  m47  m83  m14  m92  x162  m21  x190  m71  x112  m36  m15  m54  m45  m23  m25  m21  m20  x15  x13  x12  x9  x8  x5  x5  x3  x3  x3  x2  x2  x2  x2  x2  x2  x2  x2  x2  x2  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1  x1
```
and a seemingly good, but different model:

![image of learned state machine](models/tutorial_finalred.png)

Comparing the performed merges to the algorithm output from the largestblue run, we observe that the second merge

```
x1345  x603  m127  m146
```

is not performed. Instead the considered blue state is added to the red core (x459). These two plots show why:

![image of learned state machine](models/tutorial_finalredm4.png)
![image of learned state machine](models/tutorial_largestbluem4.png)

The merge would add a transistion with label 2 to the state reached by a 1 from the root. Although this state occurs only 3 times in total, the structural change is considered sufficient to prevent this merge from happening. This is the main purpose of the finalred parameter. Whether such prevention is good or bad depends on your data and use of the learned model. If you only intend to use them as predictors/classifiers, then having only a few traces create a larger model seems wrong from a model selection perspective. If you intend to visualize the model in order to obtain insight, however, then not using finalred incorrectly shows that the trace 1 2 can occur, while in fact it never did. This is unavoidable in machine learning since every algorithm generalizes to unseen events. However, limiting it to not yet identified states seems sensible. What to do of course also depends on your data. Is the presense/absense of a symbol a strong indicator for the transitions that are possible? In our view, in this case, it is perfectly defendable to not merge a state with 459 occurrences out of which 3 are a 2 symbol, with one that occurs 2305 times out of which none are a 2 symbol. We frequently use the finalred parameter in our experiments, but we advice that you try and see the effect it has on your model.

**Predictive performace vs insight**

....

For this tutorial, let's see what is most likely we can cheat a little bit by increasing the number of traces

If you use FlexFringe in your research paper, please consider citing the following reference:

> @inproceedings{verwer2017flexfringe,
>  title={Flexfringe: a passive automaton learning package},
>  author={Verwer, Sicco and Hammerschmidt, Christian A},
>  booktitle={2017 IEEE International Conference on Software Maintenance and Evolution (ICSME)},
>  pages={638--642},
>  year={2017},
>  organization={IEEE}
>}
