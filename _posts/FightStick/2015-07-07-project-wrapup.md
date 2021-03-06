---
title:  "Wrapping it all up!"
categories:
  - FightStick
tags:
  - xinput
  - fightstick
  - Wrap Up

author_profile: false

sidebar:
  title: "FightStick Project"
  nav: project_fightstick_sidebar
---
We finally did it! We have created a custom USB device on the TeensyLC and allowed it to send the packets expected from a X360 controller. We even went as far as to allow it to receive LED commands from the host. This writeup became much longer than expected rather quickly. It also can be a bit dry at times, I admit that. That is partially the reason the writeup took so long to complete. I really wanted to go over everything as much as possible and then allow the reader the ability to skip what they deemed unnecessary. I figured the first few sections would be most useful for the advanced readers since they discuss more of how the controllers work and creating the non standard HID device. I didn’t want to leave any newcomers in the dust during this write up either. Since this project crosses two hobbies, there is a chance that people who have little or no experience with embedded development will end up here because of the fightstick aspect.

I am hoping this code gets ported to many other devices. You just need to learn how the USB stack interfaces with the rest of the code and then the same principles discussed here should apply. This should open the door for a lot of cool controllers to be developed and directly supported by current generation of games on the PC. It would be nice to see people add to this project and make it better. Even while doing this writeup, I noticed many instances where I could have reduced cycle times and plenty of places where I used “magic numbers” instead of defines and constants. One main function I am going to be looking into is possibly switching out the descriptors with PS3 compatible ones. I would like to be able to hold a button while plugging the device in and have it overwrite the current descriptors with descriptors similar to a dualshock 3. Adding PS3 support accessible with a button press would be a great addition to the fightstick.

I hope everyone enjoyed this project and I am excited to see what is done with it. Always feel free to email me links to some of the projects you may have done using the XINPUT code provided here!
