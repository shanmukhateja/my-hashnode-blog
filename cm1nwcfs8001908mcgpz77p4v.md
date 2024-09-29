---
title: "fedimg: ActivityPub server written in NodeJS"
datePublished: Sun Sep 29 2024 18:10:42 GMT+0000 (Coordinated Universal Time)
cuid: cm1nwcfs8001908mcgpz77p4v
slug: fedimg-activitypub-server-written-in-nodejs
tags: nodejs, opensource-journey, mastodon, activitypub

---

# Background

I started the project just because I wanted to build something again. After looking at some projects which I use daily, I realized I wanted to build an [ActivityPub](https://activitypub.rocks/) server.

I go into detail in the article but this spec has been a fascinating concept ever since I learned about its existence back in 2022.

I believe in “learning by building stuff” ideology and as such, this project will help me understand a lot of concepts including ActivityPub, building sophisticated social media experiences and so on.

## Inspiration

I have been following activity around [ActivityPub](https://activitypub.rocks/) since 2022. It was also around the time when I migrated from Twitter to [Mastodon](https://social.linux.pizza/@shanmukhateja/).

I would highly recommend checking out Mastodon and ActivityPub. It could truly be the next-gen social media platform, if given a chance.

There are several projects which implement ActivityPub but the way they *currently* work is, each project manages one *type* of content - for example:

1. Mastodon is a microblogging platform.
    
2. Lemmy & Kbin are link aggregator platforms similar to Reddit.
    
3. PeerTube is a video sharing platform.
    

> Unlike traditional social media platforms, these software have 2 fascinating features:
> 
> 1. Allow following users across different ActivityPub servers (collectively called the #Fediverse).
>     
> 
> 2. Users are allowed to migrate to different server hosted at a different domain within the same project. For example, a user from `mastodonserver1.com` can migrate to another Mastodon server hosted at `techstuffonly.com`.
>     

# Goals

I aim to accomplish the following goals for this project:

1. Multiple content types are supported - text, images and short videos (sometime in the near future)
    
2. “My Feed” section that is **tuned according to your followers & following collections.** *I need to research this a lot before I begin!*
    

## License

I wanted to go with MIT license just like I do with all my personal projects but after some consideration, I decided to go with AGPLv3 for this project. It is identical to that of Mastodon. The rationale for this choice stems from the fact that I studied Mastodon’s code for many days for implementation purposes.

## Progress

This project lives on GitHub [here](https://github.com/shanmukhateja/fedimg). It is bare-bones as of writing as I have prioritized building necessary features into the project.

> This is actively under development and things will change (and break) often.

<iframe src="https://social.linux.pizza/@shanmukhateja/113060374960269456/embed" class="mastodon-embed" style="width:100%;border:0" height="350"></iframe>

I have recently returned to the project from a 7 month hiatus. This personal time has allowed me to think clearly and I am happy to report it’s been a good decision. The project lives!!! :)

<iframe src="https://social.linux.pizza/@shanmukhateja/113114755696074592/embed" class="mastodon-embed" style="width:100%;border:0" height="460"></iframe>

### What works:

1. Webfinger endpoint \[[Ref](https://webfinger.net/)\]\[[Code](https://github.com/shanmukhateja/fedimg/blob/main/src/routes/well-known.route.ts#L11)\]
    
2. NodeInfo endpoint \[[Ref](https://github.com/jhass/nodeinfo)\] \[[Code](https://github.com/shanmukhateja/fedimg/blob/main/src/routes/nodeinfo.route.ts)\]
    
3. User registration & login
    
4. Follow a remote user
    
5. Accept remote user’s Follow request (can’t Reject though!)
    
6. Profile page with option to upload profile picture and set “display name”
    

### What’s missing:

There are many things missing in my implementation but here’s an overview of what I think is important at the moment.

1. Landing page
    
2. UI is terrible and in some cases, it doesn’t exist yet.
    
3. Reject a remote user’s follow - UI & backend
    
4. Create Activity management
    
5. Undo activity management
    
6. Implement `/actor/outbox` endpoint
    
7. Severe lack of documentation for project
    

# Challenges

1. It was **hell** trying to implement HTTP Signatures. This was mandatory as all AP severs won’t accept certain requests without it. I remember spending several weeks trying different things to get the expected output.
    
2. I ran into trouble setting up generating public & private keys for registered users.
    
3. Staying motivated was a **HUGE** problem. I always got cornered with [imposter syndrome](https://en.wikipedia.org/wiki/Impostor_syndrome) when something fails.
    
4. The “do it right the first time” entity in my head was difficult to negotiate. We currently have several `FIXME` comments throughout the code because that’s the ONLY way I was able to move forward when building something this huge.
    

# Conclusion

I am excited about this project and the learning potential I gain from this experiment is immense. Consider following my Mastodon account [here](https://social.linux.pizza/@shanmukhateja/) or following my project’s hashtag [here](https://social.linux.pizza/tags/fedimg) for more updates.

I plan on making this a series provided I have something new to report!

Bye for now :)