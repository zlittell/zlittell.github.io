---
layout: post
title:  "Creating an Xinput Library for Teensyduino"
categories: Xinput
---
Recently Jeremy Williams from Tested! has been working on his very own
controller. This one emulated another classic arcade giant, the pinball machine.
 Running into the same issues I experienced creating a fight-stick, he came
 across my project and began contacting me. I was more than ecstatic to work
 with Jeremy on his project. I began editing my own fight-stick code to
 accommodate the needs of his pinball controller. This went on for a while
 before I had a large amount of unused code and variables sitting oddly idle on
 my screen. Making my code clunky and convoluted was not my intention, but it
 quickly became my reality. I discussed my concerns and options with Jeremy and
 he expressed that he was more than willing to rework his code base if I moved
 to an Xinput library for the teensyduino IDE. This was my new course of action
 and I think it turned out really well in the end.

I created lots of useful functions and even multiple ways of doing certain
actions based on use case. I kept adding functionality as Jeremy saw needs for
things in his project. He even suggested lots of features that he didn’t need
himself, but just happened to think of while working with the library. I
updated the fight-stick code to work with the new library and pushed it to GIT.
I also fixed compatibility issues people were having with compiling on the
latest teensyduino. I included the new fight-stick code and a simple example in
the library and made them accessible using the Example menu option.
Unfortunately, changing the code and switching to using a library nullified a
lot of the documentation I did here in my blog posts. Most of the USB portion
holds true, but has just been moved to the library. I really hope to rewrite
the “making of” blog to better suit the library, but for the time being you can
either cross reference the code into the new library or get an older version of
the fight stick code from GitHub.

My current workload for this project revolves around completing the
documentation for the library. I want to do a nice API guide write-up for using
the library. I really think this update to a library has made my code more
accessible for generic use instead of how specific it was before. I am hoping
this allows people to start crafting really neat and innovative controllers,
especially for VR use like Jeremy has done!

Check out these links:

[MSF FightStick Code Base][fightstick-codebase]  
[MSF Xinput Library Code Base][xinput-codebase]  
[Jeremy William’s PinSim Controller over at Tested!][jw-pinsim]  



Thanks  
-ZACK-

[fightstick-codebase]: https://github.com/zlittell/MSF-FightStick
[xinput-codebase]: https://github.com/zlittell/MSF-XINPUT
[jw-pinsim]: http://www.tested.com/tech/gaming/569647-how-build-pinsim-virtual-reality-pinball-machine/
