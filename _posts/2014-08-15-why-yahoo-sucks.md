---
layout: post
title: Why Yahoo sucks
published: false
---

Recently, I've tried to create a Flickr account, and they require me to create a Yahoo account. At the Yahoo login page, I was surprised to see I could actually login into Yahoo with my Google account. So I gave it a try and, as expected, my e-mail wasn't recognized, but it's all good, I just filled all the information to sign up, and everything was alright until I was redirected to a Flickr page saying I could only share photos if I was 13 or older. Then I realized: I got distracted and entered 2014 as my birthyear.

<!-- more -->

Ok, so I figured I could just head over to yahoo.com, login and change this info.

This was actually much tougher than it sounds.

After logging into yahoo.com, I clicked the "Hi, Rodolfo" link at the top, as to edit my personal info to that account. Then I was kindly obligated to create a Yahoo "Whatever" Profile, so I can edit the very info I typed just two minutes ago.

This is regularly done as if there were not enough social networks out there. Who do you know uses Yahoo! Profiles in an regular basis?

[![Yahoo! Profile](/images/profile.png)](/images/profile.png)

So I just accepted to create the thing and searched again for what I wanted, to edit my info. My thoughts were "why do you make me look for something twice?"

[![Edit profile](/images/edit.png)](/images/edit.png)

And to my surprise, all I see in the Date of birth field is blank space.

[![Date of birth](/images/dob.png)](/images/dob.png)

Fired up DevTools and saw that there were actually three hidden fields for each date component.

[![Chrome DevTools](/images/devtools.png)](/images/devtools.png)

So I just edited the fields manually (entering numbers for each input value corresponding to my birthdate), and clicked Save. I'm glad it worked, or else I would just recommend against using Flickr or anything that is Yahoo's and write this blog post, but I'm just doing the latter.

But I wonder, I just got away with this because I'm a web developer and I know about DevTools and HTML. What about other users who are not obligated to know anything about those things?

Bad, bad service, Yahoo! No donut for you.
