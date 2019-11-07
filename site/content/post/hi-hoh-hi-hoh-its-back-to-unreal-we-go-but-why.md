---
title: 'Hi-hoh, hi-hoh, it''s back to Unreal we go... but why?'
date: 2019-11-07T17:23:44.002Z
description: >-
  If you're on our Discord server, you'll probably remember the discussion about
  us moving The Peacenet back to Unreal Engine/.  While this wasn't met with a
  completely positive reaction, we still made the decision in favor of switching
  back to Unreal... and here's why, and here's what's going to happen.
image: /img/unreal_engine_white.png
---
## First, let's address some concerns about the engine.

I know there are definitely some concerns with Unreal - it's hard to set up on Linux, C++ is scary, and it's not really meant for a 2D UI-centric game like The Peacenet...is it?

To address the first concerns - yes, it's hard to set up Unreal on Linux.  While Epic does officially support the platform, one must compile the engine on their own and Linux users lose access to the Unreal Marketplace.  These are some fairly severe caveats in a lot of cases.  But this is only if you want to _develop_ the game...and I use Windows anyway. I can get the engine to work on Linux and I plan to support the platform, but I'll be doing actual development on a Windows system because that's just what's easier for me with my disability.

As for C++ being scary - yes... it is.  It's terrifying.  It is a big scary boogieman hiding in your closet, in your dresser, under your bed, waiting for you to fall asleep, and it's gonna get you...  _...if you've never used it before._  This is the case with any form of programming or really any skill! It's always scary on first glance.  Though, part of what makes C++ scary for most programmers who already have experience with more high-level languages, is the low-level stuff from C. _Pointers! Memory management! Buffer overflows!_ I here you screaming... **but none of this is ever used in The Peacenet or any Unreal game I've seen source code from.**  Unreal takes care of the scary boogieman stuff for you, while still leaving you with the speed and flexibility of the language.  In fact, some of the Unreal APIs are remarkably similar to the .NET Framework.

_But Unreal Engine's for big 3D games with lots of fancy graphics!_ I hear you say.  **Wrong.** It can do that stuff, it's very easy to do that stuff in Unreal.  It's definitely built to handle that stuff, but that's not the only thing the engine is capable of.  Its user interface toolkit, **UMG**, is very powerful for what we use it for.  Peacegate OS's most stable user interfaces were built using the UMG UI toolkit in Unreal.  And, with some minor elbow grease, it is possible to also leverage the graphical capabilities of the engine to add some cyberpunk effects to the game's art style.

## So why make the switch?

With that out of the way, the main reasons I want to make the switch involve making development easier for me.  I've always wanted to write a game engine, which is why I started Peace Engine again.  However, I soon realized that the engine itself is a lot more work than I have the energy to do right now during school.  And since Peacenet is my way of **releiving** stress, it doesn't make sense to add an immense amount of stress to the game's development process.

Truth is, I miss my visual GUI editor, I miss Blueprint, and I miss Unreal's audio system, and I miss my mission scripting system.  All of these things can be written in a few months of work, but why reinvent the wheel for The Peacenet? We had a solid base - and although there's definitely work to be done, it's a lot less.  And that's why I'm making the switch back.

## And what'll happen to Peace Engine?

Nothing.  It'll stay in development.  The Peacenet just won't be ported to it, that's all.

## So...yeah.

That's basically it.  We're making the switch back to Unreal.  But don't worry.  The improvements I've made in the Peace Engine port WILL NOT be lost, I will be transferring them over to the Unreal Engine - these improvements being related to the in-game Terminal, mostly.
