# datomic-essentials

A series of Datomic introduction articles, using SQL to explain certain abstract ideas

## Datomic: A Database with Built-in Event Sourcing

Once, I came across a story shared by a teacher who taught DDD (domain-driven design). He mentioned that after conducting a course on event sourcing, some participants told him, “Teacher, this event sourcing thing... it seems hard to apply in practice.”

There are other similar stories. During my time at a particular organization, I asked some colleagues if they had ever implemented event sourcing. One colleague replied, “Yes, but that project ended in disaster. Someone even got so frustrated that he quitted.”

Strangely enough, when I use event sourcing, it works perfectly fine—no difficulties at all.

You might think I’m about to tell you that I’m some kind of 10x engineer. That’s not the case. What I’m really here to share is my secret: I use a database with built-in event sourcing—Datomic.


## Acknowledgments

The completion of this article series is largely inspired by my work at [Gaiwan](https://gaiwan.co/) and [LambdaIsland](https://lambdaisland.com/). Additionally, while attending [Heart of Clojure](https://2024.heartofclojure.eu/), I learned that non-Chinese readers were also interested, so I translated the original [Traditional Chinese version](https://ithelp.ithome.com.tw/users/20161869/ironman/7432) into English.  

If you have any thoughts after reading this series, feel free to reach out to [me](https://replware.dev/).

## License

Copyright @ Laurence Chen

Licensed under the term of [the Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/
).
