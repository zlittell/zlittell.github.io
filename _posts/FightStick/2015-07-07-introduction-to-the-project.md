---
title:  "Introduction to the Project"
categories:
  - FightStick
tags:
  - xinput
  - fightstick
  - Introduction

author_profile: false

sidebar:
  title: "FightStick Project"
  nav: project_fightstick_sidebar
---
![Project Introduction](/assets/images/fightstick/collage-2015-05-01-20_00_26.jpg){: width="400px"}      

The idea for this project came up when discussing the release of Mortal Kombat X. The guys at Mechanical Squid Factory (our gaming group) decided they wanted to branch out into the competitive fighting game scene. We started doing research into fighting games and that brought us to a forum called Shoryuken. The realization that a regular gamepad might not provide the best experience quickly set in. We decided that a fight stick would be just too expensive and thought keyboard would work out. The entire team played through the story of MKX and thought our fingers would never function again after contorting them to the keyboard. Soon enough we couldn’t take it anymore and pieced together fight sticks. Most of our parts were ordered from FocusAttack and the cases from Art at Tek-Innovations. As you can tell, the cases are laser cut by Art and they do have a 2 week turn around time. However, they really are beautiful when assembled. We then opted for a DIY solution to a controller board. This could save us $40-$50 since we only planned on using these on the PC anyways.

I began scouring through development boards. I had to meet just a few criteria but they were strict. I needed it to be small, support USB, and most of all, be CHEAP. Many of the boards just didn’t provide enough cost benefit over an existing solution. Finally I found an awesome little ARM cortex development board called the TeensyLC. It met all the criteria with flying colors and it had an ARM on it, which was an awesome bonus. What I did not know is the frustration that would follow after receiving these boards. There was nothing wrong with the TeensyLC, it was the bundling with arduino that caused most of the headaches. I am strong in my hate against arduino. I feel that it definitely has a place, but it has recently crept into places it does not belong. In the beginning, however, the arduino integration was actually a plus. I needed a joystick that plugged into a PC. It is pretty common for a device to have a HID joystick example if it supports HID USB. I went ahead and ordered 3 of them for the group.

What happened next is what I guess you could call scope creep. The project was finished and it was working great. Standard HID USB Joystick and it showed up with our name on it. Then someone uses it in a portion of the game that doesn’t just use the DPAD. Shouldn’t be an issue right? Wrong! The character continued to just look up constantly. I tried adding extra axes and setting them to center to fix it. I even tried changing all the bit lengths for the axes to the same that the XBOX uses, but nothing would work. We then started loading up other joystick games and realizing that most of them just didn’t work or if they did, you just constantly looked up or spun in a specific direction. With a little bit of google-fu I quickly learned that I had created a joystick compatible with Dinput and that was no longer a thing unfortunately. The industry was all about Xinput, which is a real bummer for hobbyist. That’s when the scope expanded and my struggles with adjusting my work flow to the Arduino IDE began.

You can find my code in my [GIT repository][git-repo]!

[git-repo]: https://github.com/zlittell/MSF-FightStick
