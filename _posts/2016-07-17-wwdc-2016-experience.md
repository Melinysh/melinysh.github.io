---
layout: post
title: My Time as a Student at WWDC 2016
description: A look back at my time as a Student Scholarship winner at WWDC 2016
date: 2016-07-17
categories: WWDC, Apple, Swift, Scholarship 
---

{% include image.html img="me-wwdc-2016.png" title="Me at WWDC 2016" caption="Hello WWDC16." %}

Last year, I wrote about [my WWDC 2015 experience]({% post_url 2015-07-27-wwdc-2015-experience %}) and this year, I returned to WWDC 2016 on another [Student Scholarship](https://developer.apple.com/wwdc/scholarships/). I had a fantastic week meeting and learning from fellow students, attendees, and engineers. I heard from some students that they read my 2015 post and enjoyed it, so this year I'm going to build off last year's post. If you haven't checked it out yet, you can find it [here]({% post_url 2015-07-27-wwdc-2015-experience %}).

## Updates on the Scholarship
The 2015 scholarship guidelines were quite similar to the 2014 guidelines, but this year, things were changed up. Applicants were told to submit any iOS, macOS, tvOS or watchOS app that demonstrated "creative use of Apple technologies." Applicants were allowed to submit any application that they've previously worked on, whether it be an app on the App Store, a school project or a personal one. In addition, applicants could submit two apps, one currently on the App Store and another private submission if they'd like to. During the application process, essay questions were also asked with answers being limited to 500 words or less.

1. Why did you choose the features and technologies that you used in your app. What were your biggest technical challenges in implementing these features and technologies, and how did you overcome them? If given more time, what would you add or improve upon?

2. If you worked on this app as a team, tell us about your team structure. How did you divide the work? What were your biggest challenges in integrating the components that each of you worked on?

3. (Optional) In what ways have you considered sharing your coding knowledge and enthusiasm for computer science with others?

I liked that these questions were asked. It provides a way for applicants to explain more about their application that may not be obvious to the judge of the submitted app. For example, I worked on my app during my final examination week, so I could explain more about what I chose to implement and did not in question #1. The last question was optional, and from an informal and bias survey it seems like most of the accepted students answered the optional question. Yes, it is optional, but it could only help your application to answer it. Within the guidelines this year the specific kind of device that your app would be tested on was specified, depending on the platform.

Travel assistance was also available for 125 students this year and could be applied for during the regular application process.


## My (Updated) WWDC App
As I mentioned above, the scholarship competition ran during my final examination period, so I had limited time to make my WWDC 2016 app. With the broader rules this year, I had a larger scope of ideas to consider, but in the face of the time crunch I decided to build upon my 2015 app. After showing it to people of the past year, I found a few areas to iterate the design and UX. Also, with the requirement of demonstrating "creative use of Apple technologies" I changed the content of the app from a personal resume style to a projects focused resume. Each card would now display another app that I've worked on that used interesting Apple technologies instead. For example, I featured [NearbEYE](https://github.com/melinysh/nearbeye), and [Meshwurk](https://github.com/meshwork) among my apps.

{% include image.html img="wwdc-2016-app.gif" title="My WWDC 2016 app" %}

In the gif above, you can see I have the same cards interface as in 2015. The main UX complaint I noticed from my 2015 app I saw was that when a user was in the expanded card, they would try to swipe away the card, but couldn't as I would transition to a completely new view controller and only provided a ❌  button to close the card. After tapping the ❌  button  the user could swipe the small card away. To fix this, this year I changed the detail architecture so cards could be swiped away right from the expanded view 😄.


Also this year I got more Swift-y and took advantage of protocols and protocol extensions to help abstract away logic for determining which type of expansion view to present and handle. Again I used a `CardInfo` struct to help pass each card's data between various parts of my app. 

{% highlight Swift %}
protocol DetailViewControllable {
	// Point of customization for each detail view controller
	func additionalSetup(withCardInfo info: CardInfo)
	var view : UIView! { get set }
	// Each expanded view would have at least one text field
	var infoField : UITextView! { get set }
}

extension DetailViewControllable {
	// Common initialization of views
	func initViews(withCardInfo info: CardInfo) {
		view.backgroundColor = info.backgroundColor
		let bgC = info.backgroundColor.cgColor.components

		// darken the background
		infoField.backgroundColor = UIColor(red: (bgC?[0])! * 0.8,
						  green: (bgC?[1])! * 0.8,
						   blue: (bgC?[2])! * 0.8,
						  alpha: 1)
		infoField.text = info.text
		infoField.textColor = info.textColor
		infoField.layer.cornerRadius = 5
		infoField.isEditable = false
		infoField.isSelectable = false
		infoField.font = UIFont(name: "Avenir-Book", size: 19)
		view.subviews.forEach { $0.alpha = 0 }
		additionalSetup(withCardInfo: info)
	}
	
	func fadeInViews() {
		UIView.animate(withDuration: 0.4) { 
			self.view.subviews.forEach { $0.alpha = 1 }
		}
	}
}
{% endhighlight %}

Each expanded view corresponded to a different view controller, like `NearbeyeCardViewController`, and conformed to `DetailViewControllable`. When a user tapped on a card I could create any view controller and cast it to `DetailViewControllable` where its methods could be used in my expansion animations regardless of what it's actual type was. This saved me a writing a multi-case `switch`. 

Elsewhere in my app I took advantage of the latest APIs to let the system do more work for me, such as using [`UIStackView`](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIStackView_Class_Reference/)s.

To follow the submission guidelines and show off creative technologies, I added screenshots of the projects, a video of MarketMesh in action and 3D Touch throughout my app.

I was also planning to use AutoLayout to support a horizontal layout as well, but I also had to study, so that feature got cut. You can see the whole project [here](https://github.com/Melinysh/WWDC-Student-Scholarship-App-2016). I've converted the syntax to Swift 3 and will verify that it functions as intended on iOS 10. 


## Acceptance

{% include image.html img="wwdc-2016-email.png" %}

After waiting anxiously most of the day, around 8 pm on May 9th I received this email. Time for WWDC round three! 🎉

## WWDC Week
  
### Sunday
I arrived late Saturday night and spent Sunday morning down in Fisherman's Wharf before the student orientation. Buses ran from Moscone West to the Mission Bay Conference Center. Here we received our jackets, badges (with a student-labelled lanyard) and, later, a free Apple TV Developer Kit 😮! After eating lunch we were invited upstairs to hear a series of short talks from Apple employees, including a scholarship winner from last year. After that we went outside for our group photo and Tim Cook joined us and was subsequently overrun. 

{% include image.html img="tim-cook-swarm.jpg" caption="Can you see his gray hair?"%}

You can read more about why I choose not to join in with the selfie mob near the end of my 2015 post. The rest of the afternoon was spent mingling with fellow students and Apple engineers inside the conference center.

### Monday
Like the two years before, I woke up and got in line around 4 am, but this year at the Bill Graham  Civic Center Auditorium.

{% include image.html img="bill-graham-2016.jpg" %}

I was happy to find that there was Wi-Fi outside here too, and was usable up until around 6 am when many more people joined us in line. After 7 Apple provided some breakfast in the park area behind the line as well as additional bathrooms. 

It 8:00 was when things started to get interesting. A few Apple employees were pulling every student they could find out of the line and towards the auditorium. We were given an additional silver plastic bracelet and n priority seating access on the left and right hand sides of the stage 😮! It's kind of funny for the students who lined up at 12 am sitting next to those who arrived at 7 am.

{% include image.html img="keynote-2016-seating.png" caption="A snippet of a panorama I took from my seat, 6th row!" %}

{% include image.html img="wwdc-2016-me-keynote.png" caption="I even made it into the keynote video, around an hour and 24 minutes in!" %}

I thought the keynote was great and after lunch returned to the auditorium for the Platform State of the Union. There was no priority seating for students, but I think my seat in the balcony had great photography benefits.

 
{% include image.html img="state-union-seating.jpg" caption="Just before the State of the Union"  %}

I capped Monday night off with Microsoft's & Xamarin's party a few floors below Twitter's HQ where I ran into some friends I met last year.

### Tuesday

Back in Moscone West, I first caught an App Store Review Lab where I validated some app ideas with a member of the App Store team to see if my ideas would make viable approved apps. I also caught some sessions throughout the day, such as [What's New in Swift](https://developer.apple.com/videos/play/wwdc2016/402/) and the great iOS overview session [What's New in Cocoa Touch](https://developer.apple.com/videos/play/wwdc2016/205/).

{% include image.html img="wwdc-2016-first-banner.png" caption="The amazing first floor banner, each sentence alluding to an app on the App Store."  %}

I rounded out the evening by attending a mixer put on by Nest at the [Thirsty Bear](http://thirstybear.com) just down the street.

### Wednesday
One of the most interesting talks I attended this year took place in the morning, [Optimizing App Startup Time](https://developer.apple.com/videos/play/wwdc2016/406/). It was split into two parts; the first on the technical side of what takes places before your application's `main` function gets called and the second part on practical ways of reducing this pre-`main` time. Xcode and Swift Open Hours were very helpful labs that day.


{% include image.html img="wwdc-2016-second-floor.jpg" caption="A look down from the stairs onto the second floor"  %}

### Thursday

{% include image.html img="wwdc-2016-ethernet-speeds.png" caption="Mind-boggling ethernet connections"  %}

Thursday is usually a really busy day and for many people, their last day at WWDC. In the morning, I caught [What's New in UICollectionView in iOS 10]( https://developer.apple.com/videos/play/wwdc2016/219/) and [Making Apps Adaptive, Part 1](https://developer.apple.com/videos/play/wwdc2016/222/). In the afternoon, I sat down with 3 Apple engineers in the Power and Performance Lab and learned more about the new UIKit animations in the UIKit and UIKit Animations Lab.

In the evening was the annual Bash. Like the keynote, it was held for the first time in the Bill Graham Civic Auditorium. In previous years, the Bash was held in Yerba Buena Gardens, near Moscone West and the students had their own private section atop the Meteron. There would be food, drinks, engineers and special guests. This year, at Bill Graham there was plenty of food and drinks, but things were different. The student section was up on the balcony and students were also free to go down onto the main floor where regular attendees were. 

This year's band was [Good Charlotte](https://en.wikipedia.org/wiki/Good_Charlotte), who, I have to admit, never heard of before WWDC. They played for around an hour, after a DJ. Unlike the Meteron, the student section of Bill Graham was quite close to the loud band so it was much harder to hold conversations. Along with a few friends, we moved down from the balcony to the main floor for the last few songs. There were no special guests on the balcony, I'd imagine it would be very difficult with the limited space and loud music, but the Bash was more fun. Here are a few photos.

{% include image.html img="wwdc-2016-bash-poster.png" caption="Bash poster outside Bill Graham"  %}

{% include image.html img="wwdc-2016-bash-pano.jpg" caption="Panorama from the student balcony with the opening DJ"  %}

{% include image.html img="wwdc-2016-bash-balcony.jpg" caption="Good Charlotte from the balcony" %}

{% include image.html img="wwdc-2016-bash-band.jpg" caption="Getting up close with Good Charlotte on the main floor"  %} 

### Friday

My last day in SF, I had a flight to catch at 4 pm, so it was a half-day for me. I caught the second part to [Making Apps Adaptive](https://developer.apple.com/videos/play/wwdc2016/233/) and also caught the [What's New in Core Data](https://developer.apple.com/videos/play/wwdc2016/242/) session. I'm really excited about the improvements to Core Data and it's Swift interface that'll make writing Core Data apps easier and more enjoyable. 

{% include image.html img="wwdc-2016-hotel-view.jpg" caption="The view from my hotel window"  %} 

## Looking Back 
Once again, attending WWDC is an invaluable experience. From the people you meet, the things you learn, and the problems you solve, it's a tiring, yet inspirational week. Near the end of [last year's post]({% post_url 2015-07-27-wwdc-2015-experience %}), I wrote tips for both applicants and recipients of the scholarship that I believe are still accurate and useful, with the exception of running on multiple devices. This year, we were told in the guidelines which device our app would be tested on, so if you're in a time crunch, you can focus on making your app work great on a single device. Also, I'd like to note for scholarship winners, there is usually a Facebook group that other winners organize to help you connect with other students that you should check out.

## Questions

Here are some common questions I've received over the past 2 years about the experience. If you have one, send me an email at [stephen.melinyshyn@gmail.com](mailto:stephen.melinyshyn@gmail.com) and I'll put it up here. 

> Why didn't I get a scholarship?

This one's a tough one. You can look online and see many submitted applications, some better than others. You might wonder why some people got in and you didn't. The scholarship application process has evolved over the years and the things Apple looks for have too. My (unreleased) 2014 app, to me at least, seems sub-par from a developer perspective. It was very basic, and had some subtle bugs. I think the winning part of my 2014 app was what it showed about me. I was able to highlight my other achievements and outlined my future goals. It showed my passion. That's what I believe made it a winning app. So if you're wondering why an app that you think is technologically inferior to yours won over you, you should understand the scholarship application is more than just the app. It's also about you. With the new essay questions asked this year, this is more apparent. Apple is looking for talented, passionate problem-solvers and you should consider this throughout your entire application. Even if you didn't win this year, I encourage you to push yourself further and apply again next year.

> Should I go to labs or sessions more?

Easy; labs. WWDC week is the only chance you have to sit down with Apple engineers and get your questions answered directly. Sessions are always available for streaming later. They should be your top priority if you have questions. What my strategy was: each morning at WWDC, pour over the schedule and plan out your day. I would first read through the labs and pick out the ones I wanted to attend, then see which sessions took place in between labs to fill up time. If there was no session or lab, I'd go to the Scholarship Lounge and relax for a bit, but I would always prioritize labs first. 

> What do I do in labs if I don't have a specific question?

I get this question the most. Labs really are the defining benefit of WWDC. The chance to go one-on-one with the Apple engineering who worked on the thing you're curious about. You want to take advantage of the lab, but you're not sure how. Having questions is the best option. Read the documentation, attend the related session or start a project using what you're not sure about to generate more questions. The thing is that many of these labs are filled with people who are interested like you, but might have really good questions. There are always filler questions like "What is the most interesting part framework X?", or "Could you give me some feedback on my code?", but these aren't really the best use of everybody's time. So I recommend having a plan when you go to really busy labs. But some labs aren't busy at all. These ones are great opportunities. You can walk in and sit down with a few engineers and pick their brain. They often are full of ideas and energy on topics not just related to the lab. In 2015 I sat down with the GamePlayKit team and they walked me through the framework and the design decisions they made along the way. We also had a great chat about AI and rule systems. Some of the quietest labs can yield the most interesting information.

TL;DR: Be respectful and have a plan for busy labs, for quiet labs you should go and pick their brains, it can be really interesting.

> Should I try to get selfies with executives?

I covered this in detail near the end of [last year's post]({% post_url 2015-07-27-wwdc-2015-experience %}).

TL;DR: If you really want to go for it, but I think there might be better uses of your time.

# Wrapping up the last 3 WWDCs
I never thought in 2014 that I'd be attending WWDC, let alone going in 2015 and 2016. The WWDC Scholarship program is the perfect opportunity for aspiring developers to dive head first into the real, exhilarating world of app development and I'm grateful to Apple for the chance to attend. It's been a wonderful 3 WWDCs. 

This time next year, I'll be on my 2B academic term at the University of Waterloo. WWDC will likely take place around the same time as my midterms, so I won't be applying for a scholarship in 2017. I'm a bit disappointed I know I won't be able to go, but I know that another keen, intelligent developer will be able to and, for that, I'm delighted. 

