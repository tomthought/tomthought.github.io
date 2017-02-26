---
layout: post
title:  "Can We Quantify Credibility? - Exploring the Fake News Problem"
date:   2017-02-25 20:37:00 -0500
categories: clojure news
---

A few months ago, the concept of `fake news` made its way into my
daily life. The term has been around for some time, but just recently
has become seemingly inescapable online. It was just before the
Christmas break in 2016 that I decided to think about what could be
done to help solve this issue.

On the surface of the issue, it appears as though creating something
that allows us to say with absolute certainty whether a claim that
someone is making is demonstrably true or false is the holy grail of
fake news solvers. In my opinion, there are two major problems with this 
approach. The first is that I remain unconvinced that being able to
cite facts, automated or not, is going to solve the fundamental
issue we face today, in that there is a lot of contention about the
validity of the various sources for our facts. The second problem is
that building such a solution is unlikely to be possible until we can
create a system that
approaches ["human-level artificial intelligence"][human-ai], as
described in the FAQ section of the [fake news challenge][fnc-url]
(FNC), under the heading "Do you think that automated fact checking is
really possible? If yes, in the next 10 years?"

Of course, this doesn't mean progress can't be made in the meantime.
The folks at the FNC are focusing on a stance detection approach for
now, which is likely to yield great results in the future. In the
short to medium term, it is in the weedy grey area is where practical
solutions are likely to be found. I believe this is where my solution
fits.

In fact, I refuse to even label it a fake news detector. I've gone
with the more diplomatic `Article Credibility Profiler`. Instead of
trying to prove whether the claims in a given piece of text are
demonstrably true or false, I have aimed to build a tool that allows
us to make more robust decisions about the credibility of an article
while browsing the web. There are several signals one might use to
quantify the credibility of an article, and it turns out that the
stories most likely to trick us have a lot of common patterns. In
doing so, I have tried to avoid blaming one group or another for
making up lies, or refusing to believe truths. We can instead create
an unbiased threshold to which all credible sources should be held.

I've used a [list][buzzfeed-list] that [Buzzfeed][buzzfeed-article]
compiled late last year as a baseline test. Only about 45 of the
original 51 stories seem to be accessible still, but this is a good
number for a test. I've uploaded the results of running my automated
solution against the URLs provided in Buzzfeed's
list [here][analysis-csv]. 

Here is a screenshot of the results:

![Analysis Results]({{ site.url }}/assets/20170225/20170225_analysis.png)

I haven't made any attempt to weight any of the signals other than a
flag system. Red flags are almost always indicators that an article
requires further investigation. Orange flags are frequently good
indicators, and yellow flags are usually just hints that an article is
a bit sloppy in its delivery.

It turns out that the three best indicators that a story should be
investigated further are, in no particular order: no about page,
contact page, terms of service, or privacy policy (lack of
authentication); one of a few different signals have shown the source
to be a satire site (a very large chunk of the "fake news" stories
originated on satire sites); and the story originated on a site that
violated one of the first two signals (source detection).

If anyone is trying to solve this or a similar problem, or would like
to get in touch about the solution,
please [send me an e-mail][mailto].

[fnc-url]: http://www.fakenewschallenge.org/
[buzzfeed-list]: https://docs.google.com/spreadsheets/d/1sTkRkHLvZp9XlJOynYMXGslKY9fuB_e-2mrxqgLwvZY/edit#gid=652144590
[buzzfeed-article]: https://www.buzzfeed.com/craigsilverman/top-fake-news-of-2016?utm_term=.tx7pQqPJx#.eavpDPxLG
[human-ai]: https://en.wikipedia.org/wiki/Artificial_general_intelligence
[analysis-csv]: {{ site.url }}/assets/20170225/20170225_analysis.csv
[mailto]: mailto:goldsmith.tee@gmail.com
