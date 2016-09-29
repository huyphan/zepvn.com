---
title: '[Pidgin Plugin] Yahoo Messenger &#8211; Buzz blocker'
author: zepvn
layout: post
categories:
  - linux
  - Programming
tags:
  - block
  - buzz
  - disable
  - pidgin
  - pidgin plugin
  - yahoo
excerpt: A Pidgin plugin to block Yahoo's BUZZ messages.
---
Just a small piece of code to block annoying BUZZ messages from your \*Yahoo\* buddies.

<pre>use Purple;

%PLUGIN_INFO = (perl_api_version =&gt; 2,
	        name             =&gt; "Buzz blocker",
	        version          =&gt; "1.0",
	        summary          =&gt; "Block all BUZZ messages",
	        description      =&gt; "Block all BUZZ messages",
	        author           =&gt; "Huy Phan ",
	        url              =&gt; "",
	        load             =&gt; "plugin_load",
	        unload           =&gt; "plugin_unload");

sub conv_received_msg {
    my ($account, $sender, $message, $conv, $flags, $data) = @_;
    if ($message =~ /$sender has buzzed you!/)
    {
        return 1;
    }
    return 0;
}

sub plugin_init {
    return %PLUGIN_INFO;
}

sub plugin_load {
    my $plugin = shift;

    $conv = Purple::Conversations::get_handle();
    Purple::Signal::connect($conv, "receiving-im-msg", $plugin,
                            &conv_received_msg, "received im message");
    Purple::Debug::info("plugin_load()", "loading Buzz Blocker");
}

sub plugin_unload {
    my $plugin = shift;
    Purple::Debug::info("plugin_unload()", "unloading Buzz Blocker");
}</pre>

Save the code above to a file with extension **.pl** under **~/.purple/plugins/** and restart your pidgin.
