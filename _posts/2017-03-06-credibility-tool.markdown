---
layout: post
title:  "Introducing Browser Buddy - Exploring the Fake News Problem #2"
date:   2017-03-06 1:17:00 -0500
categories: clojure fakenews
---

In [my last post][acp-post1], I described an app that was able to
look for problem areas, or at the very least eyebrow-raisers, in
the news articles we consume every day online. Having such a tool at
our disposal while we browse the Internet could become incredibly
useful. It would keep us honest, informed and at the very least remove
some of the fear of sharing something with our friends and family that
had no obvious signs of being anything but an interesting story.

The technical term I've come up with for my app is an `Article
Credibility Profiler` (important to note that it is _not_ a fact
checker), but something along the lines of a `Browser Buddy`, or a
`Browsing Guide` might make more sense. As we surf the web (do people
say that still?), we can be given the passive ability to discover
interesting and possibly deceiving bits of information about the
articles we read, and the stories we share.

We can think of this app as our tour guide. While the main attractions
hold our attention, our guide can provide background information on
what we're seeing that would be difficult to discover otherwise.

And with that, I am excited to introduce the first working version of
the `Browser Buddy` below. Give it a try, and let me know if/when you
discover things that do or don't work very well, and if you have any
suggestions on how it might be used or modified going forward. I have
used the same scoring system as before (red flags, orange flags, and
yellow flags, in order from most to least severe).

All feedback is welcome by [e-mail][mailto].

Sample articles:

[Ottawa Citizen - Snowplow Operator][sample1]

[Reuters - Nutella Insight][sample2]

[Empire Herald - Clinton Campaign Paid Beyonce and Jay Z][sample3]

[CBC - Kevin O'Leary][sample4]

[Mercola - Capsaicin Chili Peppers][sample5]

[Washington Post - Interview With Snopes Editor][sample6]

[Infowars - Pivotal Week in American History][sample7]

[Tomthought - Ramblings of a Computer Scientist][sample8]

[The Onion - 2 Year Old Unaware][sample9]

{% include acp_v1.html %}

[acp-post1]: https://tomthought.github.io/clojure/news/2017/02/26/article-credibility-profiler.html
[mailto]: mailto:goldsmith.tee@gmail.com

[sample1]: http://ottawacitizen.com/news/local-news/snowplow-operator-charged-with-impaired-driving-after-collision-with-hospital
[sample2]: http://www.reuters.com/article/us-italy-ferrero-nutella-insight-idUSKBN14V0MK
[sample3]: https://empireherald.com/clinton-campaign-paid-beyonce-and-jay-z-62-million-for-cleveland-concert-to-secure-black-votes/
[sample4]: http://www.cbc.ca/news/politics/kevin-oleary-conservative-leadership-race-1.3939876
[sample5]: http://articles.mercola.com/sites/articles/archive/2017/01/16/can-capsaicin-chili-peppers-beat-cancer.aspx
[sample6]: https://www.washingtonpost.com/news/fact-checker/wp/2015/12/17/an-interview-with-the-editor-of-snopes-technology-changes-but-human-nature-doesnt/
[sample7]: http://www.infowars.com/welcome-to-one-of-the-most-pivotal-weeks-in-modern-american-history/
[sample8]: https://tomthought.github.io/clojure/news/2017/02/26/article-credibility-profiler.html
[sample9]: http://www.theonion.com/article/2-year-old-unaware-hes-basis-6-couples-decisions-n-55166
