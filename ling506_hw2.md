---
layout: ling506_frame
img: rosetta
img_url: http://www.flickr.com/photos/calotype46/6683293633/
caption: Rosetta stone (credit&#59; calotype46)
title: Homework 2 | Decoding
active_tab: homework
---

<div class="alert alert-info">
This homework is due Saturday 7 March 2015, 11:59pm.
</div>

Decoding:  <span class="text-muted">Challenge Problem 2</span>
=============================================================

Decoding is the problem of taking an input sentence in a foreign language:

*honorables sénateurs , que se est - il passé ici , mardi dernier ?*

and finding the best translation into a target language, according to a model:

*honourable senators , what happened here on Tuesday ?*

**Your task is to find the most probable translation, given the foreign input,
the translation likelihood model, and the English language model.**
We assume the traditional noisy channel decomposition:

$$
\begin{align*}
  \mathbf{e}^* &= \arg \max_e p(\mathbf{e} \mid \mathbf{f})\\
  &= \arg\max_e \displaystyle{p_{TM}(\mathbf{f} \mid \mathbf{e}) \times p_{LM}(\mathbf{e})
     \over p(\mathbf{f})}\\
  &= \arg\max_e \displaystyle p_{TM}(\mathbf{f} \mid \mathbf{e}) \times p_{LM}(\mathbf{e})\\
  &= \arg\max_e \displaystyle \sum_{\mathbf{a}} p_{TM}(\mathbf{f},\mathbf{a} \mid \mathbf{e})
     \times p_{LM}(\mathbf{e})
\end{align*}
$$

We also assume that the distribution over all segmentations and all alignments is *uniform*.
This means that there is **no** distortion model or segmentation model.

## Getting Started


In your hw2 github repository, you will find a Python program called
`decode`, which is an implementation of a simple stack decoder.
The decoder translates monotonically &mdash; that is, without reordering the
English phrases &mdash; and by default it also uses very strict pruning limits, 
which you can vary on the command line.

The provided decoder solves the search problem, but makes two assumptions, the first of which
is that

$$
\begin{align*}
  \mathbf{e}^* &= \arg\max_\mathbf{e} \sum_\mathbf{a}
  	          p_{TM}(\mathbf{f},\mathbf{a} \mid \mathbf{e}) \times
  	          p_{LM}(\mathbf{e})\\
  &\approx \arg\max_\mathbf{e} \max_\mathbf{a}
  	   p_{TM}(\mathbf{f},\mathbf{a} \mid \mathbf{e}) \times p_{LM}(\mathbf{e}).
\end{align*}
$$

This approximation means that you can use the dynamic programming Viterbi algorithm to
find the best translation.

The second assumption is that there is no reordering of phrases during translation,
therefore the reordering of source words can only be accomplished if the reordering is
part of a *memorized phrase pair*. If word-for-word translations are all that are available
for some sentence, the translations will always be in the source language order.

In the starter kit there is also the `data` directory which contains a translation model,
a language model, and a set of input sentences to translate. Run the decoder using this 
command:


<tt>decode &gt; output</tt>

This loads the models and decodes the input sentences, storing the
result in <tt>output</tt>. You can see the translations simply by looking
at the file. To calculate their true model score, run the command:


<tt>grade &lt; output</tt>

This command computes the probability of the output sentences
according to the model. It works by summing over all possible ways that
the model could have generated the English from the French. In general this
is intractable, but because the phrase dictionary is fixed and sparse, the
specific instances here can be computed in a few minutes. It is still
easier to do this exactly than it is to find the optimal translation.
In fact, if you look at the grade script you may get some hints about 
how to do the assignment!

Improving the search algorithm in the decoder &mdash; for instance by enabling
it to search over permutations of English phrases &mdash; should permit you 
to find more probable translations
of the input French sentences than the ones found by the baseline system. 

The <tt>grade</tt> program will tell you the probability of your output.

## The Challenge

Your task for this assignment is to <b>find the English sentence
with the highest possible probability</b>.
Formally, this means your goal is to solve the problem:

$$
  \displaystyle \operatorname{\arg\max}\limits_\mathbf{e}
  p(\mathbf{f} \mid \mathbf{e})
  \times p(\mathbf{e}),
$$

where $$f$$ is a
French sentence and $$e$$ is an English sentence. In the model we have 
provided you,

$$
  p(\mathbf{e})
  = p(e_1 \mid START)
  \times \prod_{j=2}^J p(e_j \mid e_{j-1},e_{j-2})
  \times p(END \mid e_J,e_{J-1})
$$

and

$$
  p(\mathbf{f}\mid\mathbf{e})
  = p(\textrm{segmentation})
  \times p(\textrm{reordering})
  \times p(\textrm{phrase translation}).
$$

We will make the simplifying assumption that segmentation and reordering
probabilities are uniform across all sentences, and hence constant. This results
in a model whose probability density function does not sum to one. But from a
practical perspective, it slightly simplifies the implementation without
substantially harming empirical accuracy. This means that you only need
consider the product of the phrase translation probabilities 

$$
  p(\mathbf{f} \mid \mathbf{e})
  = \prod_{\langle i,i',j,j' \rangle \in \mathbf{a}}
  p \left (\mathbf{f}_{\langle i,i' \rangle} \mid \mathbf{e}_{\langle j,j' \rangle} \right )
$$

where $$ \langle i,i' \rangle $$ and $$ \langle j,j' \rangle $$ index phrases
in $$\mathbf{f}$$ and $$\mathbf{e}$$, respectively.

Unfortunately, even with all of these simplifications, finding the most
probable English sentence is completely intractable! To compute it
exactly, for each English sentence you would need to compute
$$ p(\mathbf{f} \mid \mathbf{e}) $$ 
as a sum over all possible alignments with the French sentence:
$$p(\mathbf{f} \mid \mathbf{e}) = \sum_\mathbf{a} p(\mathbf{f},\mathbf{a} \mid \mathbf{e})$$.
A nearly universal approximation is to
instead search for the English string together with a single alignment, 
$$\mathop{\arg\,\max}\limits_{\mathbf{e},\mathbf{a}}~ p(\mathbf{f},\mathbf{a} \mid \mathbf{e})
\times p(\mathbf{e}) $$.
This is the approach taken by the monotone default decoder.

Since this involves multiplying together many small probabilities, it is 
helpful to work in logspace to avoid numerical underflow. We instead solve for
$$\mathbf{e},\mathbf{a}$$ that maximizes:

$$
  \displaystyle
  \log p(\mathbf{f},\mathbf{a} \mid \mathbf{e}) + \log p(\mathbf{e})
  = \log p(e_1 \mid START)
  + \displaystyle \sum_{j=2}^J
    \log p(e_j \mid e_{j-1}e_{j-2})
  + \log p(END \mid e_J) +
  \displaystyle \sum_{\langle i,i',j,j' \rangle \in a}
    \log p(\mathbf{f}_{\langle i,i' \rangle} \mid \mathbf{e}_{\langle j,j' \rangle}).
$$

The default decoder already works with log probabilities, so it is 
not necessary for you to perform any additional conversion; you can simply work
with the sum of the scores that the model provides for you. Note that
since probabilities are always less than or equal to one, their equivalent 
values in logspace will always be negative or zero, respectively (you may
notice that `grade` sometimes reports positive translation model 
scores; this is because the sum it computes does not include 
the large negative constant associated 
with the log probabilities of segmentation and reordering). Hence your
translations will always have negative scores, and you will be looking for
the one with the **smallest absolute value**. In other words, we have
transformed the problem of finding the most probable translation into a 
problem of finding the shortest path through a large graph of possible
outputs.

Under the phrase-based model we've given you, the goal is to 
find a phrase segmentation, translation of each resulting phrase,
and permutation of those phrases such that the product of the phrase
translation probabilities and the language model score of the resulting
sentence is as high as possible. Arbitrary permutations of the English 
phrases are allowed, provided that the phrase translations are one-to-one and
exactly cover the input sentence. Even with all of the simplifications we
have made, this problem is *still*
NP-Complete, so we recommend that you solve it using an approximate
method like stack decoding, discussed in Chapter 6 of the textbook. In fact,
the baseline score you must beat was achieved using **stack decoding with reordering**.
You can trade efficiency for search effectiveness
by implementing histogram pruning or threshold pruning, or by playing around
with reordering limits as described in the textbook. Or, you might
consider implementing other approaches to solving the search problem:

* [Implement a greedy decoder](http://www.iro.umontreal.ca/~felipe/bib2webV0.81/cv/papers/paper-tmi-2007.pdf).
* [Reduce the problem to a traveling salesman problem (TSP) and decode using an off-the-shelf TSP solver](http://aclweb.org/anthology-new/P/P09/P09-1038.pdf).
* [Reformulate the problem as a shortest-path problem through a finite-state lattice and decode using finite-state transducers](http://mi.eng.cam.ac.uk/~wjb31/ppubs/ttmjnle.pdf).
* [Decode using Lagrangian relaxation](http://aclweb.org/anthology-new/D/D11/D11-1003.pdf) (often finds exact solutions!)


Several techniques used for the IBM Models (which have very similar 
search problems as phrase-based models) could also be adapted:

* [Implement A* Search](http://aclweb.org/anthology-new/W/W01/W01-1408.pdf).
* [Implement a bidirectional decoder](http://aclweb.org/anthology-new/C/C02/C02-1050.pdf).
* [Decode using an integer linear programming (ILP) solver](http://aclweb.org/anthology-new/N/N09/N09-2002.pdf).

Also consider marginalizing over the different alignments:

* [Use Monte Carlo techniques](http://www.springerlink.com/content/d55r04j3850h8473).
* [Use variational techniques](http://www.cs.jhu.edu/~zfli/pubs/variational_decoding_zhifei_acl09.pdf).

But the sky's the limit! There are many, many ways to try to solve the decoding
problem, and you can try anything you want as long as you follow the ground rules:

## Ground Rules

* For this assignment you may work independently, or in groups.
  1. Groups must be publicly announced in class
  1. Every member of the group must email the instructor (subject: LING 506 HW2 group) listing all group members in the email
  1. Everyone in the group will receive the same grade on the assignment
  1. Claims regarding who did or did not do what work will be completely ignored
  
* You must turn in three things:
  1. All of your work, including your code and LaTeX writeup, must be checked in
     and pushed to your HW2 github repository.
  1. You must submit your assignment, using the following command:

    /home/lanes/Homeworks/hw2/submit.sh

  1. A clear, mathematical description of your algorithm and its motivation
     written in scientific style. This needn't be long, but it should be
     clear enough that one of your fellow students could re-implement it 
     exactly. We will review examples in class before the due date.
* You do not need any other data than what is provided. You should feel 
   free to use additional codebases and libraries _except for those expressly intended to decode machine translation models_. 
   You must write your
   own decoder. If you would like to base your solution on finite-state
   toolkits or generic solvers for traveling salesman problems or
   integer linear programming, that is fine. 
   But machine translation software including (but not limited to)
   Moses, cdec, Joshua, or phrasal is off-limits. You may of course inspect 
   these systems if you want to understand how they work. But be warned: they are
   generally quite complicated because they provide a great deal of other
   functionality that is not the focus of this assignment.
   It is possible to complete the assignment with a quite modest amount
   of python code.
   If you aren't sure whether 
   something is permitted, ask us.

If you have any questions or you're confused about anything, just ask.
