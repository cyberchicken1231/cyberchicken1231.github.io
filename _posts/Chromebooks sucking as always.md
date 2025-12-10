---
title: "Chromebooks sucking as always"
layout: single
author_profile: false
---
The Most Miserable Audio Fix Of My Life On An HP Chromebook

So here is the story of how I tried to install Linux on my HP Chromebook and instantly regretted every choice that led me here. I put MrChromebox firmware on it, installed Xubuntu, booted in, felt great for like five seconds, and then realized something horrible. The audio was dead. I do not mean glitchy or quiet. I mean absolutely silent. Speakers gone. Headphones gone. The system could have been a brick for all Linux cared.

Step One: Realizing This Chromebook Has A Frankenstein Audio System

Most laptops have one normal audio chip. This Chromebook has two because somebody at HP must have thought that would be funny.

MAX98357A which only exists to blast digital signals into the speakers

DA7219 which handles headphones and aux but likes to wake up with every output set to off as if it hates the user personally

Both of them sit behind the Intel Sunrise Point audio controller. ChromeOS knows all the secrets. Linux knows nothing and stares blankly at the hardware like it is looking at alien technology.

Step Two: Finding Out Why Everything Broke

This part felt like doing surgery with a spoon.

Linux had zero UCM files for this setup. Nothing told the system where audio should go. It was like the OS was holding two wires and refusing to touch them together.

The DA7219 chip starts with everything shut off. Headphones off. Line out off. Mixer outputs off. The DAC has no path. The audio literally has nowhere to travel.

The mixer routing was not just wrong. It was nonexistent. Imagine pipes in a building that do not connect to each other. That was the audio stack.

At this point I honestly thought the speakers might have physically died. I was ready to give up and accept that silence was part of the Chromebook experience.

Step Three: Building UCM Files From The Void

I ended up making a whole UCM directory by hand at:

/usr/share/alsa/ucm2/Intel/avs-max98357a-da7219/

Inside it I created configs for every part of this cursed setup because nothing existed before.

one file for the MAX98357A

one file for the DA7219

a HiFi speaker profile

a HiFi headphone profile

I had to figure out the routing for each device by reading ALSA dumps and praying I did not cause a kernel panic.

Step Four: Forcing The Mixer To Actually Turn On

You would think audio chips want to output audio. This one apparently prefers the quiet life.

I had to run these one by one:

amixer -c 3 sset "Headphone" on
amixer -c 3 sset "Line Out" on
amixer -c 3 sset "Mixer Out FilterL DACL" on
amixer -c 3 sset "Mixer Out FilterR DACR" on


Every time I ran one I felt like I was waking up another part of a monster stitched together from leftover Chromebook parts.

Step Five: Begging The System To Remember The Fix

Linux loves forgetting things. So even after fixing all of that, the settings reset every time I rebooted. I saved the mixer state using:

sudo alsactl store

Then I made a tiny script that runs on login just in case ALSA feels like ignoring the saved state. I should not have to do this but here we are.

Step Six: Making WirePlumber Stop Ignoring Me

PipeWire’s session manager refuses to read UCM files unless you poke it with a stick.

I had to add a config in:

~/.config/wireplumber/main.lua.d/

This finally convinced WirePlumber to load the routes I made instead of pretending the hardware did not exist.

Step Seven: Testing Hardware Like A Caveman

Before trusting any part of Linux again I talked straight to the hardware.

speaker-test -D hw:2,1
speaker-test -D hw:3,1


Hearing the test sound play through the chips felt like discovering water in a desert. It proved the hardware was alive after all. The software was the problem the entire time.

Step Eight: Manual Output Switching Because Of Course It Is Manual

Headphone jack detection on this machine is not fully supported on Linux. So I switch outputs by hand like a tech support caveman.

wpctl set-default 56     # headphones
wpctl set-default 59     # speakers


If I plug in headphones and expect it to switch automatically I will be disappointed.

Why This Whole Thing Felt Like Suffering

Fixing this was a long chain of tiny victories mixed with hours of staring at mixer diagrams, codec registers, UCM examples, WirePlumber logs and plain confusion. It felt like wrestling a system that was actively trying to hide every part of its audio path from me. I had to learn ALSA, UCM, PipeWire, WirePlumber and two separate audio chips just to hear a sound again.

But in the end it works. Every piece is routed correctly. The speakers play. The headphones work. And now I understand Chromebook audio at a level I never wanted but will probably brag about anyway.

If someone else hits this problem just know that the hardware is fine. The pain is only in the software.
