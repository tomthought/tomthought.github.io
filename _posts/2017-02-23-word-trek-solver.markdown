---
layout: post
title:  "Word Trek Solver"
date:   2017-02-23 14:30:57 -0500
categories: clojure
---

I was recently introduced to a mobile (among other platforms) game by
the name of [Word Trek][word-trek]. It is in the same family of games
as Boggle. After playing for a while, I began to realize that the game
was a bit more challenging at times than I thought. I wanted to get
some hints without having to watch ads a couple times an hour for just
a few extra points.

True puzzle enthusiasts will of course consider this blasphemy, but
the result can just be used in cases of extreme frustration :)

But on to the code. The github repository can be
found [here][github-repo].

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
millions of possible paths to take. With some optimization techniques
we can speed up the process a little, but the big win is to reduce the
size of the input by short circuiting paths that we know will never
form words.

I did a bit of Googling, and came across a great [thread][so-thread]
on Stack Overflow discussing this exact problem. A solution using a
[trie][trie-wiki] ended up being the most compelling. I know that
[Lucene][apache-lucene] uses this data structure under the hood in order to achieve
search at the level of performance we've become accustomed to, so I
brought in a Clojure wrapper around [Lucene][apache-lucene], [weavejester/clucy][clucy-github].


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
prefix to any of the words in our dictionary. We can use this function
to short circuit any paths in the matrix that won't lead to any words
being discovered. Our workload sees a massive reduction by following
this simple idea. I have used a depth-first search as my graph-walk
algorithm.

{% highlight clojure %}

(defn- dfs
  "Run a depth-first search against the provided 'matrix', preventing any nodes
  that fail a check against 'pred' from being added to the queue. We are using a
  more imperative approach here in order to avoid as many intermediary
  representations as possible. A volatile for the queue, and a transient for the
  words has been used."
  ([matrix] (dfs (constantly true) matrix))
  ([pred matrix]
   (let [queue (volatile! [])
         words (transient #{})
         rows (count matrix)
         cols (count (first matrix))
         get-letter (fn [i j] (get-in matrix [j i]))]

     ;; seed the queue
     (dotimes [i cols]
       (dotimes [j rows]
         (let [c (get-letter i j)]
           (when (<= 97 (int c) 123)
             (let [node [i j (str c) #{[i j]}]]
               (when (pred node)
                 (vswap! queue conj node)))))))

     ;; run dfs
     (loop []
       (when-let [queue* (not-empty @queue)]
         (let [[i j s path-set] (first queue*)]
           (vswap! queue next)

           ;; for every node adjacent to the current node, inspect the word it
           ;; constructs, and then add it to the queue provided it is
           ;; 1) on the matrix, 2) is a word and 3) passes the 'pred' function
           ;; running against it.
           (doseq [[di dj] [[1 0] [1 -1] [0 -1] [-1 -1]
                            [-1 0] [-1 1] [0 1] [1 1]]]
             (let [i2 (+ i di)
                   j2 (+ j dj)]

               ;; verify the node is on the matrix, and has not yet been seen by
               ;; the path this node has followed.
               (when (and (>= i2 0) (< i2 cols)
                          (>= j2 0) (< j2 cols)
                          (not (path-set [i2 j2])))
                 (let [s2 (str s (get-letter i2 j2))
                       node [i2 j2 s2 (conj path-set [i2 j2])]]

                   ;; when a word is found, add it to the list (a set ensures no
                   ;; duplicate words are added)
                   (when (word? s2)
                     (conj! words s2))

                   ;; when the node passes the 'pred' function test, add it to
                   ;; the queue to be inspected later.
                   (when (pred node)
                     (vswap! queue conj node)))))))
         (recur)))

     ;; return a persistent data structure (i.e. the immutable data structure we
     ;; are used to in Clojure) of the words discovered.
     (persistent! words))))
    
{% endhighlight %}

Finally, tying it altogether, we have the public function from the
namespace, `solve`.

{% highlight clojure %}

(defn solve
  "Given a string of 'letters', and optionally a collection of desired
  'word-sizes', output a collection of words that can be formed by
  assembling the letters into a matrix.

  The length of 'letters' must be a perfect square (i.e. the number of rows must
  be equal to the number of columns.

  'word-sizes' is a collection of integers representing the acceptable word
  lengths in the solution."
  ([letters] (solve letters (range (count letters))))
  ([letters word-sizes]
   (let [trie (load-dictionary-trie)
         len (count letters)
         dim (and (perfect-square? len) (int (Math/sqrt len)))
         valid-size? (set word-sizes)]
     (assert dim "Input string length must be a perfect square.")
     (->> (mapv vec (partition dim (st/lower-case letters)))
          (dfs (dfs-pred word-sizes (load-dictionary-trie)))
          (filter (comp valid-size? count))))))
          
{% endhighlight %}

There are a few helper functions I haven't included in this port but
can be found in the `core` namespace provided in the project. Once
more, the repo can be found [here][github-repo].

Happy Word Trekking! Just remember to solve a few on your own from
time to time.

[word-trek]: https://itunes.apple.com/in/app/word-trek-wordtrek-brain-game/id994855806?mt=8
[en-dict]: http://www-01.sil.org/linguistics/wordlists/english/
[github-repo]: https://github.com/tomthought/word-trek-solver
[so-thread]: http://stackoverflow.com/questions/746082/how-to-find-list-of-possible-words-from-a-letter-matrix-boggle-solver/
[trie-wiki]: https://en.wikipedia.org/wiki/Trie
[clucy-github]: https://github.com/weavejester/clucy
[apache-lucene]: https://lucene.apache.org/
