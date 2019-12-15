---
categories: 
- Process
- Robin
- Team Health
date: 2019-12-15
layout: post
tags: 
- Cloud Austin
- Austin DevOps
- Contrast Security
- robin
title:  "How Robin Keeps Batman Idle"
---

The role of "Robin" (Slack handle for daily work team) is to serve as the primary executor of regular maintenance 
to our systems. Secondarily, Robin is the external interface to other teams for planned tasks requiring coordination. 

Where "Batman" shows up in an emergency for the unplanned things that cause planned work not to get done, Robin's mission is to ensure the many tasks that must get done to keep our infrastructure and systems healthy get done in a predictable way.

# Overview 
It is the nature of an Ops team to do maintenance on infrastructure and systems. From a team's perspective, 
this can become interrupt work as what can be scheduled (e.g. the rotation of keys) ends up a small fire b/c they 
expired and the auditor is coming. This is not healthy for the business, nor is it healthy for the humans on the 
team as it amounts to gratuitous stress.

To address the problem of "schedulable" work falling into an unplanned and stressful state, the Cloud Engineering 
(CloudEng) team will present an anonymous Slack handle (@robin) and a process for getting important tasks accomplished 
in a non-urgent fashion whenever possible. 

# Caveat

The notion of "scheduling" is not well defined at this time. It is freely admitted at the time of this writing (2019-10-29) 
that the problem of scheduling tasks in a way that is public, transparent and effective is unsolved.

Rather than coming up with some sort of gold-plated answer we will continuously experiment with ways to do this.

For now, envision a simple queue and a burden on the requester to specify when they need something done. 

## Some Helpful Metaphors

It is generarlly believed that operations teams are cost centers and filled with angry wizards who perform dark magic at
night. As you are speaking with folks external to the team, these metaphors can help them understand that we have a 
direction and crucial impact on their key performance indicators (KPIs).

### Maintenance
Consider a company that delivers pizza with a fleet of vehicles. It is reasonable to assume that a leading indicator 
for delivery time is the execution regular oil changes every 3000 miles. If this maintenance is not done, vehicles 
will randomly break down and cause late deliveries. It is also easy to see that there will be further unplanned expenses 
in towing and repair. It is less easy to see, but evident, that the stress of the driver and the store manager will 
also increase. 

Oil changes are a form of regular maintenance not just for the car but for the entire system that is delivering the pizza. 
Oil changes are only one example of such a maintenance task. Tire replacement is another. Oven calibration is an 
example of something not directly related to deliveries but affects the amount of rework if pizzas come out undercooked. 

In the same way there are many regular maintenance tasks for a software platform. This can be upgrading an operating system, 
running a test of a business continuity process or improving based on the findings of a penetration test. 

# The Work Robin Does

Robin does maintenance work and small continuous improvement tasks. 

Robin does not take external interrupts except for scheduling requests. That is to say, if something is urgent, then 
Batman should handle it. 

Robin does not participate in project work that is either business or IT facing.  

If a single task can be scheduled, then it should be scheduled and Robin is the role to do it. 

# Process

Maintenance and small continuous improvement efforts done by CloudEng should be done with a DEVOPS ticket of type 
Task and a label of "robin". This work is triaged daily by Robin, it is story pointed and scheduled on a calendars.

*TODO:* We need some sort of way within Jira to schedule work so that it is externally visible. 

*Pro Tip:* You might be a team like Contrast Labs that has run a penetration test and wants to review and schedule 
the findings. You might be compliance. If you'd like Robin's attention, please  use the "@robin" tag in the Slack channel #operations.

## External Interface to Contrast
Robin is the face of the CloudEng team to the rest of Contrast for getting important tasks done. Our partners 
in Security, Compliance, Customer Success and in Engineering need us to do work for them. The more we can plan 
this work out ahead of time, the less stress there is for all concerned and the more performant the entire value 
delivery system for our customers. 

### Robin is Polite and Timely

"Lack of planning on your part does not constitute an emergency on our part."

The people who come to us have important jobs to do and deadlines to meet. We expect them to ask us with the idea we 
have current work we are delivering. It is reasonable then that they expect us to give them predictable lead times on 
their tasks. 

Robin is calm and empathetic. Robin serves our customers by putting their requests at the bottom of the queue, 
doing a calculation and given them an expected date for the work to be complete. (aspirational)

*TODO:* Need to get this calculation figured out.

### Robin is the Single Way for Daily Work Tasks  to get Done

