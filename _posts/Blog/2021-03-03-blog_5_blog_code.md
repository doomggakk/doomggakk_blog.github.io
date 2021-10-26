---
layout: single
title: "[Blog] 4. 블로그에서 코드블락의 코드 그대로 보여주기"
categories: [Blog]
author_profile: true
excerpt: 블로그에서 코드블락의 코드를 그대로 보여주는 방법에 대하여 정리한다.
toc: true
toc_sticky: true
---

## 블로그에서 코드블락의 코드 그대로 보여주기
- 블로그 게시물 작성 중 코드를 보여주어야 할 때가 있다.
그럴 경우 markdown 문법을 사용하여 물결3개 안에 코드를 적어 코드를 출력하도록 한다.
- liquid나 html코드, markdown 코드 작성 시 게시물에서 raw code 그대로 출력되지 않는 문제가 발생한다.
- 각 코드에 따라 raw code를 보여주도록 하는 방법이 다르다.

<br>

### 1. liquid
----------------------------
- jekyll에서 중괄호와 퍼센트마크를 사용한 특정 문구를 넣어 원하는 명령을 실행한다.
- liquid 코드를 코드블록안에 넣어서 raw code를 출력하려고 하니 렌더링이 되어서 출력되지 않는 문제가 발생한다.<br>
**cf) 렌더링 : html,css,JavaScript등 개발자가 작성한 문서를 브라우저에서 그래픽 형태로 출력하는 과정을 말한다.**
<br>
- 2가지 방법이 존재<br>
    1.raw - endraw<br>
    2.highlight - endhighlight

- raw - endraw 코드를 양끝에 사용 한 뒤 **pre** 태그와 **code** 태그를 붙여주면 코드가 하이라이트 되어 좀 더 보기 쉬워진다.<br>

ex)<br>
![pre-code 사용](/assets/img/blog/5_blog_code_1.png)

<br>

### 2. html
----------------------------
- html 코드를 그대로 보여주는 방법도 2가지가 있다.<br>
    1.물결 3개쓰고 html : 코드블록선언할 때 html을 적어준다.<br>
    2.highlight 표시할언어 - endhighlight<br>

ex)

![pre-code 사용](/assets/img/blog/5_blog_code_2.png)

<br>

### 3. Markdown
----------------------------
- 위에서 사용한 방법 중 pre, code 문구를 사용하면 된다.

<br>

### 4. 3개 언어 동시 사용
----------------------------
- 위의 3가지 방법을 잘 조합하면 된다.
- pre, code 문구는 html언어
- highlight - endhighlight는 liquid언어<br>
ex)

![pre-code 사용](/assets/img/blog/5_blog_code_3.png)

-> 출력화면
<pre>
<code>
{% highlight html %} 
{% raw %}            
<ul class="taxonomy__index__sidebar">
</ul>

{% if page.sidebar.nav %}
      {% include nav_list nav=page.sidebar.nav %}
{% endif %}

### Markdown TEST

{% endraw %}
{% endhighlight %}
</code>
</pre>

<br>

------------------
**◎ 참고자료**

- [깃 블로그에서 코드블록 만들기 - liquid, html, markdown을 한 번에! - 한헌종의 Git Blog](https://hhj6212.github.io/blog/2020/08/22/Jekyll-highlight-codeblock.html)