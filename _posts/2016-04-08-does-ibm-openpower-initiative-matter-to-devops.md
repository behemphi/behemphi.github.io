---
categories: 
- ibm
- power
date: 2016-04-11
layout: post
tags: 
- blogging
- IBM
- Power
title:  "Does IBM OpenPower Initiative Matter to DevOps"
---

My friend, [Lynn Bender](https://twitter.com/geekaustin), Godfather of [GeekAustin](https://www.linkedin.com/groups/49889), recently asked me if I thought there was a DevOps story in [IBM's OpenPower initiative](http://openpowerfoundation.org). I dug around a bit and came up with these thoughts.

The short answer is, "No, not directly, and only indirectly if the promise of OpenPower is realized."

## Reasoning from a DevOps Perspective

IBM seems to be trying to figure out how to join the technology community without owning it. It's an ongoing process, and I hope they can figure it out. IBM matters, and bringing its brilliance to DevOps practice rather than DevOps marketing would benefit everyone concerned.

The promise of OpenPower, to me however, appears to be more about providing the next decade's worth of hardware performance enhancements. This appears to be a direct play against Intel's stranglehold on today's chip market. 

Intel's rate of innovation has not kept up with [Moore's Law](https://en.wikipedia.org/wiki/Moore%27s_law) in the last two years while its chip prices have crept up relatively quickly over the last three years. This is what most economists would point to as the expected result of a monopoly. Physicists will say that we are reaching limitations around the speed of light in a given media. Laymen like me think it's a perfect storm and the smell of disruption (ozone?) is in the air.

## About Power

The [Power architecture](https://en.wikipedia.org/wiki/IBM_POWER_microprocessors) has some interesting properties that allow it to perform better than `x86` in certain areas. The one that jumps to mind is the computationally intensive encryption algorithms of Perfect Forward Secrecy - loosely speaking, the newest TLS/SSL. Given that security is the 900lb Gorilla right now, such innovation by IBM and its partners will put pressure on Intel and other `x86` manufactorers.   

My memory is also that the Power instruction set is significantly simpler than the bolted-on-then-bolted-on some more of `x86`. This means that the companies in the OpenPower consortium who care to could potentially leverage this for optimizations that are not possible by Intel.  

As strange as it seems, the biggest factor cripppling Power all these years might be something as esoteric as the fact that it was orginially (big-endian)[https://en.wikipedia.org/wiki/Endianness]. Power chips are now capable of running Little Endian as well.  Why is such an obscure fact so import?  Linux is a little-endian operating system.  By supporting little-endian, Power systems can run the entire ecosystem of Linux applications.

## About DevOps

Since DevOps is really about how software is made and delivered to systems, I would say the connection between any chip architecture and the movement or practice of DevOps is tenuous at best. While we DevOpsians would certainly love to deploy to more powerful systems, in the end we deploy to what is available. The only thing that matters to us is a more powerful, less expensive cloud. That is several abstractions away from chip architecture in practice.

I will take it one step farther and assert that DevOps is a hedge against the need for the OpenPower Movement. 

One could say [Moore's Law](https://en.wikipedia.org/wiki/Moore%27s_law) is about to be repealed based on the Constitution of Physics.  Efforts like OpenPower are essential to stave off that coming fact from the perspective of raw compute power.  

In the mean time, those practicing DevOps are building systems to deliver software faster and more reliably than ever before.  As the rate of change in hardware speed necessarily slows, software engineers will find again the need to write high performing code as a business feature.  Languages will evolve to allow for better optimization of CPU time - possibly even at the expense of human time.  

So, while there is no DevOps story in the OpenPower movement.  There is a story about how the DevOps movement will be on the vanguard of enabling human innovation - including our next generation hardware.  

I look forward to living and reading that story.








