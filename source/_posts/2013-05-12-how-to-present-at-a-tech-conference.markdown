---
layout: post
title: "How to present at a tech conference"
date: 2013-05-12 21:33
comments: true
categories: 
---

Ohai Chefs!

This week I'll share my tips and tricks for presenting at a tech conference,
based on my vast experience of one conference. So, take it with several grains
of salt.

<!-- more -->

## Chef Conf 2013

I had the opportunity to speak at Chef Conf 2013 on April 26th on [Tips and
Tricks for Automating Windows with Chef][1]. I actually proposed a completely
different talk back in January which when it just wasn't coming together in
early April, I had to abandon.

[1]: http://youtu.be/APBSff1_oVY

## The Proposal

I don't have a lot of advice here, except to share my mistake. In January, I
submitted this proposal:

"Come hear about how Nordstrom developed a Continuous Delivery workflow for Chef
development. Engineers test cookbooks locally using Vagrant, then commit to Git,
which triggers a Jenkins build. Jenkins runs cookbook tests, then uploads
cookbooks to our Private Chef server."

Which would have made a great talk. In January, we hadn't even started on a CD
pipeline for Chef Cookbooks. By April, we hadn't even started on a CD
pipeline for Chef Cookbooks. I finally had to admit, there was no way I could
get up and talk about something we hadn't done yet. I had to email Nathen Harvey
at Opscode and tell him I needed to talk about a different topic. He was great
and I was able to pick a new topic, one I knew I had quite a bit of experience
with.

So my advice is to pick a topic on which you already have experience instead of
something you hope to get to before the confernece.

## The Preparation

Preparing for a talk is a lot of work. I estimate I spent an 60-75 min for every
minute of actual speaking time, roughly 30-35 hours of work for a 25 min talk.
When you know you're going to be speaking in front of very smart engineers, you
really need to know your topic thoroughly and anticipate the kinds of questions
you'll get.

## Tell a Story

Don't just dive into technical details and code samples. Tell a story. We humans
are hard-wired to engage with stories. Share a problem you faced, why it was
challenging, what you tried that didn't work, and how you overcame it. Then dive
into the technical details. Your audience will be ready to hear you talk about
the technical nitty-gritty after you've got their attention with a story.

## Presentation Zen

I have used [Presentation Zen][2] by Garr Reynolds for several presentations and
I can't recommend it enough. You should read this before your next presentation.

[2]: http://www.amazon.com/Presentation-Zen-Simple-Delivery-Edition/dp/0321811984

## Practice

I gave my talk three times before I gave it for real. I presented it to my team
twice and I presented it to the Seattle Chef meetup the week before Chef Conf.
Even though these were unfinished drafts, I got very valuble feedback from a
friendly audience. My talk was much better because of the practice and feedback
I received. It also gave me confidence that my topic was useful and helpful to
people.

Nathen Harvey (Opscode) also scheduled a few Google+ hangouts for conference
speakers which were very helpful. I got a chance to meet some other speakers and
get my questions answered. He also encouraged speakers to send him our slides
before the conference. It was just additional reassurance that I was on the
right track.

## Logistics

And finally, some logistical tips.

1. Follow the code of conduct. This should go without saying, but no
   inappropriate content in your slides. 

2. Ask what resolution you'll be presenting at. I formatted my slides for 1024 x
   768 originally, and then had to change to 1280 x 720. This makes a big
difference if you are using full-bleed photos.

3. Make sure any photos are licensed for reuse. I find nearly all my slide
   photos by doing Creative Commons-licensed searches on Flickr.com.

4. Give proper attribution for photos you use. I usually just paste the link to
   the photo in the bottom right-hand corner of the slide.

5. Have your slides on your laptop __and__ on a USB stick. Even if you are
   supposed to present on your laptop, it may not work and you may have to use a
conference provided laptop.

6. Don't depend on using Keynote Remote from your phone to drive the slides. You
   can almost count on conference wifi or a Bluetooth connection being flaky.
Use a real remote.

7. Show up 15 min before your talk to get setup with the mic, make sure your
   laptop can connect to the projector, etc.

8. Ask for a glass or bottle of water for your talk.

9. Get enough sleep the night before.

10. Bring any video adapters you may need, e.g HDMI, DVI, etc.

11. If you are having one of your teammates join you on stage, ask the
    conference organizers for a mic for them.

12. Repeat the question. People in the audience and people watching your
    presentation later will appreciate you repeating any questions you get
before answering them.

13. Display code on a light background. If you create Github gists for each of your
    code samples, you can copy/paste those into your slides and it will preserve
the formatting.

14. Have a printed copy of your slide notes with you. Your carefully crafted
    prensenter display may not work with the conference projector and you may
end up with just your slides showing on your laptop instead of your presenter
display with slide notes.

## Summary

That's it for this week. I hope you'll get a chance to present at a future
conference if you haven't already. I'd love to hear your presentation tips in
the comments. Cheers.
