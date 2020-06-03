---
title:  "Dinput VS Xinput"
categories:
  - FightStick
tags:
  - xinput
  - fightstick
  - dinput

author_profile: false

sidebar:
  title: "FightStick Project"
  nav: project_fightstick_sidebar
---
![Controller Fight](/assets/images/fightstick/dinputVSxinput.jpg)  
There is likely to be quite a few misconceptions and errors in this area of my understanding. I am not a game developer and actually do very minimal PC application development. Originally games used a DLL named Dinput to communicate with controllers in their software. You could use Dinput.dll to probe a controller, learn its ins and outs, and then utilize it in your game. Microsoft deemed this too difficult a task for PC game developers. Since they were releasing a new controller, they really wanted to make it accessible to developers to work with. They decided to reinvent how developers access their controllers. The big issue that affects the hobbyist community is that it is no longer a HID device. Look at any game you play, it supports the XBOX 360 controller. Not being HID is the biggest downfall to Xinput devices (sometimes referred to as XUSB devices in Microsoft documentation). This creates a huge problem for us because without being HID you will need drivers for your device. Getting drivers signed is an expensive expenditure and not something most are willing to do for non-commercial projects.

So naturally the next thought is “Who cares? We will just stick to Dinput HID controllers!” That would work, if the whole gaming world didn’t adopt Xinput. For a long time, people supported both Xinput and Dinput in their games. Sadly, it is no longer that way. Actually many games incorrectly manage Dinput devices and cause errors to occur that make games unplayable. This is where the constant looking up or the constantly spinning left comes in. The Dinput device is actually assigned an extra axis on accident and that causes you to look whichever way it is assigned, indefinitely. Many examples of this are found on games that do not need a controller but behave incorrectly when you have one plugged in, Tribes: Ascend being a good example. So now you need to make your Dinput device look like an Xinput one. Up until this point you had one option, you would put software called X360CE (or other similar software) into your game directory, you would assign your controller’s inputs to similar ones on a 360 controller, and it would create a custom xinput.dll in the game folder that correctly mapped your controller for use.

Using X360CE was an easy option for us, but I just couldn’t do it. We worked hard on making sure these fightsticks turned out nice and I wasn’t about to settle for putting some software in all my game directories. That was when the idea was born. There has to be a way to imitate the XBOX 360 controller. I mean it’s just a USB device afterall, right? It was time to investigate!
