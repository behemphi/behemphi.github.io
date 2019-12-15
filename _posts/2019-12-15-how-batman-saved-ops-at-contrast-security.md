The role of "batman" (our Slack handle) is to serve as the primary external interface to other teams for interrupt work. 
The batman handle rotates with the on-call person each week at this time. 

# Overview

The nature of Ops is to be asked for small and medium tasks frequently and sometimes with urgency.
From a team's perspective this can be a drag on project work as people throughout the organization choose 
their favorite Op and make requests at all times. 

To address the problem of all Ops people being "interrupt-able" as a favorite of engineers, 
managers or customer service folks The Cloud Engineering (CloudEng) team will present an anonymous Slack handle 
(@batman) and a process for getting work done. 

# Process
All urgent or unplanned work done by the CloundEng team should be done with a DEVOPS ticket of type Task and a 
label of "batman". This work is triaged daily by The Batman for its urgency and its story point value. The Batman 
then works this queue in priority order. 

*Pro Tip:* When you create such a ticket it is immediately posted in the #operations channel. To further 
escalate the issue, please tag "@batman" in that channel as well. 

When the Batman role changes the new Ops person picks up the backlog where it was left off and triages new work coming in. 

## External Interface to Contrast

Batman is the face of the CloudEng team to the rest of Contrast when a problem has come up. You must be 
an effective and affirmative communicator. 

### Batman is Polite and Effective

The people who come to us have a need that they deem important enough to ask for an interrupt about. They 
are often stressed. They often have a deadline or external pressure driving them.  Batman is calm and empathetic. 
Serve our customers well.  Remember we often need their help, in the form of an interrupt or even a full blown project, 
to get Ops priorities addressed!

### Batman is a Single way for Unplanned Work to get Done
Rather than have many ways for the Ops team to accept work, Batman sits at the head of the ticket 
queue of Ops Requests. A primary role is to triage work to ensure it gets to the right place. We will help our 
customers write good tickets, but it is Batman's responsibility to ensure the work gets documented and done in a 
timely manner. 

If someone from marketing, customer service and engineering are talking about Ops they should all be able to 
tell the same story about how to get urgent work into our queue and when to expect work to get out.  

### Affirmative Communication
Batman should set the expectations of our customers with either:

* "Yes," and then stating when the task will be started
* "No," and why the task will not be accepted and where the customer can go to get help

Set customer expectations and meet them. When you cannot meet them, then tell the customer at the soonest 
possible moment.  When you start a ticket, tag the requester and tell them you'll be done in X hours.  When you 
are done, tag them in the ticket and ask for permission to close the task.  If the person tagged "@batman" in Slack,
then tag them back in the channel when you start and when you finish.  

By being public and "loud" about what you are doing, people know Ops (in the person of you) are working on their behalf. 
If a task takes longer than about 2 hours, follow up on the ticket or in Slack with an update.  The amount of 
good will built up with these simple behaviors will spend well when our Team needs help to move our initiatives forward. 

### Batman's Office Hours
For people from other teams working late on non-critical task, "@Batman" is not available. For example, 
a Dev working on a query tuning issue should not tag Batman outside of their specific working hours. This will 
be determined by Batman's timezone and communicated through the use of the Office hours feature of Outlook.

For critical issues that arise outside of business hours, use the normal incident response procedures. 

### Service Level Goals
For customers of Ops who file an Ops Request ticket, they should expect to see their ticket Triaged and a 
due date placed on it within two business hours. 

For customers of Ops who escalate a need through Slack, they should expect a response within 15 minutes 
during normal business hours. 

Remember that a response is different than a solution. A response is simply that you are aware of the request 
and your status. For example:

* I am out to lunch and will return in 30 minutes. Will that suffice?
* I see your request. I am currently working on another urgent matter for X and expect to be done in 1 hour. Will that suffice?
  * If they say, "No" this might be a good time to ask a team member for help. 
* I see your request and will start in 5 minutes.  Please make sure all the details are in your ticket. Is there anything you would like to explain in person?

## Internal Team Effect and Expectations
Within the CloudEng team, the person behind the Batman mask is the superhero who keeps our current project on track.  
They take the bulk of interrupts by design leaving the remainder of the team to focus on high value project work. 

The primary mission of the Batman is to shield the team from toil (interrupt work) and provide quick turn around to 
other teams who need Ops help to get their own jobs done.

### Internal Expectations
Batman is Too Busy
In a perfect world, the Batman would know how to do everything and would be able to serve the role without 
interrupting other Ops team members. In reality it may be necessary for one or more members to temporarily help out 
if too much work comes in at one time. 

#### Batman is Underutilized

When underutilized, the Batman should never handle project work.  This has a negative affect on deadline performance 
and can be frustrating for folks who end up depending on their work.

Instead the Batman should work the Ops backlog by picking something fun. Our backlog for non-project work should be full 
of tasks meant to reduce toil in our team and toil on other teams. A key mission of the batman is to leave our 
team just a bit better that week by removing a bit of toil. 

#### Batman is Asked to Take on a Task that is too Big

*_Stop!_* Do not be a hero.

Often the requester simply does not understand the scope or complexity of the ask.  They should be told. 
Explain to them that their task may need to be handled with our project work and that they should expect a much longer 
lead time. Ensure you understand the problem they have and not just the solution they are asking for. 
There might be a much easier way.

# Why your Boss Wants you Thumb Twiddling

Remember that the goals of the Batman are to shield the rest of the team from interrupts and provide 
predictable lead times to our external customers. To accomplish this, there is real math that explains the the person 
in this role should not be 100% utilized. That is to say, they should not be busy.  

Here is the math:

LeadTime =  (% utilization * time) / (100% - % utilization),

Here is a concrete example:

Consider Batman is busy 50% of the time in this role.  He might appear to a slacker, but (50% * 1 hour) / (100% - 50%) 
means the lead time on a starting the request is about 1 hour.  Let's call this acceptable.   

Now let's think about Superman and that he is busy 90% of the time in the role. 
At first blush Superman looks like a super hero, but (90% * 1hour) / (100% - 90%) = 9 hours lead time to get a 
task started.  That means someone would have to wait over a day to get something from us.

Clearly Batman is better than Superman when considering that we want to be able to handle things quickly and 
predictably for our external customers.

## Put Another Way
When you are Batman, it is not only "okay" not to be busy, it is necessary. That feels weird. Use the time to 
remove some toil. Study for an AWS exam. Read up on App Sec or play with Hunter 2. Be ready to set that aside when 
new work comes to us via an Ops Request ticket or an escalated tag in Slack. 

Another way to think of it is like this:  Firemen spend most of their time on shift cleaning and doing minor training. 
Fireman are first responders. So is Batman. 

## Put in Context of On Call

On-call sucks. The person who has to stay close to their work space so they are ready to respond for their week and get 
up at night when things are at their worst is the right person to also be in a role that is designed to be less busy 
and pressure filled during the day. 
