---
title: Python2/Python3/Ruby/C/Haskell polyglot
author: zepvn
layout: post
categories:
  - ctf
  - Programming
  - Uncategorized
tags:
  - haskell
  - polyglot
  - python
  - ruby
comments: true
excerpt:  Polyglot is a way of programming such that the same source code can run on more than one compilers simultaneously. Sometimes people make it more difficult by requiring the code to behave exactly the same on each programming language.
---
Polyglot is a way of programming such that the same source code can run on more than one compilers simultaneously. Sometimes people make it more difficult by requiring the code to behave exactly the same on each programming language.

[HITCON CTF 2014][1] last weekend has a similar challenge where we needed to write a Python2/Python3/Ruby/C/Haskell polyglot code that does the same thing as bash command `cat flag`.

<!--more-->

I couldn&#8217;t solve the problem during the competition and later on was quite amazed of how simple the solutions were. Below is the solution from the player t0mcr00se:

<div class="code-embed-wrapper">
  <pre class="language-python code-embed-pre line-numbers" ><code class="language-python code-embed-code">b = 3;
a = 2 // 3 &#8211;b; c = &#8220;&#8221;&#8221;/.inspect.length; &lt;&lt;eos
* 2 /* 5
where (//) x y = x + y
(/*) x y = x + y
{-
&#8211; */;
main() { system(&#8220;cat flag&#8221;); }
/*
&#8220;&#8221;&#8221;
import os; os.system(&#8220;cat flag&#8221;)
&#8220;&#8221;&#8221;
eos
system(&#8220;cat flag&#8221;)
__END__
# -}
main = do { a &lt;- readFile &#8220;flag&#8221;; putStr a }
&#8211; */ // &#8220;&#8221;&#8221;
</code></pre>
  
  <div class="code-embed-infos">
    <span class="code-embed-name"></span>
  </div>
</div>

Another one from [Ricky Zhou][2] :

<div class="code-embed-wrapper">
  <pre class="language-python code-embed-pre line-numbers" ><code class="language-python code-embed-code">hash
# pragma = (/*) ; 1 /* a =a; a ## b = b
main = 1 ## readFile(&#8220;flag&#8221;) &gt;&gt;= putStr {&#8211;
#*/&lt;/code&gt;
#if 0
x = &#8216;puts `cat flag`&#8217; and &#8216;__import__(&#8220;os&#8221;).system(&#8220;cat flag&#8221;)&#8217;
eval(x)
#endif
#ifdef __GNUC__
#define True = 1; int main() { system(&#8220;cat flag\n&#8221;); }
True
#endif
#define Y -}&lt;/p&gt;
&lt;p&gt;</code></pre>
  
  <div class="code-embed-infos">
    <span class="code-embed-name"></span>
  </div>
</div>

It&#8217;s fairly easy to modify t0mcr00se&#8217;s code to a &#8220;Hello World&#8221; polyglot just for fun:

<div class="code-embed-wrapper">
  <pre class="language-python code-embed-pre line-numbers" ><code class="language-python code-embed-code">b = 3;
a = 2 // 3 &#8211;b; c = &#8220;&#8221;&#8221;/.inspect.length; &lt;&lt;eos
* 2 /* 5
where (//) x y = x + y
(/*) x y = x + y
{-
&#8211; */;
main() { printf(&#8220;Hello World!\n&#8221;); }
/*
&#8220;&#8221;&#8221;
print &#8216;Hello World!&#8217;
&#8220;&#8221;&#8221;
eos
print &#8220;Hello World!\n&#8221;
__END__
# -}
main = do {  putStr &#8220;Hello World!\n&#8221; }
&#8211; */ // &#8220;&#8221;&#8221;&lt;/p&gt;
&lt;p&gt;</code></pre>
  
  <div class="code-embed-infos">
    <span class="code-embed-name"></span>
  </div>
</div>

 [1]: http://hitcon.org/2014/CTF/ "HITCON CTF 2014"
 [2]: https://rzhou.org/~ricky/hitcon2014/polyglot/ "Ricky Zhou"
