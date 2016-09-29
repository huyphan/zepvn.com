---
title: Defcon CTF Quals 2014 – 100lines
author: zepvn
layout: post
categories:
  - ctf
  - security
tags:
  - ctf
  - defcon
  - reversing 
excerpt: A reversing challenge from Defcon CTF Quals 2014. The task is to simplify a lot of calculations in the binary so that it won't exceed memory usage and then reverse an encryption function to get the flag from some random bytes returned by server.
---
Challenge description:

> It&#8217;s not broken, you just need more RAM.
> 
> http://services.2014.shallweplayaga.me/100lines_53ac15fc7aa93da92629d37a669e106c
> 
> 100lines_53ac15fc7aa93da92629d37a669e106c.2014.shallweplayaga.me:20689 

This is the second 64-bit binary file that I deal with in this CTF. It has 4 main functions called `calc`, `loop`, `getByte`, and of course `main`.

<!--more-->

A quick pseudo-code to describe what `main` does:

    Generate 38 number (32-bit each) as OTP, store to (long long) OTP[38]
    Calculate (long long) seed variable based on a constant (named __randpad_len in the binary)
    Generate a (char*) static_buffer based on static buffer (named __randpad in the binary) 
    Print all generated numbers to STDOUT.
    Read 8 chars from STDIN.
    set counter = 0
    For each input char:
        Do a check with very long condition which involves a lot of calls to getByte() function.
        If the check returns True: 
           counter += 1
    if counter > 6:
        Read file 'flag' and store to (char*) flag.
        Loop i from 0 to 37:
            (char) c = getByte(OTP[i], seed, static_buffer) ^ flag[i]
            Princ c to STDOUT in hex format. 
    

