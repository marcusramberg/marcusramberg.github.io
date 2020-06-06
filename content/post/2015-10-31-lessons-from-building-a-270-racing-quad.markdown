---
date: "2015-10-31T00:00:00Z"
published: true
title: Lessons from building a 270 racing quad
---

![quad in the grass](https://scontent-ams3-1.xx.fbcdn.net/hphotos-xpf1/v/t34.0-12/12181999_10153549847170659_1811329990_n.jpg?oh=7da29c3733b7307b5af1f0413279b1a7&oe=5637ABDB)

After buying A little toy quad copter this summer, I got very interested in RC flying. My wife would probably say obsessed. :) In particular I'm interested in First Person View racing.

This is a hobby where you crash a lot, so I decided to build it from scratch so I would be able to service it. Turns out it is a lot of work! Yesterday I finally reached my goal of flying FPV, and it was glorious. This post is about some of the pitfalls I met on the way, so you might avoid them. 

## Bought a glass fiber frame

Glass fiber is cheap, but it's heavier and softer than carbon fiber. I'm now slowly replacing all the parts. One upside of glass fiber is that it does not conduct electricity. But really, you want carbon fiber.

## Discharged my lipo

In order to power a quad, you need to discharge a lot of electricity in a short time. The best batteries for this task are lithium polymer batteries. They capable of discharging a lot of amps. However they have some significant disadvantages.

For one they are highly flammable. What I didn't know is that they get unstable and unable to hold charge. While I was building and testing, the motors suddenly stopped. Turned out I had depleted it, and my charger refused to charge it. I tried forcing it to take current, but when it started heating up and swelling I decided to accept defeat and bought another.

## Put the motors on in the wrong order

In order to create lift, quadcopters need to have two clockwise and and two counterclockwise motors. Brushless motors can spin both ways. You just need to switch two of the cables to change the direction. Unfortunately the screws holding the propellers are ordered. I mixed up these, so the propellers kept unscrewing themselves when I was attempting to take off. Spectacular fail. I didn't want to remount all the motors, so I changed the flight controller to use a custom mix, and reversed the order of all my motors.                

## Put the flight controller in the wrong orientation

My first test flight was quite disheartening. As soon as I tried taking off, the copter flipped around and broke its propellers. After some digging, I realized this was because the I had mounted the naze32 flight controller sideways. Luckily you can tell the flight controller which direction is forwards. You can see if you have this problem by checking the visualization of the copter in cleanflight configurator

## Attached the motors with nylon screws

One bright idea I had when building the copter was using nylon screws to attach my motors. The thinking was that I would rather have the motor come off than break an arm in a crash. That part worked fine, however the nylon screws can't take all the load from the motors, so they started snapping even when I wasn't crashing. This meant I needed to remount all the motors after all. 
 
## Bought the wrong kind of 5.8ghz antenna.

I didn't know this, but there's two different standards for the 5.8 ghz antennas commonly used for video transmission from the copter to the pilot. One is SMA, and the other is RP-SMA. Of course I managed to buy the wrong one, and ended up with two male pins facing each other. Not good! Fortunately, I managed to find an adapter at the newly opened [kjell & co](http://kjell.com/). 

These are the main mistakes I've made so far. I'm still a newbie in this hobby, but I'm hoping that I'll be able to increase my build-to-fly ratio in the next months, and work more on my pilot skills. I still have some improvements planned tho, for one I probably need to make the quad more winter proof. I'm sure I still have many mistakes to come :)
