---
layout: cv
title: CV
permalink: /cv/
---

<div class="intro-box">
  <div style="width: 68%;">
    {{ site.description }}<br>
    <em>design type systems,&nbsp; build compilers,&nbsp; verify programs</em> <br>
    Email: {{ site.email }}
  </div>
  <div style="width: 30%;">
    {%- include social.html -%}
  </div>
</div>

{% include education.html %}

## Publication

{% include publication.html cv="full" %}

## Research Projects

{% include experience.html tags="research" %}

## Work Experience

{% include experience.html tags="work" %}

## Teaching & Services

{% include experience.html tags="service" %}

## Talks

{% include experience.html tags="talk" %}

## Awards & Honors

{% include awards.html %}

<div markdown="1" style="page-break-inside: avoid;">
## Skills

| Programming Languages | Pascal, C/C++, C#, Python, Java, Scala, TypeScript, F#    |
| Web Development       | HTML/JS/CSS, Vue.js, jQuery, Flask, Django                |
| Performance Tuning    | C, Assembly (x64, aarch64), Intel intrinsics, OpenMP, MPI |
| Compiler Frameworks   | Clang/LLVM, TVM, LMS, Scala                               |
| Theorem Proving       | Rocq, Lean, Dafny, Boogie                                 |
| Natural Languages     | English (fluent), Chinese (native)                        |

</div>

<div markdown="1" style="display: none; page-break-inside: avoid;">
## Travel

- United States, 2021-now
- Japan, 2018

</div>
