# _WACTF2 2018 Misc Challenges Writeups - Shyamal Varsani (**@baconandegg5**)_

## apk-misc-challenge
### Challenge Prompt:
 >Decompile and unpack java from the provided apk file to find the encrypted flag. Look inside /com/example/hax/loginapp/ to obtain the secret files. Use the decoding files to reveal the secret.

 We were provided with a 'decompile_this.apk' - .apk file, which is a file for Android applications. As the challenge was talking about decompiling and unpacking the provided apk, the use of `apktool` - A tool for reverse engineering 3rd party, closed, binary Android apps could be used to decode resources to nearly original form and rebuild them. 

 I used this to essentially strip out the file structure out of the apk, in which I could subsequently traverse to `/com/example/hax/loginapp/` and then hopefully just cat out the 'secret' file that contained the flag.

The following was used to pull out the resources from the decompile_this.apk:

`apktool decompile_this.apk -o apkout`

This essentially outputted all of the data into a directory called apkout/. When traversing into this directory I found that the /smali directory that had been extracted contained a /com directory.

```
root@kali:~/Desktop/Misc-1/apkout# ls
AndroidManifest.xml  apktool.yml  original  res  smali
root@kali:~/Desktop/Misc-1/apkout# cd smali/
root@kali:~/Desktop/Misc-1/apkout/smali# ls
android  androidx  com
```
I then proceed to traverse to `/com/example/hax/loginapp/`:
```
root@kali:~/Desktop/Misc-1/apkout/smali/com/example/hax/loginapp# ls
 BuildConfig.smali      'R$anim.smali'   'R$drawable.smali'      'R$menu.smali'    'R$styleable.smali'
'MainActivity$1.smali'  'R$attr.smali'   'R$id.smali'            'R$mipmap.smali'  'R$style.smali'
'MainActivity$2.smali'  'R$bool.smali'   'R$integer.smali'        Rot13.smali       secret.smali
 MainActivity.smali     'R$color.smali'  'R$interpolator.smali'   R.smali
'R$animator.smali'      'R$dimen.smali'  'R$layout.smali'        'R$string.smali'
```
A secret.smali can be seen at this location, and we can also see Rot13.smali! So putting 2-and-2 together, I understood at this point that i probably had a Rot13'd flag in secret.smali.

When opening up the secret.smali, the following text was in that file:
```
root@kali:~/Desktop/Misc-1/apkout/smali/com/example/hax/loginapp# cat secret.smali 
.class public Lcom/example/hax/loginapp/secret;
.super Ljava/lang/Object;
.source "secret.java"


# direct methods
.method public constructor <init>()V
    .locals 0

    .line 5
    invoke-direct {p0}, Ljava/lang/Object;-><init>()V

    return-void
.end method

.method public static main([Ljava/lang/String;)V
    .locals 2
    .param p0, "args"    # [Ljava/lang/String;

    .line 8
    sget-object v0, Ljava/lang/System;->out:Ljava/io/PrintStream;

    const-string v1, "secret: JNPGS2{IRABZBHF_SVPUH_NVEZNA}"

    invoke-virtual {v0, v1}, Ljava/io/PrintStream;->println(Ljava/lang/String;)V

    .line 9
    return-void
.end method
```

Putting the string of `JNPGS2{IRABZBHF_SVPUH_NVEZNA}` into cyberchef and Rotating by the amount of 13 resulted in the plaintext flag being revealed:

*WACTF2{VENOMOUS_FICHU_AIRMAN}*

---
##  Mutus (Latin)
### Challenge Prompt:
>SquirrelX has been doing some very interesting things in the machine learning blockchain AI space and we would love to get our hands on their technology. They have some high-profile employees. Can you get into their employee portal?

### Write Up:
> This was challenge designed to make use of OSINT and Social Engineering in order to get my way into the login portal. 

http://challenge.capture.tf:5200/ - SquirrelX login portal

The login page of the portal did not contain a whole heap  of information to work off (only fields to enter a email and password),although there was a "reset password" button, which directed us to
http://challenge.capture.tf:5200/reset.html, which contained the following information:

```
SquirrelX Password Reset
To reset your password, please call the service desk on: +61 497 810 276
You will be asked for your full name and date of birth to validate your identity 
```
From this information I knew the main object was to find the names and birthday of SquirellX employees. 

Doing a bit of google-fu, after trying all the usual social media such as Twitter,LinkedIn && Facebook (which didn't bear any usual fruits) lead me a obscure social media website:

https://mastodon.social/@squirrelx - social media for squirrelx? Matches the description of "SquirrelX is a machine learning blockchain AI company!" 

The description on the page info matched the one from the command prompt, so huzzah I thought! 

So in an attempt to socially engineer/convince the person (@sudosammy) who was on the other end of +61 497 810 276, without thinking too much into it and wanting to get simply get the point for the challenge asap, I decided to try with the first lead that I had (contained the Bday in profile description as well as stating she was an employee! (yay right?!)).... which turned out to Ms. Laura Banks (https://mastodon.social/@laurabanks) .... I called up the number and put on my best girly voice, only to get Sam responding with "YOU DONT SOUND LIKE LAURA?". I have never hung up the phone so quickly! (call 1)

After I had gotten outed, I then saw that Mr. Ryan Collier (https://mastodon.social/@ryancollier) also a follower of the @squirrelx page && with all the required employee creds to reset the password!

`
Lead Tech AI Blockchain guy - SquirrelX
Bday: 2 Nov 1988
`

So I went on to call the number once more and stated that I could not login and if i could get my password reset! (End of call 2 now):
*Password* == Sqx1password

 Now that I had a password I still needed a email to go along with that password, so another call ensued, where I claimed to still have difficulties logging in and asked if the person on the phone could they could help me verify if i was typing everything in correctly. 

 IIRC the email that was given/verified with me was "ryan@squirrelx.com", so logging into the portal with the email and the reset password netted the flag upon signin :)

 (Unfortunatly I did not save this flag) 


