---
layout: post
title:  "Word Trek Solver"
date:   2017-02-23 14:30:57 -0500
categories: clojure
---

I was recently introduced to a mobile (among other platforms) game by
the name of [Word Trek][word-trek]. After playing for a while, I began
to realize that the game was a bit more challenging at times than I
thought. I wanted to get some hints without having to watch ads a
couple times an hour for just a few extra points.

True puzzle enthusiasts will of course consider this blasphemy, but
the result can just be used in cases of extreme frustration :)

But on to the code.

The goal was to build a solution that takes a string-representation of
a Word Trek matrix and the lengths of the words being searched for, then
compute a list of all possible solutions to the matrix. A solution is
considered a path through the matrix, where no positions in the matrix
are repeated, and the characters at each node (in order) form a word
from a provided dictionary. I pulled my dictionary from [here][en-dict].

![Example Word Trek Matrix]({{ site.url }}/assets/images/word_trek_grid.jpg)

The matrix above has been plugged into the API, where the resulting
solutions are shown. The quality of the words will depend on the
quality of the dictionary used. 

The github repository can be found [here][github-repo].

{% highlight clojure %}
;; solve for all words in the provided 4x4 matrix that are of length
;; 3, 6 or 7.
word-trek-solver.core> (solve "ezmrnubeslospaon" [3 6 7])
=> ("embolus" "loo" "saloon" "saloons" "sub" "ens" "pal" "plumbs" 
"nos" "lap" "alb" "sum" "sob" "sap" "sol" "lob" "reb" "bus" "eon" 
"rem" "zen" "emu" "usa" "looser" "plumes" "lbs" "number" "spa" 
"boa" "mer" "alp" "sal" "sun" "unloose" "nub" "boo" "nooser" "bon" 
"pas" "bum" "salons" "sue" "plumber" "asp" "son" "slumber" "lumber"
"bun" "nob")
{% endhighlight %}

A naive approach to the problem (and my first attempt), is to generate
all possible paths one might follow on the matrix, and then remove all
paths that aren't words in the dictionary we loaded. The problem we
rapidly run into is that even in a 4x4 matrix, there are usually
millions if not tens of millions of possible paths to take. With some
optimization techniques we can speed up the process a little, but the
big win is to reduce the size of the input by short circuiting
paths that we know will never form words.

I did a bit of Googling, and came across a great [thread][so-thread]
on Stack Overflow discussing this exact problem. A solution using a
[trie][trie-wiki] ended up being the most compelling. I know that
Lucene uses this data structure under the hood in order to achieve
search at the level of performance we've become accustomed to, so I
brought in a Clojure wrapper around [Lucene][apache-lucene]
, [clucy][clucy-github].


{% highlight clojure %}
(defn- trie
  [lst]
  (let [index (clucy/memory-index)]
    (apply clucy/add index (mapv (fn [word] {:v word}) lst))
    index))

(defn- prefix?
  [trie prefix]
  (pos? (:_total-hits (meta (clucy/search trie (str prefix "*") 1)))))
{% endhighlight %}

The above code creates an in-memory [Lucene][apache-lucene] index
using [clucy][clucy-github], and adds a collection of strings to it,
effectively creating our trie. The `prefix?` function accepts a trie
and a string as inputs, and tells us whether the string appears as a
prefix to any of the words in our trie/index.

[word-trek]: https://itunes.apple.com/in/app/word-trek-wordtrek-brain-game/id994855806?mt=8
[en-dict]: http://www-01.sil.org/linguistics/wordlists/english/
[github-repo]: https://github.com/tomthought/word-trek-solver
[so-thread]: http://stackoverflow.com/questions/746082/how-to-find-list-of-possible-words-from-a-letter-matrix-boggle-solver/
[trie-wiki]: https://en.wikipedia.org/wiki/Trie
[clucy-github]: https://github.com/weavejester/clucy
[apache-lucene]: https://lucene.apache.org/
