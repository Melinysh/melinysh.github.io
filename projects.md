---
layout: page
title: Projects
permalink: /projects/
---
Not everything that I am currently working on or have worked on is listed here.

Dropboxfs
======

Winter 2018
-------

I've had an [interest in FUSE](https://melinysh.me/go,/fuse/2016/12/31/exploring-go-fuse.html) for a while now, but I hadn't felt the real need to create something useful with it until recently. I'm back at school and I find my work environments for school assignments are spread out over multiple devices, like my Mac, my iPad Pro, and the school's servers. I'd prefer to use a specific device depending on the situation, but I don't always have my most update copy of my work on there. Instead I started storing things on Dropbox, which made it easier to work across my Mac and iPad, yet not on the school servers. I first looked at a few existing solutions, like [ff4d](https://github.com/realriot/ff4d), but I had issues with how it performed. Dropbox has a [Linux daemon](https://www.dropbox.com/install-linux), but it downloads your full Dropbox for you to use it. On school servers, I do not have the disk space to do this. This is where I decided it would be a good use of my time building my own FUSE client for Dropbox. Thus, [dropboxfs](https://github.com/Melinysh/dropboxfs) was born. Dropboxfs only loads what you need, when you need it, with limited caching.

Fastify
======

Winter 2017
-------

Fastify is a download accelerator as a service. Everyday, millions of people download large files and Fastify provides a way to speed up the transfer. When users give a URL to the file they'd like to download, Fastify distributes the link to each server in the our EC2 cluster where the file is downloaded, and is then seeded as a BitTorrent while we return a `.torrent` file to the user. This `.torrent` file can be used in your favourite BitTorrent client to download your file much faster than a conventional download.
In our tests Fastify was able to download a file 4x faster than a regular download. Fastify was written by [Daniel](http://prilik.ca) and I at [QHacks](http://qhacks.io) where it won the "Best Use Of AWS" prize! You can read more on [DevPost](https://devpost.com/software/fastify) and [Github](https://github.com/Melinysh/fastify).

Nfinite.space
======

Fall 2016
-------

Nfinite.space is a fresh look at cloud storage. Instead of the traditional model, where your files are stored on a company's central servers, nfinite.space leverages the distributed storage of connected clients to store your files. We have a React web app, written by my teammates [David](http://www.davidtsenter.com) and [Daniel](http://prilik.ca), upload files to a Go backend (written by me). This Go backend breaks up the file and intelligently distributes the file parts to multiple peers. Metadata about which clients hold what file parts are held in [CockroachDB](https://www.cockroachlabs.com) on our server. When the original file owner requests to download their whole file, we lookup, fetch, and recombine all file parts from peers. This project won our team "Best use of AWS" at [Hack The North](https://hackthenorth.com) and your can find the code and screenshots [here](https://github.com/Melinysh/nfinite.space).

WWDC 2016 Student Scholarship App
======

Spring 2016
-------

This is my winning application for [Apple's 2016 WWDC Student Scholarship](https://developer.apple.com/wwdc/scholarships/). This year's scholarship competition ran during my final exams, so I didn't have time to create a brand new app, but instead I iterated on my 2015 app. It is still completely written by me; no third party libraries. I added more natural card dismissals and of course, new content. You can read about this year's WWDC experience [here]({{ site.url }}/wwdc,/apple,/swift,/scholarship/2016/07/17/wwdc-2016-experience.html). I've tried to keep the app compatible with Swift 3 and iOS 10 so that anyone in the future can play around with it. You can check it out on GitHub [here](https://github.com/Melinysh/WWDC-Student-Scholarship-App-2016).  

Chkmail 
======= 

Winter 2015
-------  

Chkmail is a commandline Gmail client written in Go. As I've learned vim over the past year, I've become more accustomed to working in the terminal and I have wanted to move more of my daily workflow there as well. Chkmail allows me to check my email from the comfort of my terminal. I've built on top of Roi Martin's fantastic package, [gocui](https://github.com/jroimartin/gocui), to create a simple and easy-to-use UI. Chkmail is still under development with lots of future features I'd like to implement, given the time to do so. See Chkmail [here](https://github.com/Melinysh/Chkmail).

NearbEYE
=======

Fall 2015
------

NearbEYE is an augmented reality app for iOS that helps you explore the city of Waterloo through your smartphone. NearbEYE utilizes open data from the City of Waterloo to provide informative, relevant information instantly. The app shows you information about nearby locations in the direction you are currently looking through a custom algorithm. It won 3rd place at the City of Waterloo's [Codefest Hackathon](http://www.waterloo.ca/en/government/WaterlooCodefest.asp) and was created by [Aaron Cotter](http://aaroncotter.me/), Ethan Hardy, and myself. The code for this project can be found [here](https://github.com/Melinysh/NearbEYE).


Meshwork
=====

Fall 2015
-------

Meshwork is an iOS app that uses Apple's Multipeer Connectivity framework to find people around you and makes it a breeze to add their contact info to your phone. Meshwork also allows you to see if there are people nearby that you know and displays all your peers in a beautiful graph. Meshwork requires no internet connection to function, unlike other contact sharing apps. It was created at [Hack The North](http://hackthenorth.com) by [David Tsenter](http://www.davidtsenter.com), [Jonathan Galperin](http://jgalperin.github.io), [Sam Haves](http://shaves.tk), and myself. Check it out on [GitHub](https://github.com/Melinysh/Meshwork).

PyMake
=======

Fall 2015
-------

PyMake is a Makefile generator written in Python. It currently works with C, C++, Go and Fortran projects. When experimenting with small C++ projects, I found it tedious to write a Makefile and to hard code the configuration and compiler flags for each project. PyMake was created to remedy this. PyMake uses variables to allow for easy reconfiguration at the command line and has support for `make install`, `make uninstall`, and `make clean`. You can view the source and download PyMake from its [GitHub repository](https://github.com/Melinysh/PyMake).

MarketMesh
=====

Summer 2015
-------

MarketMesh is an iOS app for local marketplaces built upon a peer-to-peer mesh network. It allows for an individual to list physical objects or services for sale and displays it to nearby peers. One such service that is built-in to the app is a wireless proxy service. The proxy service allows a device's data connection to be shared and rented through the peer-to-peer network to another device. This app was created at the [Tech Retreat Hackathon](http://techretreat.ca) by [Alex Tomala](http://atomala.com), [David Tsenter](http://www.davidtsenter.com), [Jonathan Galperin](http://jgalperin.github.io) and myself. In the 9 hours we had to implement this idea, we built a working prototype that also included the proxy service. MarketMesh won 1st place for its technical difficulty, creativity, and usefulness. Take a look at the MarketMesh post on [DevPost](http://devpost.com/software/marketmesh).

WWDC 2015 Student Scholarship App
=======

Spring 2015
-----

This is my winning application for [Apple's 2015 WWDC Student Scholarship](https://developer.apple.com/wwdc/scholarships/). It is entirely written in Swift, except for one portion that was a small Objective-C game I made in 2014 (before Swift even existed). It is completely written by me; no third party libraries! I had the idea of creating a stacked card interface for awhile, but I was blown away with what I was able to accomplish. You can read about my WWDC experience [here]({{ site.url }}/wwdc,/apple,/swift/2015/07/27/wwdc-2015-experience.html). I have only updated the app to support Swift 2 and iOS 9 so that anyone in the future can launch and explore my app without trouble. The core functionality and content of the app remains the same as the day I submitted it. You can check it out on GitHub [here](https://github.com/Melinysh/WWDC-2015-Student-App).

Project Clay
=======

Spring 2015
------

Project Clay is my personal server side service that recreates the removed Twitter [Activity](https://blog.twitter.com/2015/updating-trends-on-mobile) feature by collecting and deriving this same data and more (unfollows, unfavourites). Given your user ID, API keys and correct OAuth tokens, Project Clay will continuously scrape, store, and analyze Twitter data with a Mongo database. Complete data (user profile, user tweets, favourites, followers, and following) is collected for all users up to two degrees away and just user profiles for the third degree. You can read more about the project by seeing the sanitized [snapshot](https://github.com/Melinysh/Project-Clay).

Sweatervest.club 
======

Winter 2015 
------

Sweatervest.club was a website created by a friend and I for a final project in my Chemistry class. The website had three main components: a chemistry specific calculator, links to Khan Academy videos, and PDFs of important class handouts. My friend wrote the calculator in Flash while I handled the rest of the website. Sweatervest.club had more than 3,600 page views, with most of them from Russia... We ran the website for one semester of the school term and was often recommended to other students by the Chemistry teacher.

Rez Reader 
=======

Fall 2014
-----
Rez Reader was my first iOS app on the App Store. It's a feed reader I created out of dissatisfaction with the existing offering of feed readers. I started it right after I attended WWDC 2014, where I was first introduced to Swift and met with designers about my ideas for the UI. It was through the development of this application that I learned Swift. Rez Reader was launched in early October 2014 and I have updated it multiple times since. It was downloaded by users around the world and exceeded my expectations. I enjoyed my time working on Rez Reader, but it is no longer maintained as I want to move on to bigger things. I have since [removed Rez Reader from the App Store]({{ site.url }}/ios,/apps,/swift,/app/store,/rez/2016/03/01/rez-reader-discontinued.html).

