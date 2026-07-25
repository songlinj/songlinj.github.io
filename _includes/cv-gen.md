{% assign mode = include.mode | default: "academic" %}

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

{% if mode == "industry" %}
## Select Publication

{% include publication.html cv="featured" %}
{% else %}
## Publication

{% include publication.html cv="full" %}
{% endif %}

{% unless mode == "travel" %}
## Research Projects

{% include experience.html tags="research" %}
{% endunless %}

## Work Experience

{% if mode == "industry" %}
{% include experience.html tags="work,work-old" %}
{% else %}
{% include experience.html tags="work" %}
{% endif %}

{% if mode == "academic" %}
## Teaching & Services

{% include experience.html tags="service" %}

## Talks

{% include experience.html tags="talk" %}
{% endif %}

## Awards & Honors

{% if mode == "industry" %}
{% include awards.html tags="competition" %}
{% else %}
{% include awards.html %}
{% endif %}

{% unless mode == "travel" %}
## Skills

| Programming Languages | Pascal, C/C++, C#, Python, Java, Scala, TypeScript, F#    |
| Web Development       | HTML/JS/CSS, Vue.js, jQuery, Flask, Django                |
| Performance Tuning    | C, Assembly (x64, aarch64), Intel intrinsics, OpenMP, MPI |
| Compiler Frameworks   | Clang/LLVM, TVM, LMS, Scala                               |
| Theorem Proving       | Rocq, Lean, Dafny, Boogie                                 |
| Natural Languages     | English (fluent), Chinese (native)                        |
{% endunless %}

{% if mode == "travel" %}
## Travel

- United States, 2021-now
- Japan, 2018
{% endif %}
