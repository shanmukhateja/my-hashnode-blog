---
title: "fedimg: Shutting down the project"
datePublished: Wed Nov 20 2024 16:58:46 GMT+0000 (Coordinated Universal Time)
cuid: cm3q4o8gt000k09mn3n9x7rin
slug: fedimg-shutting-down-the-project
tags: blogging, nodejs, diary, activitypub

---

Hello,

It is with a heavy heart that I have decided to shut down the fedIMG project. This project started out of my curiosity towards the ActivityPub protocol. It began as a silly, fun and a “toy” server for educational purposes but as the development went on, I wanted to try my hand at turning into something serious (like [snac](https://fedidb.org/software/snac)).

I had also planned to run a dev series on the blog (akin to MyStudio IDE) however I quit working on the project around the end of March 2023 because of my inability to solve several features, one of which was “Follow” user feature.

This bug haunted me for the coming months as I went through several stages to grief every time I saw activity related to ActivityPub software. After a 6 month hiatus, I returned to the project with a vengeance. I had managed to fix the bug ([Link](https://github.com/shanmukhateja/fedimg/commit/5b26b1537837ff2df7e3b99077465670e5beccc7#diff-ec2d07b6bc5eef9be8e900f4bd54ac144c330325f672f5353a311341903797dcR180)) which was therapeutic and satisfying to say the least. *I slept like a baby that night!!*

With the newly found interest, I continued development and managed to land some bug fixes and other misc. additions to the project but I soon realized the **original** **spark** had died. I wasn’t feeling the fun with this project and the work kept piling up.

Also, the project was costing me money regardless of my engagement towards it. Now, due to technical reasons, I created 2 AWS EC2 instances for this project - one for the NodeJS server and the other was dedicated MySQL database. I know I could’ve used something like [ngrok](https://github.com/inconshreveable/ngrok) but their offerings needed a paid account for my needs. Also, I had to pay extra to my ISP for needing a static IP address so that I can SSH into the server to test things when they break.

Looking back, this project was a lot of fun. It was very educational and it taught me many things. After a lot of consideration, this project is being shutdown. I would rather see the project archived than let it die a slow death (abandoned).

The code will live on GitHub [here](https://github.com/shanmukhateja/fedimg/) and the repo will be archived once the article is published. I hope this will help others venturing into ActivityPub.

A huge thanks to [https://social.linux.pizza/@selea](https://social.linux.pizza/@selea) for allowing me to test fedIMG against his Mastodon server. I really appreciate his patience and compassion towards me and this project. ❤️

Here’s hoping my next passion project will succeed.

Thank you for your time.

Bye for now :-)