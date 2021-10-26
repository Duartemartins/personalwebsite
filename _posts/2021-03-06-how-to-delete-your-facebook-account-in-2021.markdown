---
layout: post
title: How to delete your Facebook account in 2021
description: Instructions and tips to effectively delete your Facebook account in 2021. Download tagged photos, delete posts, and more. Make time to focus on high value activities.
date: 2021-03-06 16:57:38 +0000
published: true
categories: social-media
tags: tutorial
---
There are several reasons to consider deleting Facebook, a couple good ones being user privacy and psychological impact. As a company, Facebook has been fairly hostile to user privacy concerns, even when controlling for the fact that as a major platform it attracts more scrutiny than the vast majority of other software companies. As a platform it was designed to work a bit like a slot machine, in that it promotes addictive behaviours and keeps you from engaging in deep work. 

A few things might keep you from wanting to fully leave, however if you're reading this, in all likelihood you don't use Facebook to the same extent you once did. Many people have found that they don't feel like they're missing out on anything of value after leaving the platform, and I include myself among those that have enjoyed adopting a more minimalist approach to their digital lives. 

A relevant concept here is Dunbar's number - a suggested cognitive limit to the number of people with whom one can maintain stable social relationships - named after British anthropologist Robin Dunbar. That number is 150, however it's worth bearing in mind that is the upper limit. In reality, most of us never come close to maintaining 150 close relationships at any one time.

Should you want to go ahead with deleting your account, there are a few things worth considering. One of Facebook's main uses is as a reminder for your friends' birthdays, so we want to make sure we export its calendar. Another thing we might want to do is a bit of privacy-focused prevention: delete our posts, leave all groups, download and delete our photos, and unlike all of our liked pages. This is in case Facebook's ghost profiles turn out to be a real practice, and the evidence seems to suggest they are. Ghost profiles are accounts that Facebook creates, with information it contains on people that aren't signed up to the platform. Finally, we might want to let our contacts know that we're leaving the platform and ask them to not upload any photos or information about us, should privacy be a concern.

1. Download all photos
2. Download Facebook's calendar
3. Unfriend, unlike, and delete all photos
4. Announce departure

---

1: Download all photos
--

It's fairly easy to export all of the photos you uploaded by simply using using Facebook's [download your information tool](https://www.facebook.com/settings?tab=your_facebook_information). It's less easy to download photos you've been tagged in. For this you'll have to use some external tools, and most likely have to make use of a little programming.

The first and most simple option is using one of these IFTT recipes:
- [FB to iOS](https://ifttt.com/applets/126727p-back-up-photos-you-re-tagged-in-on-facebook-to-an-ios-photos-album)
- [FB to Dropbox](https://ifttt.com/applets/47704776d-save-photos-you-re-tagged-in-on-facebook-to-a-dropbox-folder)

If for whatever reason you want to use another alternative you can follow [this tutorial](https://matthew-johnston.com/DownloadAllFacebookTaggedPhotos/). Essentially go to your photos page at [facebook.com/me/photos](https://facebook.com/me/photos) and  run this JS script in your browser console to get the "FBIDs" of your photos:
{% highlight javascript%}
for (link of document.getElementsByTagName('a')) { 
    if (!link.href.includes("?fbid=")) continue; 
    console.log(new URL(link.href).searchParams.get("fbid")); 
    }
{% endhighlight%}
Then feed that list to [this python script](https://github.com/mgjohnston/fmpd/tree/patch-1) which downloads your photos.

I would recommend then hosting your photos on another cloud-hosting provider such as Apple Photos.


2: Download Facebook's calendar
--
You can use [this Chrome extension](https://chrome.google.com/webstore/detail/birthday-calendar-extract/imielmggcccenhgncmpjlehemlinhjjo) for that, it's quite straightforward.

3: Unfriend, unlike, and delete all photos
--
There are quite a few command line interfaces that can help you with this:
- [https://github.com/spieglt/fb-delete](https://github.com/spieglt/fb-delete)
- [https://github.com/weskerfoot/DeleteFB](https://github.com/weskerfoot/DeleteFB])

Another strategy is to scramble the information before you delete your account, however this requires a fair bit more time and programming some custom scripts.


4: Announce departure
--
As a final step I suggest announcing to your network that you're leaving the platform, leaving alternative contact details and requesting that others don't upload photos of you.
The process should take a few days to complete on Facebook's end, at which point your data is _supposedly_ deleted from their servers.

Now enjoy being away from the platform - try to select for high value activities to do in your free time, and notice how you'll focus more on your most meaningful and rewarding relationships.