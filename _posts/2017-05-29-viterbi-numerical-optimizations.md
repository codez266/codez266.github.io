---
layout: post
title: Viterbi and Numerical Optimizations
permalink: viterbi-num-optimizations
tags:
- research
- optimization
---

I'm quite excited to write about my recent set of investigations on a seemingly
simple mini-project.
Recently I wrote a small code for [Machine Translation](https://github.com/codez266/ai_iitp/tree/master/MachineTranslation) from English to Hindi
using HMM Viterbi decoding. The code used word alignments from the [GIZA++](http://www.statmt.org/moses/giza/GIZA++.html) tool
and after aligning the words in both languages, we get probability scores like:

|Src lang index|Dst lang index|Probability|
|--------------|--------------|-----------|
|0|6|0.0105564|
|0|9|0.0888391| 
|0|11|0.123789| 
|0|12|8.75807e-06| 
|0|14|0.000144811| 
|0|17|3.12898e-05|

The Machine Translation model is analogous to an HMM POS-Tagger where words in one
_language can be treated as hidden states transitioning from one to another( and defined by the language model ) and each word( state ) generating a word in the target language using emission probability given by the alignments above._

## Viterbi Decoding

Once this analogy between POS-Tagging and Machine Translation is setup, I simply
ported the POS-Tagger code for Machine Translation, but there were surprises to
be had. I ran the algorithm and it would take ages to translate one sentence.
Lets see what was wrong:
#### Initialization
```
SEQSCORE(1,1)=1.0
BACKPTR(1,1)=0
For(i=2 to N) do
SEQSCORE(i,1)=0.0
[expressing the fact that first state is S 1 ]
```

#### Iteration
```
For(t=2 to T) do
For(i=1 to N) do
	SEQSCORE(i,t) = Max (j=1,N)
	[ SEQSCORE( j , ( t − 1 )) * P ( Sj --ak--> Si)]
BACKPTR(I,t) = index j that gives the MAX above
```
__Complexity: O(T x n x n)__

POS-Tagger: Hidden states = tags = 20(say), words = 5000, Complexity = 5 x 10^8

Machine Translation: Hidden states = Hindi words = 94125~10^5, English words = 72167~10^5, Complexity ~ 10^10

Clearly, because the number of hidden states in Machine Translation is
equivalent to the number of words, our complexity is of that order of magniture
higher.

## Can we do better?

## First Attempt: _Base approach_

To start with I had a transition matrix that contained transitions of every word
to every other words, which I readily indexed using __'transition[w_idx1]'[w_idx2]__. This was the naivest approach and which did not even meet memory constraints, letting my laptop hang.

To solve this, I decided to use a scipy [sparse matrix](https://docs.scipy.org/doc/scipy/reference/sparse.html), and after some research, given
the way my transition was structured, I decided to go ahead with [csr](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.csr_matrix.html#scipy.sparse.csr_matrix) sparse
matrix

## Attempt _two_

After discussion with one of my friends, it came to my mind that the above
algorithm naively checks transition from every state to every other state,
whereas in a real language _one word leads two only a handful number of other
words in the language. I was unnecessarily iterating over null transitions!_

Lets take a look at the code:
```
# 't' is the current word
for idx1, i in enumerate(transition[0]):
	maxScore = seqscore[0][t]                                       
	for idx2, j in enumerate(transition[0]):                        
		score = seqscore[j][t-1] * transition[idx2][idx1] * emission[wordindex][idx1]                                                               
```

Clearly the transition evaluation needs some love, 
I googled for ways to only get nonzero rows in transition matrix, and found
[find](https://docs.scipy.org/doc/scipy-0.18.1/reference/generated/scipy.sparse.find.html), so lets see:

```
tr_row = transition.getrow(idx1)
.
.
.
rs, indices, vals = find(tr_row)
for idx2, v2 in zip(indices, vals):
```

I did get some nasty code in the inner loop, but thats what was, I had to
tradeoff an easy to understand implementation for speed.

As a result, I did get an instant improvement in speed! The __search__ method
which would take ages for one sentence now took 5 minutes. However I was still
not satisfied, so I decided to dig further, this time using tech ;)

## Attempt _three_

I remembered that python had a profiling library so decided to take up actual [profiling](https://docs.python.org/2/library/profile.html)

Using
```
python -m cProfile [-o output_file] [-s sort_order] myscript.py
```
I could save my stats to a file, which I later retrieved as:
```
import pstats
p = pstats.Stats('stats')
p.sort_stats('cumulative').print_stats(50)
```
The result, after ordering by cumulative time:

```
   Ordered by: cumulative time
   List reduced from 2002 to 50 due to restriction <50>

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
    557/1    0.040    0.000  197.171  197.171 {built-in method builtins.exec}
        1    0.005    0.005  197.171  197.171 mt.py:1(<module>)
        1    0.000    0.000  196.803  196.803 mt.py:122(main)
        1    0.031    0.031  196.803  196.803 mt.py:100(train)
        1   17.758   17.758  182.292  182.292 mt.py:51(search)
   564750    4.789    0.000  112.658    0.000 /usr/lib/python3.6/site-packages/scipy/sparse/extract.py:14(find)
1129511/564761    7.526    0.000   86.061    0.000 /usr/lib/python3.6/site-packages/scipy/sparse/coo.py:118(__init__)
   564757    5.523    0.000   53.782    0.000 /usr/lib/python3.6/site-packages/scipy/sparse/compressed.py:905(tocoo)
  1129511   20.949    0.000   52.660    0.000 /usr/lib/python3.6/site-packages/scipy/sparse/coo.py:212(_check)
   564757    0.943    0.000   51.593    0.000 /usr/lib/python3.6/site-packages/scipy/sparse/csr.py:356(getrow)
   564757    3.779    0.000   50.650    0.000 /usr/lib/python3.6/site-packages/scipy/sparse/csr.py:411(_get_submatrix)
564770/564764    4.156    0.000   40.853    0.000 /usr/lib/python3.6/site-packages/scipy/sparse/compressed.py:24(__init__)

```
From above data, it is clear that 'scipy/sparse/coo.py' was taking the largest
number of individual calls. After some digging, I found that though I'm reducing
the number of transitions by only evaluting non-zero transitions, the method
'get_row' from scipy for getting individual rows of a sparse matrix is in itself quite __intensive__

## Wrap-up

Whoa, so much for a simple Machine Translation task! but it was a good exercise
that me familiar with sparse matrices, actual optimizations in terms of memory
and time at the data structure level and profiling!