When a customer of CloudEng needs work done that can be planned, Robin is the one and only controller of the queue. 
In this way there is a consistent message to our customers about what to expect from the person likely to do the work. 
Robin also helps our customers write effective DevOps tickets. Robin is responsible for working with the customer 
to define the acceptance criteria of a task explicitly.

Robin must politely, but firmly reject work when:

* It is urgent. 
  * Direct the requestor to Batman immediately, or facilitate the transfer if the requestor is stressed (i.e. be empathetic)
* It is a project, then you should decide to
  * Direct the requestor to the Product team to have the request placed on the CloudEng roadmap
  * Direct the requestor to the CloudEng team lead

### Robin's First Task is Always Scheduling and Affirmative Communication

Robin should set the expectations of our customers with either:

* "Yes," and then stating when the task will be started
* "No," and why the task will not be accepted and where the customer can go to get help

Each morning, Robin should take a look at the backlog of new "robin" labeled tasks in Jira. These tasks 
should be sized (story pointed) and scheduled (placed in the order they will be worked). A comment should 
be placed into each ticket stating when the requester can expect the work to be started and finished. (aspirational)

In the event a requester says they need something before Robin can get to it, Robin is expected to use their best 
judgement in placing them in the queue. It is necessary then to notify all other people behind them in the queue 
their tasks will be displaced by a certain amount of time.  

*Pro Tip:*  While it will happen the need to insert a task anywhere but the end of the queue should be extremely 
rare. Robin should involve the CloudEng team lead if he/she feels the system is being gamed.

### Robin's Office Hours

Robin works normal business hours for his or her timezone. If something is urgent it should go to Batman.

Robin will occasionally get behind in time commitments. This should be communicated early and often to the customers 
in the queue. No "heroics" should take place to preserve the promises. Rather Robin must let the CloudEng team lead 
know about capacity problems ahead of time.

In most cases our customers will be happy to adjust if slipping a due date is a rare event. 

## Internal Team Effect and Expectations

The primary mission of Robin is to shield our systems from accumulating technical debt by deferring necessary 
maintenance for interrupts and project work. 

Within the CloudEng team, the person behind the Robin mask is just a normal human who ensures maintenance 
is getting done and small improvements are happening. He/She schedules the maintenance tasks, write the 
requirements and executes on them.

### Internal Expectations

#### Robin is Too Busy

In a perfect world Robin would triage work in the morning and go home with an empty queue.  In practice, 
we know there will be times that work piles up and deadlines begin to press.  Robin may, in these cases, 
ask for help from Batman. Batman should only work tasks that important and at risk on their deadline. 

If too many tasks make it to Batman (i.e. they have both the "robin" and "batman" labels), this should trigger a 
discussion in the team meeting about adding capacity to the Robin role. 

#### Robin is Underutilized

Ideally Robin should have free time because maintenance is going according to plan. In these cases, the backlog of 
small tasks to remove toil should be pull from. Robin is free to pull from this backlog in any order he/she sees fit. 
It is suggested this be discussed in daily standup however to identify high value items that are also of interest to Robin.

#### Robin is Asked to Take on a Task that is Too Big

*_Stop!_* You are not a superhero. 

First understand the problem they are trying to solve and not just the solution they are proposing. You might be able to 
schedule something easy that meets most of their need. 

Often the requester does not understand the scope of what they are being asked for. They should be told the scope 
and Robin must explain that their task is actually a project and they will need to work with 
the Product Team or CloudEng lead. 

# Continuous Improvement in the Wild

Robin is the CloudEng teams primary way of practicing continuous improvement. You are the unsung hero of our team who will:

* Identify toil and remove it from the system (Removing unnecessary work from the system)
* Identify repetitive tasks and provide automated interfaces for external requesters to self service. (e.g. DB Insights)
* Raise the alarm when our maintenance requirements outstrip our capacity to service them.
  * You are the first line of defense in preventing the accumulation of technical debt. 

## Put Another Way

When you are Robin you are constantly asking yourself the questions,

* "How can I avoid doing this again?"
* "How can I make this less error prone?"
* "How can I better serve the people who ask for this task?"
* "What can I do for the project teams to make their lives easier?"
* "What did I see Batman repetitively doing that needs to be addressed?"
* "How can I improve that runbook?"
* "How can I tune that alarm?"

Another way to think of this: Firemen spend most of their time on shift cleaning the equipment so they can detect 
defects before they are used to save lives (including their own). Fireman also do and redo training exercises on a 
schedule to ensure they are prepared as a team and as individuals. Fireman are first responders, so is CloudEng. 

## Put in the Context of On Call
On-call sucks. Batman is the hero on call. Robin is the first person Batman will ask if he/she is overwhelmed, 
needs a break or becomes ill. 
