---
published: false
---

## Automating Pylint via Jenkins and Gerrit

As many programmers have found, static analysis and sticking to a style guide is incredibly helpful, especially when your team is from diverse code backgrounds. Choosing a static analysis and style guide is the easy part, sticking to it is another story altogether. 

To assist with this, I decided to integrate our Jenkins server with Gerrit to automatically run pylint (our choice for static analysis - not perfect, but it helps) on incoming reviews and post results. There's a handful of guides around the web that will teach you how to setup Jenkins to run a job on gerrit events, but not a lot of information on custom messages back to Gerrit, or automating static analysis. 