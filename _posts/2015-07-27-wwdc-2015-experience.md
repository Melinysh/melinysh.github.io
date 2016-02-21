---
layout: post
title: "My WWDC 2015 Student Experience"
date: 2015-07-27
categories: WWDC, Apple, Swift
---

{% include image.html img="wwdc-2014-group.jpg" title="WWDC 2014 Group Photo" caption="Can you find me in this photo from WWDC 2014? Source: Apple" url="https://developer.apple.com/wwdc/scholarships/" %}

I was extremely excited when I was awarded one of Apple's World Wide Developers Conference Student Scholarships in 2014, but I was even more elated when I was awarded the WWDC Student Scholarship in 2015. I left WWDC 2014 feeling like I could have learned more from the week, if I had managed my time and priorities better. The WWDC 2015 scholarship gave me another opportunity and I did not waste it. 

## About The Scholarship
The [WWDC Student Scholarship](https://developer.apple.com/wwdc/scholarships/) has been a program for many years, but in different forms. I heard from one developer that he attended WWDC 2006 on a scholarship given to him only because WWDC didn't sell out so Apple gave away extra tickets. Another developer told me that he had to write an essay about himself a couple of years ago.

The WWDC Student Scholarship may seem a bit misleading to some because it is not money for school, but rather a paid WWDC ticket. These tickets cost $1599 USD and aren't easy to come by. In recent years, as WWDC sold out so quickly, Apple moved to a lottery style system for allowing the purchase of tickets. Winning the scholarship guaranteed that you will have a conference ticket for free. 

I wanted to apply for the WWDC 2013 scholarship, but I didn't have confidence in programming skills so I passed on it. In 2014, I felt why not at least try? I looked up the [WWDC Scholarship Guidelines and Terms](https://developer.apple.com/wwdc/scholarships/Scholarship-Guidelines-and-Terms.pdf) and started. Over the past two years, the guidelines haven't changed much. They basically outlined the following requirements for the application:

- Mac or iOS app that highlights accomplishments and achievements
- Use your name as the app name
- App must be written in Objective-C only (2014)
- App must be written in Swift or Swift and Objective-C (2015)
- Your app builds without errors
- All app content must be in English
- You must provide some proof of enrollment for your school, including contact info

Overall, the requirements were pretty straightforward. Not once in the last two WWDCs have I met a student who made a Mac app for the scholarship, just saying. Apple is clearly pushing young developers toward Swift and I think it's the right option. After the announcement, I only had about 10 days until scholarships were due, but my WWDC 2015 app idea had been brewing for months. 

## My WWDC App
I wanted my WWDC 2015 app to display the best of my iOS programming skills with a fun user interface. The idea of a stacked deck of cards appealed to me and it became the focus of my app. The order of the cards were important too. As you delve further into the deck, the more recent the information becomes. Within the first couple of days after the announcement I had built the card deck engine. When a user first opened the app, the cards fell down in front of them. Slowly, at first, then speeding up. Instead of having the card stack perfectly upon each other, blocking the one beneath it, each card was rotated at a random angle when it fell.   

{% include image.html img="wwdc-2015-mockup.png" caption="A very early mockup of the card based UI" %}

The result was a natural feeling deck of cards and every launch of the app provided a slightly different view. When the user tapped on the top card, the card sprung up to fill the whole screen and provided more information to the user. When the user was done, a press of a button would shrink the card back down to the deck where, with the flick of a finger, the user could slide the card off the deck. I added a few extra things to the final design that you can't see in the mockup. The background color changes to a slightly lighter hue of the color of the current card and the UIProgressView at the top that fills as you go further into the deck also changes to the color of the current card. You can see the final UI below. 

{% include image.html img="wwdc-2015-card-deck.png" %}

Of course development didn't go without its challenges.

### Challenge #1: Choppy UI
I thought the card deck was a beautiful, playful UI, but it had a problem. With all the shadows on the cards, performance while dragging was terrible. The frame rate fell as all the shadows needed to be redrawn each time the user moved the card.

{% include image.html img="slow-cards.gif" caption="See the lag when I swipe a card away in the simulator?"%}

StackOverflow pointed me in the right direction, but created a new visual problem.

{% highlight Swift %}
card.layer.drawsAsynchronously = true  
card.layer.shadowPath = UIBezierPath(roundedRect: card.bounds, cornerRadius: card.layer.cornerRadius).CGPath
{% endhighlight %}

{% include image.html img="shadow-card.gif" %}

Looks like the shadow doesn't get redrawn with the card's constraints and things are still a bit slow. Looking around, overriding the card view's layoutSubviews function did the trick.

{% highlight Swift %}
override func layoutSubviews() {
	super.layoutSubviews()
	self.layer.shadowPath = UIBezierPath(roundedRect: self.bounds, cornerRadius: self.layer.cornerRadius).CGPath
}
{% endhighlight %}

{% include image.html img="smooth-cards.gif" %}

After those minor tweaks, the card dragging was silky smooth and all of	the shadows were realistic. Seems like an easy fix in retrospect, but at the time I had very little experience with CALayers and UIBezierPaths.

### Challenge #2: Autolayout and Animations
As you can see above, a card on the deck consists of a UIImageView and a UILabel. When the user taps on the card, I had the UILabel fade and wanted the UIImageView to maintain its ratio and scale alongside the card to fill the screen. This challenge was much harder. At first, I tried to animate the card like this:

{% highlight Swift %}
UIView.animateWithDuration(0.3, animations: { () -> Void in
	card.label.alpha = 0   
	card.frame = self.view.frame
	card.imageView.frame = CGRect(x: 0, y: 0, width: self.view.frame.width, height: self.view.frame.width * (2.0/3.0))
	card.layer.shadowOpacity = 0
}
{% endhighlight %}
{% include image.html img="choppy-image.gif" %}

The problem is the UIImageView wouldn't animate until the card had finished animating. This resulted in an odd expansion of the card that felt completely unnatural. I imagine that this is the result of using autolayout and the UIImageView maintaining its constraints during an animation of its superview? I never had the time during development to figure this out entirely. With only a few days left, I had to come up with a bit of a hack for animating the expansion of a card:

{% highlight Swift %}
// setup a UIImageView to animate instead of the one on the card
topCardImage = UIImageView(image: UIImage(named: cardInfo.imageName))
topCardImage.frame = card.imageView.frame
topCardImage.layer.cornerRadius = card.imageView.layer.cornerRadius
topCardImage.layer.masksToBounds = true
self.view.insertSubview(topCardImage, aboveSubview: card)

UIView.animateWithDuration(0.05, animations: { () -> Void in
		card.label.alpha = 0
		card.imageView.alpha = 0 // just so some images won't been seen below topCardImage during scale animations
	}, completion: { (finished) -> Void in
		UIView.animateWithDuration(0.3, animations: { () -> Void in
			self.topCardImage.frame = CGRect(x: 0, y: 0, width: self.view.frame.width, height: self.view.frame.width * (2.0/3.0))
			card.frame = self.view.frame
			card.layer.shadowOpacity = 0
		})
})
{% endhighlight %}

{% include image.html img="smooth-image.gif" %}

Yes, I didn't think this was ideal either. So when the user taps on the card, the UIImageView that they see expanding isn't the real one from the card. This also added complexity in the code for when the user returns to the card deck. You wouldn't believe all the other crazy animation things I tried to get to work, but this one worked, and it worked every time. If you want to see the whole UIView animation which is actually much longer, check out the [source](https://github.com/Melinysh/WWDC-2015-Student-App/blob/master/Stephen%20Melinyshyn/MasterViewController.swift#L292-327). 

I submitted the app a day before the deadline. I felt a lot more confident with my WWDC 2015 app than my WWDC 2014 app, but I also knew that competition would be greater. In 2014, there was 200 scholarships available and in 2015 there was 350 available. I thought it would be a more difficult competition in 2015 because not only were more people aware of the scholarship and interested in app development, but Apple allowed submissions from participants of STEM organizations. Later, at WWDC 2015, students winners were told that there was twice as many applicants this year than in 2014!

## Acceptance

{% include image.html img="wwdc-2015-email.png" %}

On May 8th, I received the email above and was ecstatic. Since I was 18, was able to travel and check into a hotel by myself. Some of my best times during my week away occurred even before my flight took off.

## WWDC Week
  
### Saturday
On Saturday June 6th, I flew from Toronto to San Francisco. In the gate, I had already met another WWDC scholarship winner and two other attendees. The two other attendees worked for a major Canadian bank and I was seated next to one for the flight. He's a senior executive for the bank's mobile solutions department and this WWDC was his 7th. As we chatted, I mentioned I was going to Waterloo and he told me he has a friend who took the same program as me at the University of Waterloo. He gave me his business card and encouraged me to apply to his bank for [my co-op](https://uwaterloo.ca/software-engineering/future-undergraduate-students/co-op-and-careers). I took my first Uber from SFO to my hotel and it only cost me $0.64 because of a $30 first time discount. I had the rest of the afternoon to myself in San Francisco so I walked around, took a cable car to Fisherman's Wharf and strolled along the coastline.

{% include image.html img="moscone-wwdc-2015.jpeg" caption="My photo of Moscone West Convention Center on Saturday" %}

### Sunday 
Registration for attendees opened at 9:00am inside Moscone West. There were separate long lines for students, but it only took me around 30 minutes to get to the front. Along with my conference badge, I was given a stylish WWDC 2015 jacket and directions to the student orientation at the nearby Four Seasons Hotel that started at 11. 

I arrived at the Four Seasons at 10:45, but it already seemed like I was late. There was food, drinks, Apple engineers and lots of students mingling. I met up with a few I had just met in line at registration. At around 1 they opened up a large presentation room that was setup strikingly similar to Presidio, the keynote room. I tagged alongside another eager student and managed to snag sets in second row! Right behind some of the speakers. A series of excellent presentations followed and we were told tips on how to make the most of such a busy week. After that wrapped up we grabbed a big group photo and returned to the mingling area. It was these last two hours of the orientation that I met many more students and Apple engineers, some of whom recognized and remembered me from WWDC 2014! The student orientation was a fantastic event for students, much better than the Apple Store event last year that felt a little too cramped. I didn't stay up too late on Sunday because the keynote was right around the corner.

### Monday 
3:30: Alarm. 4:00: In line. 8:30: In building. 9:40: Seated. 10:00: Keynote begins.

Lining up at 4:00am for a 10:00am keynote may sound crazy, but some students I talked to lined up around 2am. The six hours I spent in line weren't wasted just waiting patiently (or impatiently?), I was surrounded by plenty of excited chatty developers. The two developers in front of me work on apps for a large American news agency and have a great technical podcast called [Pseudocode](http://pseudocode.fm) that you should check out. They gave me some great interview advice that I'll have to remember for my co-op interviews. At around 7:00 the line started to move forward and compress. Apple had cameras out scanning cheering sections of the line. I wonder if I'll see myself in a video in a couple of months. I'm not sure if I'll look too excited or too tired. 

I had terrific seats, two rows behind the VIP section. When the keynote finished, the usual mass exodus occurred, but instead of leaving with the rest of the crowd, a group of students and I made our way to the front stage where Tim Cook and Eddy Cue were taking group pictures with students. What you can't really tell from watching the keynote is that there is a 1 meter wide, 1 meter deep gap between the stage and the presentation screen. I only noticed when I almost fell off the stage into it.

I was disappointed to see that the incredibly fun [Stump the Experts](https://en.wikipedia.org/wiki/Stump_the_Experts) event seemed to be discontinued this year.

{% include image.html img="wwdc-2015-banner.png" %}

### Tuesday
The first day of labs and sessions! I spent my morning converting my projects to Swift 2, attending the [What's New in Swift](https://developer.apple.com/videos/wwdc/2015/?id=106) session. The Swift labs ran all day, everyday of the conference and were a huge help for me. The first floor Ethernet connection's speed reached 900 Mbit/s and absolutely blew my mind.

{% include image.html img="wwdc-1st-floor.jpg" caption="Coming up from the first floor of Moscone West" %}


Tuesday night I went out with some other students to LinkedIn's WWDC Happy Hour at their San Francisco offices, just down the street from Moscone West. I was surprised that I had to sign a nondisclosure agreement just to get in! Most of the other nightly events outside of the conference were fully booked. Some smart students RSVP'd for most of the events well in advance and had plans every night of the week, something I think I should have done.



### Wednesday
Wednesday continued on with labs and sessions. Some of the session I watched were [Achieving All-day Batter Life](https://developer.apple.com/videos/wwdc/2015/?id=707), [Introducing Search APIs](https://developer.apple.com/videos/wwdc/2015/?id=709), and one of the best sessions of the week, [Protocol-Oriented Programming in Swift](https://developer.apple.com/videos/wwdc/2015/?id=408). The first Core Data Lab was that afternoon and as I expected, the line up was long. I was seated at a round table, enqueued with 4 other developers waiting for a Core Data engineer. The Core Data engineer was patient and very intelligent. She urged me to see the Core Data session that was taking place the next day that would answer some of my questions. Also that day, WWDC attendees found out that the band [Walk The Moon](http://www.walkthemoonband.com) would be playing at the annual Bash in Yerba Buena Gardens. 

{% include image.html img="wwdc-watching-lounge.jpg" caption="Comfy lounges were setup on the first floor for watching the current Presidio session" %}

### Thursday
[Optimizing Swift Performance](https://developer.apple.com/videos/wwdc/2015/?id=409), and [Continuous Integration and Code Coverage in Xcode](https://developer.apple.com/videos/wwdc/2015/?id=410) kicked off the day. Thursday consisted mostly of labs, but of course I caught the [What's New In Core Data](https://developer.apple.com/videos/wwdc/2015/?id=220) session. The Core Data engineer was right, it did answer questions, but gave me some more so I returned to the ever-helpful labs. 

{% include image.html img="wwdc-2nd-floor.jpg" caption="Looking down from the stairs onto the second floor" %}

That night at the Bash, all the students were gathered together again atop the Meteron with Apple engineers and executives. Since I was 18, I was allowed to attend the actual Bash party down in the Gardens, but chose to stay in the student area as some great networking opportunities are presented. One person I talked to was Dave Delong, Frameworks Evangelist for Apple. He told me that the most valuable, yet under appreciated thing announced at WWDC was [GameplayKit](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/). He made the point that when developers who don't make games hear "Gameplay", they immediately dismiss its usefulness, but they don't realize that there are numerous things in GameplayKit, such as state machines, that could be incredibly useful outside of games. I made a note to myself to check it out later that night. 

{% include image.html img="bash-2015.png" caption="A snippet from a panorama I took from the student area on top of the Meteron" %}
### Friday

{% include image.html img="sausage-party-truck.png" caption="This truck kept circling Moscone as we were in line Friday morning" %}


Friday was the last day and an important one for me. I lined up at 6:30 (doors open at 8:00), so I would be able to get an appointment for the User Interface Design Lab. The UI Design lab is one of two labs that require an appointment. In 2014, I was able to line up at 7:30 for the UI Design lab and be booked for the same time that I did this year. Friday is the quietest day as many people leave before the weekend begins, but labs and sessions were still in full swing. The very first talk of the day was one of my favourites, [Advanced NSOperations](https://developer.apple.com/videos/wwdc/2015/?id=226). The session looked into how the WWDC app was created with NSOperations at its core. It was a terrific session and I learned so much about the flexibility and power that NSOperations have to offer. 

The rest of the day was packed with labs. The UI Design lab, Xcode lab, Swift lab, UIKit and UIKit Dynamics lab, and the GameplayKit lab. Surprisingly, I was only one of two attendees in the GameplayKit lab. I met the team and they were incredibly helpful in not only explaining more about GameplayKit's potential, but also gave me resources to begin exploring machine learning, an area of interest for me. 

{% include image.html img="wwdc-3rd-pano.jpg" caption="Panorama of the 3rd floor" %}

With the evening to myself and an early flight the next morning, I took another cable car back down to Fisherman's Wharf and walked further along the coast. When I was at the end of the pier near Fort Mason, I turned around and took this photo. 

{% include image.html img="coast.png" %}

### Saturday
I woke up at 3:30am to make sure I'd arrive timely at SFO for my 7:00 flight. The five hour flight gave me the chance to catch up on some sessions that I had downloaded the night before and to start playing around with GameplayKit. I didn't make it home until 9:00 that night, exhausted. 


## Looking Back
The value of the conference cannot be overstated. I met so many brilliant and talented people during the week I spent there, like the student who also writes Metal tutorials for [Ray Wenderlich](http://www.raywenderlich.com) or the 14 year old with an internship in the valley. I'm still trying to catch up on all the sessions that I skipped just so I could go to labs. The WWDC Student Scholarship gives young, aspiring developers the chance to delve deeper into their passion and reward their curiosity and thirst for knowledge. 

### Tips for scholarship applicants
- Go for it! You have nothing to lose by applying
- Experiment with your app and care about good design
- Take advantage of newer technologies, such as autolayout
- Make sure your app works nicely on both iPhone and iPad 
- Don't wait to submit your app 20 minutes before the deadline
- Get your friends to test your app and get their feedback
- Take a look at the [GitHub page](https://github.com/wwdc/2015) of past WWDC app submissions

There are two points I want to make really clear. The first is use new technologies. If you don't take advantage of newer tech, especially autolayout, the more outdated your app will seem. This doesn't mean you have to have a Watch app, extensions, insane UI dynamics, tons of integration with Kits, but be sensible about following development trends. Secondly, make sure your app runs nicely on both iPad and iPhone. There are so many universal apps on the App Store that it is something we should expect from all apps. I was told by an Apple executive who judged some of the applications that an app that didn't use autolayout and/or didn't support both iPad and iPhone was basically disregarded. You need to put the effort in. 

### Tips for scholarship winners
- Congrats! Don't tire yourself out in the first few days
- Don't spend too much time hanging out in the Student Lounge, go out and learn!
- RSVP early to a lot of nightly fun events, even if you're not sure you can make it
- Network, network, network. You're surrounded by extremely talented people, talk to them
- Try to attend more labs than sessions. Sessions can always be watched later. You can't get Apple engineers to sit down with you any other week!
- Don't be pushy about getting pictures with executives (I chose not to take any selfies with them)
- Try to get a UI Design lab appointment, it's the most valuable in my opinion
- Post-WWDC depression is real

I want to touch on the pictures with executives part. Yes, I guess it is cool. They're basically celebrities in the iOS developer world, but is it necessary? For some people, it would be. For me, I don't feel I need a picture. When most students squish in together to try to grab a selfie with Tim Cook, I stand back. It has always been the perfect opportunity for me to have great one-on-one conversations with other Apple executives and engineers.



I feel incredibly honored to have attended WWDC in both 2014 and 2015 on student scholarships. You can find my WWDC 2015 app submission [here](https://github.com/Melinysh/WWDC-2015-Student-App). Try running it in Xcode if you'd like. I don't think it's perfect, but it's an example of what can be accomplished with hard work, persistence and curiosity.
