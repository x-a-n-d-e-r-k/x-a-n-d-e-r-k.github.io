---
layout: single
title:  "Troubleshooting Skills"
date:   2022-10-21 00:00:00 -0500
categories: general
tags: troubleshooting lifeskills
author: XanderK
published: true
---

## Troubleshooting is a Skill

Troubleshooting is frequently thought of as an art. I prefer to think of it as part art, part science. This is because although it takes some creativity to be generally good at troubleshooting, most often one can apply a rigorous playbook to troubleshooting and be successful.  

Many people look at troubleshooting the same way they look at solving riddles. You're either good at it or you're not. I disagree. Troubleshooting is a skill that can be learned and practiced. It doesn't require any inate abilities other than challenging your assumptions and getting good a breaking down a system into it's component pieces. By identifying the components that make up a system, you can get better at evaluating how those components interact with one another.  

Practicing troubleshooting allows you to see patterns and areas that could be more likely to have faults. These could be things such as targeting software components that contain parsing free-form text because they frequently fail to account for certain character combinations. They could be testing that spark plugs are working correctly in an automotive engine first when a car cranks but won't start, because it's frequently either spark or fuel that is causing the problem.  

Practice is what builds expertise and troubleshooting skills are no different. I'd like to share a few concepts that, when practiced, can result in you becoming a very talented troubleshooter.  

## Key Troubleshooting Concepts

Below are a few key troubleshooting concepts that the best troubleshooters utilize in the normal course of doing their jobs.  

* Identify Failure Condition(s)
* Identifying and Removing Assumptions
* Binary Search
* Checkpoints

Including these concepts in your troubleshooting process will dramitically increase the success of identifying and solving problems in just about any technical problem you may encounter. I have used these same concepts in everything from troubleshooting software source code, structured cabling, electrical system, and even automotive engines. Let's go through these one by one and explain what they mean.  

### Identify Failure Condition(s)

The first step to pretty much every troubleshooting session requires the ability to identify conditions which cause the failure being addressed. In most cases, you can point to a specific set of conditions that consistently result in the negative outcome you are troubleshooting. This is the most preferralbe condition for you, as you can make changes until those conditions no longer result in a failure.  

However, there are situations that you will encounter that the failure condition or conditions are not dependable. In these situations, you can use the same set of conditions over and over and sometimes they will cause a failure and sometimes they won't. The take away in these situations is that the lack of dependability on the failure condition is itself a valuable piece of information. If those conditions aren't dependable then the problem is caused by something that is inconsistent. Examples of that would be:

* Software systems
  * Race conditions - multiple execution threads reading/writing a shared resource in an inconsistent order
  * Faulty hardware - faulty hard drives, memory, network cables, etc. can cause inconsistent results  
  * Hardcoded values - values defined in source code can cause different results when run in different environments
  * System load - systems under heavy load can execute code at a reduced rate, thus potentially causing failures that affect streams of data
  * Memory leaks - failures to properly allocate/release memory in code can create situations that are very difficult to predict
* Automotive
  * Misfiring spark plugs - inconsistent ignition systems can create situations that are difficult to diagnose
  * Electrical glitches - many electrical failures can be caused by shorts caused by missing wire insulation caused by various conditions like heat, friction, or even vermin
* Structured Cabling
  * High wind - in external installations, high wind can cause additional strain on connectors and potentially stretch suspended cable, causing failures
  * Moisture - moisture can include actual standing water or as simple as high humidity which can encourage corrosion or even create electrical shorts
  * Electrical surges - insufficient grounding and surge protection can cause components to misbehave or even completely fail

These examples show a consistent way of approaching inconsistent failure conditions. The presense of inconsistent conditions itself can point to potential areas to focus your troubleshooting efforts.  

### Identifying and Removing Assumptions

One difficult concept to master is identifying and removing assumptions. This means finding areas of the system where your brain just glosses over as "that's impossible" or "can't be in that area." It's natural for us to think we know what areas are bulletproof and can't possibly be the problem. However, consistently, these areas are where we find the most frustrating and persistent bugs, specifically because they are the most overlooked areas.  

It can be critically important to stop yourself from assuming _any_ part of the system is bulletproof.  

### Binary Search

Those with experience in information technology may be familiar with the concept of the binary search. The basic premise of the concept is that you start in the middle of the 'system' and evaluate it to see if it's behaving as designed. If it isn't, then the problem is occuring _before_ the current place. If it is, then the problem is occurring _after_ the current place. Repeating this process in exceedingly smaller sections of the system allows you to very quickly narrow down the area where the problem exists.  

An easy example is in a cabling system such as cable television, network cabling, or electrical power. In all of these systems you have a point that can be identified as a 'start' and an 'end.'  

Example:  
An outlet in your bathroom no longer has power. If any other location in your house has power, you can consider the _system under test_ to start at the breaker box in your house and end at the bathroom outlet. In this scenario, the breaker box obviously has power because your house has power _somewhere_. You would first identify a location roughly halfway between the breaker box and the bathroom outlet, let's say a junction box in the ceiling. When you test the power in that junction box and find that the junction box has power. Since it has power the problem has to exist between that box and the outlet. The power is wired in series and runs through two other outlets before getting to the outlet that doesn't have power. When you check the outlet in the middle, you see that it has no power. Since it doesn't have power, the first outlet to be the next place to check. When you check the first outlet, you discover it has power. So the problem must exist in the line between the first two outlets or a defect in the second outlet itself.

That's the power of the binary search. It becomes more and more valuable as the size and complexity of a system increases. Being able to quickly discard very large areas of a system as "not the problem" can save you considerable time.

### Checkpoints

Checkpoints are very basic troubleshooting mechanisms. In software systems they can be reprented by breakpoints in a debugger, debugging messages logged to a file, or even messages popping up on the screen. In systems like automotive, cable television or computer networks, it could involve probes that read values at a given point in the system and show real-time readings. No matter what system, checkpoints provide a valuable insight into the state of the system at a given point in time and space.  

## Conclusion

Having good troubleshooting skills is both personally empowering and professionally rewarding. Refining these concepts will help you become more proficient at finding the source of problems in almost any system. Skilled individuals can even apply these concepts vicariously through others for systems they know little to nothing about, merely by asking questions of them and pointing them to check the system at various points.  
