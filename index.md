---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---

Songlin Jia is a Ph.D. student since 2021 at Purdue CS. Interested in programming languages, he currently works with [Prof. Tiark Rompf](https://tiarkrompf.github.io/). He has been working on a performant, full-fledged [symbolic execution engine by staging](https://github.com/Generative-Program-Analysis/GenSym). Recently, he is interested in the implementation of algebraic effects.

In 2022, Songlin is a student volunteer in PLDI'22. He is a teaching assistant for [CS565 Programming Languages](https://www.cs.purdue.edu/homes/suresh/565-Fall2022/index.html) in the fall semester.

Songlin got his bachelor's degree at Shanghai Jiao Tong University, where he used to work on scientific computing and program analysis.

## Publication

{% for item in site.data.publication %}
<p>
<b>{{ item.title }}</b><br>
<small><em>{{ item.author }}</em></small><br>
in {{ item.booktitle }}, {{ item.year }}
</p>
{% endfor %}