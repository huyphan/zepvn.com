---
title: A little fun with Commonwealth Bank's Android app
author: zepvn
layout: post
categories:
  - Uncategorized
tags:
  - android
comments: true
excerpt:  
---
So I relocated to Australia recently and one of the first things I've done was openning a bank account at Commonwealth, the largest bank in Australia at the moment. They've got a pretty decent mobile app on iOS, however their Android version has a bug that makes it totally useless on my Blackberry. Yes, you read it correctly. I'm using Blackberry OS 10 which supports Android application natively. 

The issue happens right at the login view where users are required to input their client ID and password before hitting the 'Next' button to jump to the next step. This is how the layout looked like on my Nexus 4:

![Commbank on Nexus 4]({{ site.url }}/uploads/2015/netbank - nexus.png)

Funnily with the same apk file installed on my Blackberry, the 'Next' button just disappeared. As I mentioned earlier, this makes the app totally unusable:

![Commbank on BB]({{ site.url }}/uploads/2015/netbank - blackberry.png)

Believing that I can do something to fix this myself, I decided to give it a try and patch this app. All the fun started with pulling out the apk file from my Nexus device:

    $ adb pull /data/app/com.commbank.netbank-1/base.apk

Then used apktool to decode it: 

    $ apktool d base.apk
    I: Using Apktool 2.0.2 on base.apk
    I: Loading resource table...
    I: Decoding AndroidManifest.xml with resources...
    I: Loading resource table from file: /Users/x/Library/apktool/framework/1.apk
    I: Regular manifest package...
    I: Decoding file-resources...
    I: Decoding values */* XMLs...
    I: Baksmaling classes.dex...
    I: Baksmaling classes2.dex...
    I: Copying assets and libs...
    I: Copying unknown files...
    I: Copying original files...

After looking around, I found the faulty registration layout at `res/layout/fragment_register_1.xml`, the layout is pretty simple with only 23 lines:
{% highlight xml %}
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout android:orientation="vertical" android:layout_width="fill_parent" android:layout_height="fill_parent"
      xmlns:android="http://schemas.android.com/apk/res/android">
        <ProgressBar android:id="@id/registration_progress_bar" android:layout_width="fill_parent" android:layout_height="8.0dip" android:max="5" android:progress="1" android:progressDrawable="@drawable/progress_bar" style="@android:style/Widget.ProgressBar.Horizontal" />
        <LinearLayout android:orientation="vertical" android:paddingLeft="@dimen/registration_margin_left_right" android:paddingTop="@dimen/registration_screen_margin_top" android:paddingRight="@dimen/registration_margin_left_right" android:paddingBottom="40.0dip" android:layout_width="fill_parent" android:layout_height="0.0dip" android:layout_weight="1.0">
            <TextView android:id="@id/introduction" android:layout_width="fill_parent" android:layout_height="wrap_content" android:layout_marginBottom="@dimen/registration_margin_bottom_2" android:text="@string/registration_page_1_introduction" style="@style/RegistrationHeaderTextView" />
            <LinearLayout android:orientation="vertical" android:layout_width="fill_parent" android:layout_height="wrap_content">
                <LinearLayout android:orientation="horizontal" android:layout_width="fill_parent" android:layout_height="wrap_content">
                    <ImageView android:layout_width="24.0dip" android:layout_height="24.0dip" android:layout_margin="8.0dip" android:src="@drawable/ic_account" />
                    <EditText android:textColorHint="@color/hint_text_color" android:id="@id/client_number" android:layout_width="fill_parent" android:layout_height="wrap_content" android:layout_marginRight="10.0dip" android:hint="@string/registration_page_1_client_number" android:digits="0123456789" android:inputType="number" android:imeOptions="actionNext" android:filterTouchesWhenObscured="false" />
                </LinearLayout>
                <LinearLayout android:orientation="horizontal" android:layout_width="fill_parent" android:layout_height="wrap_content">
                    <ImageView android:layout_width="24.0dip" android:layout_height="24.0dip" android:layout_margin="8.0dip" android:src="@drawable/ic_password" />
                    <EditText android:textColorHint="@color/hint_text_color" android:id="@id/password" android:layout_width="fill_parent" android:layout_height="wrap_content" android:layout_marginRight="10.0dip" android:layout_marginBottom="@dimen/body_text_margin" android:hint="@string/registration_page_1_password" android:inputType="textPassword" android:imeOptions="actionDone" android:fontFamily="sans-serif" />
                </LinearLayout>
            </LinearLayout>
            <LinearLayout android:orientation="vertical" android:layout_width="fill_parent" android:layout_height="fill_parent" android:layout_marginTop="40.0dip" android:layout_marginBottom="@dimen/registration_margin_bottom_3">
                <TextView android:textColorLink="@color/cba_black_100" android:id="@id/forgotten_details" android:background="@drawable/link_button_background" android:layout_width="fill_parent" android:layout_height="30.0dip" android:layout_marginTop="5.0dip" android:text="@string/registration_page_1_forgotten_details" style="@style/RegistrationBodyTextView" />
                <TextView android:textColorLink="@color/cba_black_100" android:id="@id/netcode_token" android:background="@drawable/link_button_background" android:layout_width="fill_parent" android:layout_height="30.0dip" android:layout_marginBottom="10.0dip" android:text="@string/registration_page_1_netcode_taken" style="@style/RegistrationBodyTextView" />
            </LinearLayout>
            <Button android:id="@id/btn_continue" android:layout_width="fill_parent" android:layout_height="wrap_content" android:text="@string/btn_next" />
        </LinearLayout>
    </LinearLayout>
{% endhighlight %}

Apparently the designer of this layout wanted to force the container of the text box to take up as much space as is available within the layout element it's been placed in. This leads to no more space for the "Next" button to show up. This bug is not obvious to most developers because most of the modern Android devices are smart enough to fix the layout problem themselves. However turned out that my Blackberry10's "Android runtime" component is not that smart. 

I made a quick fix to that xml file to set the attribute android:layout\_height of the textboxes' container to "wrap\_content" instead of "fill\_parent". Then rebuilt the apk file:

    $ apktool b base
    I: Using Apktool 2.0.2
    I: Checking whether sources has changed...
    I: Smaling smali folder into classes.dex...
    I: Checking whether sources has changed...
    I: Smaling smali_classes2 folder into classes2.dex...
    I: Checking whether resources has changed...
    I: Building resources...
    I: Copying libs... (/lib)
    I: Building apk file...
    I: Copying unknown files/dir...

The new apk file is created at base/dist/base.apk but it would be rejected by the installer because file was still protected from being modifed by a certificate. So the last step is to re-sign the resulting apk file using one of my random keystore:

    $ jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore ~/dev/test.keystore base/dist/base.apk test

Now copy the apk file to my blackberry and install it using [Snap](http://redlightoflove.com/snap/).

Although this is not a complicated hack, I really hope Commonwealth bank could fix their Android app soon otherwise I would need to re-do all those things everytime they release a new version. 