The condition that is used to check looks like this:

    if ((input_string[i] == (LODWORD(LODWORD(LOBYTE(LODWORD(LODWORD(getByte(OTP[i], seed, static_buffer)) - LODWORD(LODWORD(LOBYTE(LODWORD(LOWORD(LODWORD(LODWORD(LODWORD(LODWORD(LOBYTE(LODWORD(getByte(OTP[i], seed, static_buffer))) & 0xff) + LODWORD(LOBYTE(LODWORD(getByte(OTP[i], seed, static_buffer))) & 0xff)) + LODWORD(LOBYTE(LODWORD(getByte(OTP[i], seed, static_buffer))) & 0xff)) << 0x5) + LODWORD(LOBYTE(LODWORD(getByte(OTP[i], seed, static_buffer))) & 0xff)) >> 0x8) + LODWORD(LOBYTE(LODWORD(getByte(OTP[i], seed, static_buffer)) - LODWORD(LOWORD(LODWORD(LODWORD(LODWORD(LODWORD(LOBYTE(LODWORD(getByte(OTP[i], seed, static_buffer))) & 0xff) + LODWORD(LOBYTE(LODWORD(getByte(OTP[i], seed, static_buffer))) & 0xff)) + LODWORD(LOBYTE(LODWORD(getByte(OTP[i], seed, static_buffer))) & 0xff)) << 0x5) + LODWORD(LOBYTE(LODWORD(getByte(OTP[i], seed, static_buffer))) & 0xff)) >> 0x8)) >> 0x1)) >> 0x6) * LODWORD(0x5d)))) & 0xff) + 0x20) ? 0xff : 0x0) & LOBYTE(LODWORD(var_11 & 0xff) == LODWORD(LODWORD(LOBYTE(LODWORD(LODWORD(getByte(OTP[i], seed, static_buffer)) - LODWORD(LODWORD(LOBYTE(LODWORD(LOWORD(LODWORD(LODWORD(LODWORD(LODWORD(LOBYTE(LODWORD(getByte(OTP[i], seed, static_buffer))) & 0xff) + LODWORD(LOBYTE(LODWORD(getByte(OTP[i], seed, static_buffer))) & 0xff)) + LODWORD(LOBYTE(LODWORD(getByte(OTP[i], seed, static_buffer))) & 0xff)) << 0x5) + LODWORD(LOBYTE(LODWORD(getByte(OTP[i], seed, static_buffer))) & 0xff)) >> 0x8) + LODWORD(LOBYTE(LODWORD(getByte(OTP[i], seed, static_buffer)) - LODWORD(LOWORD(LODWORD(LODWORD(LODWORD(LODWORD(LOBYTE(LODWORD(getByte(OTP[i], seed, static_buffer))) & 0xff) + LODWORD(LOBYTE(LODWORD(getByte(OTP[i], seed, static_buffer))) & 0xff)) + LODWORD(LOBYTE(LODWORD(getByte(OTP[i], seed, static_buffer))) & 0xff)) << 0x5) + LODWORD(LOBYTE(LODWORD(getByte(OTP[i], seed, static_buffer))) & 0xff)) >> 0x8)) >> 0x1)) >> 0x6) * LODWORD(0x5d)))) & 0xff) + 0x20)))
    

That looks pretty scary to me. However if we look at it closely, the check is basically a comparison between our input character and an expression of `OTP`, `seed`, and `static_buffer` which are all known. So by evaluating that expression with the first 8 numbers from `OTP` we could get the expected input characters.

But it&#8217;s actually impossible to do so with the current code base as it always end up calling malloc with a very huge number in `getByte` function:

<div class="code-embed-wrapper">
  <pre class="language-c code-embed-pre line-numbers" ><code class="language-c code-embed-code">
char getByte(unsigned long long index, unsigned long long seed, unsigned char* static_buffer) {
    int size = (seed &#8211; 0&#215;20) * ((seed &lt;&lt; 2) &#8211; 80);
    (char*) new_buffer = malloc(size);
    if (new_buffer == null) {
        exit(0);
    } else {
        loop(seed, static_buffer, new_buffer)
        free(new_buffer);
        return new_buffer[index];
    }
}
</code></pre>
  
  <div class="code-embed-infos">
    <span class="code-embed-name"></span>
  </div>
</div>

Notice that the value of `seed` is `0xf81000` that makes `size` equal to `1,057,159,935,887,872`, that means it&#8217;s trying to malloc about 961 Terabytes here! So we definitely need to rewrite this function. We can see that:

  * `loop` function does something to assign values to `new_buffer`
  * `getByte` function only cares about the value of `new_buffer[index]`

Let&#8217;s reverse the `loop` function:

<div class="code-embed-wrapper">
  <pre class="language-c code-embed-pre line-numbers" ><code class="language-c code-embed-code">
function loop(unsigned long long seed, unsigned char* static_buffer, unsigned char* new_buffer) {
    int N = seed &#8211; 0&#215;20;
    var_40 = 0&#215;0;
    rax = var_40;
    for (int i=0; i&lt;N; i++) {
        int K = 0;
        for (int j=0; j&lt;=3; j++) {
            K = calc(K, static_buffer, i, j);
        }
        for (int j=0; j&lt;N; j++) {
            int H = 0;
            for (int k=0; k&lt;=3; k++) {
                H = calc(H, static_buffer, j, k);
            }
            H = H ^ K
            for (int k=0; k&lt;=3; k++) {
                new_buffer&#091;(i + j * N) * 4 + k&#093; = H &gt;&gt; ((-k &lt;&lt; 3) + 0&#215;18);
            }
        }
    }
}
</code></pre>
  
  <div class="code-embed-infos">
    <span class="code-embed-name"></span>
  </div>
</div>

The value of each byte in `new_buffer` is independent to each other and it rather depends on `i`,`j`,`k`,`N`, and `H`. We could easily calculate those variable then rewrite `loop` function as below:  


<div class="code-embed-wrapper">
  <pre class="language-c code-embed-pre line-numbers" ><code class="language-c code-embed-code">
function new_loop(unsigned long long seed, unsigned char* static_buffer, unsigned long long my_index) {
    unsigned long long N = seed &#8211; 0&#215;20;
    unsigned long long k = my_index % 4;
    unsigned long long i = (my_index / 4) / N;
    unsigned long long j = (my_index / 4) &#8211; k*N;
    int K = 0;
    for (int l=0; l&lt;=3; l++) {
        K = calc(K, static_buffer, i, l);
    }
    int H = 0;
    for (int l=0; l&lt;=3; l++) {
        H = calc(H, static_buffer, j, l);
    }
    return (H ^ K) &gt;&gt; ((-k &lt;&lt; 3) + 0&#215;18);
}
</code></pre>
  
  <div class="code-embed-infos">
    <span class="code-embed-name"></span>
  </div>
</div>

I remove the `new_buffer` param as we no longer need it, the param `my_index` is added so that it only calculates and returns the value of that index.

`getByte` function also needs to be rewritten now:

<div class="code-embed-wrapper">
  <pre class="language-c code-embed-pre line-numbers" ><code class="language-c code-embed-code">
char getByte(unsigned long long index, unsigned long long seed, unsigned char* static_buffer) {
    return new_loop(seed, static_buffer, my_index)
}
</code></pre>
  
  <div class="code-embed-infos">
    <span class="code-embed-name"></span>
  </div>
</div>

Simple huh ? How about the `calc` function ? Well, Hopper does a pretty good job on decompiling it so I would keep it as is:  


<div class="code-embed-wrapper">
  <pre class="language-c code-embed-pre line-numbers" ><code class="language-c code-embed-code">
function calc(unsigned int, unsigned char*, unsigned long long, unsigned long long) {
    var_m4 = LODWORD(rdi);
    var_m16 = rsi;
    var_m24 = rdx;
    var_m32 = rcx;
    var_m4 = var_m4 | LODWORD(LODWORD(LOBYTE(LODWORD(SAR(LODWORD(LOBYTE(*(int8_t *)(var_m16 + 0&#215;1 + var_m32 + (var_m24 &gt;&gt; 0&#215;3)) & 0xff) & 0xff), LOBYTE(LODWORD(LODWORD(0&#215;8) &#8211; LODWORD(LODWORD(var_m24) & 0&#215;7))))) | LODWORD(LODWORD(LOBYTE(*(int8_t *)(var_m16 + (var_m24 &gt;&gt; 0&#215;3) + var_m32) & 0xff) & 0xff) &lt;&lt; LOBYTE(LODWORD(LODWORD(var_m24) & 0&#215;7)))) & 0xff) &lt;&lt; LOBYTE(LODWORD(LODWORD(LODWORD(LODWORD(0&#215;0) &#8211; LODWORD(var_m32)) &lt;&lt; 0&#215;3) + 0&#215;18)));
    LODWORD(rax) = var_m4;
    return rax;
}
</code></pre>
  
  <div class="code-embed-infos">
    <span class="code-embed-name"></span>
  </div>
</div>

So I got the final code to receive 38 OPT numbers from STDIN and print out ASCII values of expected input characters: [defcon-2014-quals-100lines-bruteforce][1]. In this version, I was too lazy to break down the expression when it checks the input character, so I kept the condition as it is and made a variable looping from 0 -> 255 to test whether that condition is met (ie. the expression returns True)

Simply compile the code by

    $ g++ -o bruteforce bruteforce.c
    

Since I don&#8217;t feel like writing socket code in C, let&#8217;s write some Python script to communicate with the real service and execute the `bruteforce` code that we just got.

<div class="code-embed-wrapper">
  <pre class="language-c code-embed-pre line-numbers" ><code class="language-c code-embed-code">
import socket
from subprocess import Popen, PIPE, STDOUT
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((&#8217;100lines_53ac15fc7aa93da92629d37a669e106c.2014.shallweplayaga.me&#8217;, 20689))
data = s.recv(2048)
data += s.recv(2048)
data = data.split(&#8220;\n&#8221;)[1]
print &#8220;OTP:&#8221;
print data
arr = [str(eval(_)) for _ in data.split()]
p = Popen([&#039;./bruteforce&#039;], stdout=PIPE, stdin=PIPE, stderr=PIPE)
stdout_data = p.communicate(input=&#8221;\n&#8221;.join(arr))[0]
result = &#8220;&#8221;.join(chr(int(_)) for _ in stdout_data.split(&#8220;\n&#8221;)[:-1])
print &#8220;Sending back: &#8220;,repr(result)
s.send(result)
print &#8220;Response:&#8221;
print s.recv(2048)
print s.recv(2048)
</code></pre>
  
  <div class="code-embed-infos">
    <span class="code-embed-name"></span>
  </div>
</div>

One of the sample output:

    OTP:
    0x0003477c0b80aa0f 0x00008985a385bd8e 0x000371f572962c37 0x0001bec6d1d1f7a6 0x000060f32eb8e2d1 0x00017c9388cc842b 0x00032d90a11d5d21 0x000251507644d55f 0x00017c68d0574f1d 0x0000b8e89a3578ac 0x00000ca6feed8082 0x00019427cd14b5cc 0x0001a1a87d1bb359 0x00014faacfe5a0c8 0x0002c42a8dbeb737 0x00016f216470ceb3 0x000245315dfb25a4 0x0000efb06475b013 0x00025d89b8e236e3 0x0001f7662e82c9b5 0x00031ff7762b2aff 0x0003204571d5ab11 0x000223c745c63639 0x00029662377d9a29 0x00019ad590c4492a 0x00036a5b6796f49b 0x0000763f9c93ab15 0x00018d6630f23f04 0x0001e2a3b45862bc 0x00022308c7c061ae 0x00020bb0bf60bd0b 0x0000576670a02ce7 0x00019ba6d2a8b8ba 0x0002bf9e9862b54b 0x0000e6995ea0b0c0 0x0001e2c33725b271 0x00033a65d06136d0 0x0001505234d0f248
    Sending back:  'U<{v!&Vm'
    Response:
    
    0xc6,
    0x11,0x3e,0x93,0x67,0x6a,0xf2,0xcd,0xfe,0x29,0x0d,0x4d,0xf2,0x8a,0x87,0x48,0x2e,0x81,0x39,0xb7,0x20,0x88,0xc3,0x98,0x21,0x20,0xfb,0x51,0xdc,0xb4,0x2a,0x03,0x7f,0xb7,0x79,0xe6,0x34,0x25
    

Okay, we are really close now. We got back 38 bytes and that must be some kind of encrypted form of the flag (or some text that contains flag). We continue reversing the last part of `main` function and get this:

  
&#8230;  
for (int i = 0; i<38; i++) { int c = flag[i] ^ getByte(OTP[i], seed, static_buffer); printf("0x%02x", c); } ... [/pastacode] It's pretty obvious now. I wrote another C code which is quite similar to the `bruteforce` one except that OTP is now hardcoded together with the 38 encrypted bytes: [defcon-2014-quals-100lines-bruteforce.cpp][2].

Compile, run it and enjoy the flag:

    $ g++ -o get_flag get_flag.cpp
    $ ./get_flag
    The flag is:#RadicalSpaceOptimization!

 [1]: https://gist.github.com/huyphan/1b089a7819f3a8dd54ca
 [2]: https://gist.github.com/huyphan/959e58a3baa63368d310
