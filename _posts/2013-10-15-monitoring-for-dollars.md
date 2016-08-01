---
categories: 
- Geekery
date: 2013-10-15
layout: post
tags: 
- devops 
- monitoring
- business value
title:  "Monitoring for Dollars"
---


When you walk into a new job as the guy people come to when their software system breaks, what do you want to know first? 

In the weeks leading up to my first day on the job, I was called by our CTO John Cunningham several times.  He was sorely disappointed that our customers kept notifying us about problems they were finding in the software systems we had built for them.  The specific concern was downtime.  What was going on?  Why so many failures?  Why aren't we finding them?

The first order of business was to get [situational awareness](http://en.wikipedia.org/wiki/Situation_awareness).  While the situation of a customers non-critical system going down is far from the concerns of first responders, the concept of situational awareness is very useful.  

When a customer system went down, we huddled around New Relic if it happened to be on that machine.  It quickly became apparent that nobody really understood how to use the tool to solve a problem.  We had an expensive alarm clock and not much more.

Fundamentally, we had no overarching vision for monitoring, alerting and forensic investigation.  Further digging after I arrived led to the discovery that we had 10 New Relic licenses for the 100+ machines Devops was managing.  A further 50 machines hosting our customers are managed by our separate IT group and invisible to us.  They are using Zabbix.

It is worth pointing out that New Relic is an amazing tool.  As a production monitoring system though, I am of the opinion that it is overpriced and complicated to use.  We have over 100 machines representing over 35 different systems to monitor.  The depth of New Relic was not useful.  The cost of New Relic was too high. We needed quick and dirty. 

There are a number of good solutions in the the space of SaaS monitoring:

* [CopperEgg](http://www.copperegg.com/)
* [DataDog](http://www.datadoghq.com/)
* [Stackdrive](http://www.stackdriver.com/)

At [Feedmagnet](www.feedmagnet.com) we landed on CopperEgg as an inexpensive way to easily monitor 150+ machines.  It is a tool I am very familiar with and has several key features that lend itself to quickly gaining situational awareness.

* Skill Set: Someone on the team knows how to use it.
* Inexpensive: at $500 per month we could monitor all of our machines, rather than the 10 we got with New Relic
* Value: For that same $500 per month we could put HTTP probes on all websites thus also gaining a narrow but useful picture of the end user's experience
* Flexibility: It is possible to write custom metrics in Ruby to measure key application performance metrics over time.
* Beautify Simplicity:  When the team looked at the data one of the guys said, "Wow, it is so easy to see what is going on".  No training necessary.  

We were on our way to gaining situational awareness around the production operation of our customer systems.

# Monitoring Wins

Monitoring is great.  Monitoring with alerting is better.  *Situational Awareness* is the end game.  [The Goal](http://en.wikipedia.org/wiki/The_Goal_(novel)) of a business is to make money.  Focused monitoring at the right price provides 10x or better return on investment.  

## Catching Problems Before Customers

In our first troubleshooting session we learned was that [Amazon RDS](http://en.wikipedia.org/wiki/Amazon_RDS) does not allow us to see what is happening on the host machine.  We couldn't tell what was happening because we could not put an agent on the machine.

A little bit of development work and we had a custom metric monitoring the RDS slow log.  It quickly became apparent that batch jobs run on a cadence where the issue.  A developer had the problem squared away for this key customer in a couple of days.  

This was a big account for us.  It's great that customer satisfaction improved in that we are catching any problems before they do.  Indeed, in the first six weeks of getting all machines monitored we caught over a dozen critical issues with zero customer reports.

It is tough to put a dollar value on this, so we simply won't.  We don't need to.  Read on ...

## Catching Patterns

A well considered alert strategy allows for a strong team to record patterns over time.  There is more detail below on this.  With a pattern of alerts, the 30 days of data kept by CopperEgg allow us to look into application performance without the urgency of a full blow production issue.  

We have not yet been able to capitalize on this feature of our monitoring setup.  It is certainly in the list of stories to accomplish as our Devops practice continues to mature.

Top developers have tools that tell them how their code performs.  These tools are called code profilers.  In the hands of a good developer, running tests on their code with a profiler provides a look at how each object graph performs.  With little work and some intuition, a developer can fix the easy and obvious performance issues.  

Monitoring provides the same look at a production system. Over time it profiles how the system, as a whole, performs.  With this information the DevOps team looks for patterns and tunes out trouble before it is seen by the customer.  In the worst cases we can open tickets for developers to work on in their sprints if identified issues require code changes.  

Consider that last statement.  We can take a problem that traditionally would have required a "hot fix"  and turn it in to scheduled work.  [Eliyahu Goldratt](http://en.wikipedia.org/wiki/Eliyahu_M._Goldratt) is smiling somewhere.  If you don't know why, please consider reading [The Goal](http://en.wikipedia.org/wiki/The_Goal_(novel)).  It is also a great audio book.

It is, again, tough to put an actual dollar value on this.  So, again, we simply won't.  We don't need to.  Read on ...

## Over-provisioning

Understanding hosting costs is a huge problem for W2O at this time.  We have over a dozen accounts across Rackspace Cloud and Amazon Web Services (AWS) with little rhyme or reason to how customers and their particular projects are assigned to them.

Now that we have 30 days of history on each of our machines and have likely experienced all possible batch jobs, we can conduct an audit with the following goals:

* Organize our hosting accounts in such a way that our accounts payable team can bill our customers for their usage. (Remember this is a business)
* As part of the audit, look at monitoring history to determine if any systems are over provisioned.

Provisioning in DevOps is the act of assigning a host of a certain size and power to the topology of a system architecture.  In its simplest from consider a single machine containing a LAMP stack.  If we put this on a machine with 2 cores, 2 GB of memory and 40 GB of drive space, we have provisioned the system that uses the LAMP stack on this machine.  

If the machine exhibits symptoms such as high load average, disk filling or swapping, we know we have under-provisioned the machine.  In the [IaaS](http://en.wikipedia.org/wiki/Cloud_computing#Infrastructure_as_a_service_.28IaaS.29) cloud world there is an awesome button that can be pushed that will grow a machine to a new size in a few minutes.  

Sadly, the corollary is often overlooked.  A machine can just as easily be shrunk if it is known to too powerful.   In the case of W2O, without situational awareness, the common practice was to put customer applications on a large enough machine that there is never a problem.  

CopperEgg easily showed us that most of our customers are over provisioned.  Investigating our first AWS account we learned that we can save over 60% (yes, really) by reprovisioning the majority of our customers on smaller instances.  Of course, if we make a mistake, the monitoring solution will alert us so we can fix our mistake.  

To put a finer point on it, our decisions to resize customer systems are based on data.  The [scientific method](http://www.sciencebuddies.org/science-fair-projects/project_scientific_method.shtml) might look something like this:

* Question: Can we save money on hosting by shrinking the size of some customer instances?
* Background: Each customer system we have looked at in detail appears to be over provisioned. Monitoring shows us very few systems using much memory, CPU or disk.
* Hypothesis: We can save 40% on our hosting cost by downsizing over-provisioned customers.
* Experiment: We have conducted a thought experiment via an audit of all systems in a single account.
* Analyze: We determined that the likely saving is 60%.
* Communicate:  We have communicated to our CTO that we will be resourcing this effort, thus invoking "peer" review.

An observant [BOFH](http://en.wikipedia.org/wiki/Bastard_Operator_From_Hell) will quickly point out this is all ivory tower science.  However, good implementation must stem from sound and consistent reasoning (philosophy).  With the backing of our CTO and the support of the development teams, DevOps embarked on shrinking two or three of these systems each week.

With just this one account, we will save nearly the salary of a full time employee.  By the time we are done, we will have saved close to the salary and benefits of two of our team.  

Surprising?  Not really if you consider that problems arise:

* without a visionary DevOps practice that includes a deeply considered monitoring practice.
* absent financial controls, the developers running the systems are incentivized to:
  * Make a system large enough that it could not possibly experience issues (over provision).
  * Grow the hardware when problems occur rather than investigating the system.

### Developers Are Your [Sheldon Coooper](http://en.wikipedia.org/wiki/Sheldon_Cooper)

How does your company incentive your developers? It is likely it has nothing to do with:

* Identifying inefficient code and fixing it in the name of cost savings.   
* Participating in the identification and root cause analysis of production system failures
* Thinking about the entire software life cycle.

It like has far more to do with:

* Feature delivery
* Bug fixing
* Writing automated tests
* Implementing the latest buzzword technology 

Developers are not the enemy.  They simply have a completely different reality contained within the same lifecycle as the operations team.  DevOps, as a movement, seeks to better align these realities.

Looked at through the lens of *The Goal* or *The Phoenix Project*.  Developer realities are an example of achieving a local optima at the expense of the success of the entire system.  

Aligning the realities of each team involved in the software life cycle is a future adventure.  Look for it in the months to come.

# The Monitoring Practice

Practice was a word bandied about at W2O Digital. I asked people from the CTO on down for a definition, but failed to get a fully aligned  understanding.

My own evolving definition is metaphorically aligned with the notion of a medical or law practice.  In this way, I believe:

**Monitoring Practice**

The actions taken by DevOps practioners to diagnose, treat and prevent production system performance issues.

From this definition, implementation follows:

## Installing Agents

Agents are generally very small clients installed on the machine to be monitored.  When choosing a monitoring solution keep these key elements of the agent in mind:

* Heavy agents cause the [observer effect](http://en.wikipedia.org/wiki/Observer_effect)
* Avoid a Java based agent if you are not already running the JVM for the same reason
* Consider how easy it is install and configure the agent.

The last is especially important if there is no current Monitoring Practice in your organization.  Don't waste time trying to get the solution just right.  Pick the easiest thing to be found and build the practice.  Keep these things in mind:

* The choice will likely be wrong in the long term.  It's OK.  
* Switching out the tool will become much easier as the practice matures. 
* Choosing the perfect tool will become more evident as the practice matures.

### Monitoring vs. Profiling 

When using an agent for a profiling tool such as New Relic be sure to understand the impact of the agent on production performance.  Most likely you do not need the full agent in production.  Production monitoring is, in general, not about knowing how many times a code block is executed.

A better use of such tools is in the QA environment.  In general the QA environment is smaller in terms of machine count, thus better leveraging these expensive tools in terms of cost.  The additional benefit is that a high quality QA staff can run tests and report issues to the developers as another check and balance.

## Probes

End user monitoring is great, but it is very time consuming.  If you are unfamiliar with this concept, [this blog post](http://www.real-user-monitoring.com/the-complete-list-of-end-user-experience-monitoring-tools/) is an excellent resource.

In the spirt of Just Enough Monitoring, a quick and dirty substitute is http probes.  

CopperEgg easily allowed for the creation of one or more probes with many different protocols.  Pointing a few of these probes at login pages, key images, and key javascript files then setting alerts can give you a better view of what the user experience is like _from the users perspective_.  The evaluation of "easy" should be viewed thought the lens of:

* Does the GUI of the tool make it easy and intuitive to create a probe
* Is there a Chef (Puppet, Salt et al) recipe for the auto-magic creation of them when we tackle our configuration management needs.

Again, this is not a substitute for a product like Gomez or Alert Site, but it is very easy to do and, more importantly, very easy to automate.

## Alerting

All the monitoring in the world does no good if nobody is looking.  Having someone watch is simply not practical. Too many alerts on a DevOps team member's phone means they end up getting ignored (signal to noise ratio is too low).  

It is critical to strike a balance in the collection of information on key thresholds.  To this end our monitoring practice sees alerts serving three general purposes:

* Warn:  Low threshold alerts meant to build up over time to form a picture of long term issues. No trouble ticket entered until a pattern is understood.
* High: Mid level threshold meant to warn of a problem that could quickly escalate.  Trouble tickets on these opened daily and worked within three to five days
* Critical: Production system down or user experience effected.  Alert to Devops team email.  Team member to stop all current efforts, open a trouble ticket and drive issue to resolution.  

Many system admins view an alert page as something to keep clean.  When every alert on a page is something that means a phone call to an engineer, it is easy to understand this view.  Worse, if there is an accountability system incentivizing this behavior, the result is only a trickle of return on the monitoring practice.

The problem here is that some issues develop overtime.  Simply sitting down to look through the history of individual machines is laudable but inefficient when hundreds of them exist.  Instead consider three levels of alert:

### Warning

Warnings have low alert thresholds.  

* The goal of warnings is to fill the alert page with common problems.
* Warnings should _never_ go to email, phone, pager.  Too much noise

On a cadence, usually with the regular release of the software on the systems exhibiting warnings, these should be evaluated and recorded as work for the Devops or Dev team.  

Warnings should be deleted when a new release is moved to production _even if there is no direct work on the recorded issues_.  This is because the production support team will generally not have complete visibility into the changes or the ramifications of those changes.  If the problem was fixed, even indirectly, it is no longer a problem.

Examples:
* On a modern system (even most data stores), a drive becoming 50% full is likely something to watch.
* CPU usage over 80%.  This can happen pretty regularly.  Does it correlate to other issues such as memory spikes, network traffic or batch jobs?  The alert system is telling us to go and gain more situational awareness in this are as time permits.
* Swap grows over 20% of allotted space.  Is one of our custom daemons leaking memory?
 
 These are things that happen.  Sometimes they even happen for good reason.  We don't way a call to go out for help, but we do want a place to see that a machine is exhibiting this behavior when we take our once per week look for potential issues.

### High

High alerts have thresholds that are a cause for concern but not immediate action.

* The goal of High alerts is to recognize situations that are likely to result in a a critical alert if left alone. 
* High alerts should _never_ go to email, phone, pager.  Too much noise

On a cadence, in the morning, high alerts should have a ticket opened and a due date of no more than three days before they are investigated.  It may be that more info or time is needed, but the issue should be acknowledged and more should be known than when the ticket was opened.

As a result of this a key feature of the ticketing system is effective full text search so that a recurring issue can be reopened or appended to rather than having many tickets for the same issue.

As with warnings, all high alerts relative to a system should be deleted when a release occurs.  Tickets should be closed.

Examples
* Memory use over 90% on a machine without a data store (load balancer, web server)
* Disk usage above 80%

The number of high alerts should be significantly lower than the warning count.  It will take time to school an alert system to this sort of thing.  That schooling will also occur for each kind of system.  

### Critical

Critical alerts have thresholds that are cause to immediately stop whatever work is being done and focus on this issue.  

* The goal of critical alerts is to get the attention of the operations team and get the alert to the right engineer as soon as possible.
* Critical alerts should be set to occur before the customer would notice the issue.  (i.e. don't wait for 100% disk use on the database server)
* Critical alerts should _always_ go out over email, phone, pager.  
* Critical alerts should _never_ be noise, only signal. 

When a critical alert occurs the BOFH who first catches the issue should notify their team in the appropriate chat room that they have the issue, open a ticket and immediately communicate that ticket to the team.

If the BOFH is not familiar with the system an knows who is, they should communicate directly with this person and move ownership if possible.  At this point the issue should be actively worked to conclusion.  BOFH will have access to a playbook for the system containing the names and contact information of the developers, and project manager of the system. 

It is important to note that even critical issues are not worthy of waking up a human, even a BOFH, in the middle of the night unless the SLA of the system requires this behavior.

## Escalation

At W2O, we have no 24/7 operational expectations currently.  Staffing a 24/7 escalation chain with only 4 people on the Devops team is a failure situation if critical issues are common.  In this way we have no formal escalation process and the development of one is not on the radar.

In the name of Just Enough *X* where *X* is escalation in this case, we also don't have enough visibility from past incident management.  So we will simply work normal business hours and gain the experience necessary to develop and staff a reasonable escalation chain.  

In the time between now and when 24/7 support is required, we will us our monitoring practice to drive the number of critical problems to zero.  We will continue to use the practice to catch building issues so they never reach the critical stage.  It will not be perfect, but it will:

* minimize the number of incidents
* motivate the team to catch problems before they become critical.  In other words, provide a natural incentive to follow the practice.

Additionally this is _Dev_ops.  Developers share in the responsibility for the stability of the system.  If they do not, then they will not be incentivized to fix operational issues.  Remember a developer's pressures come mostly from the business.  
 
