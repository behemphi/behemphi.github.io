---
categories: 
- Process
- Team Meetings
- Lean Coffee
date: 2019-12-19
layout: post
tags: 
- Contrast Security
title:  "Asynchronous Team Meetings"
---

# Purpose
The purpose of this article is to serve as a descriptor of the Ops' team meeting process and as a container 
for meeting agendas and minutes.

# The Meeting
The general goal of a team meeting is to come together regularly (once per week) and discuss the most 
pressing topics facing the team with a goal of taking action to eliminate complexity, fix something broken 
or improve something that is not working quite as we'd like it.

It is common during the day to think, "We need to talk about <insert-thing-you-are-working-on>."  It is nearly 
as common that we forget to bring such things up. It is also common, that when viewed at a bit of distance, what 
seemed important turns out not to be. These common experiences are taken into account with an asynchronous 
agenda and the Lean Coffee format. 

## Roles
The following Roles exist in the team for the purpose of the meeting:

* Manager - Responsible for convening the meeting, managing the format and taking/publishing notes.
* Scrum Master - Responsible tracking the Lean Coffee agenda during the meeting and keeping time. 
  Responsible for posting the new Agenda immediately following the meeting.
* Team Members - Responsible for adding agenda items to the agenda throughout the week. 
* Guest - From time to time guests may be invited if they have posted an agenda item and are willing 
  to attend the meeting. Guests follow the same rules as all team members with the exception of our Director 
  and VP who are given priority as a courtesy.

## Aysnc Agenda Management

* The team scrum master will create an agenda on the day following the meeting. 
* Items from the previous agenda are carried forward.
* Any team member, or invited guest, may create an agenda item.
* Agenda Items should be written in the Discussion Topics section. The notes section is "marketing" 
  for collecting votes on a topic important to you.
* Please leave the "Time" box empty. Time is determined by the group on the fly with the meeting format (
  see below)


Example:

| Time | Item | Presenter | Notes |
| ---- | ---- | --------- | ----- |
|      | Train Team on use of the Lean Coffee Meeting System | Boyd Hemphill | In order to conduct a meeting where all voices are heard and the most pressing items are discussed, the Lean Coffee system will be shown to the team and those interested will be taught to run such a meeting. It is possible other meetings intra- and inter-team could be run in this format going forward. |

## Lean Coffee Format

The [Lean Coffee](https://medium.com/agile-outside-the-box/lean-coffee-facilitator-s-guide-d79d9f13d0a9) format allows 
for a structured discussion by peers. The purpose is to ensure the agenda 
is controlled by all stakeholders and that discussions last only as long as necessary. 

Since we cannot sit together at a table every week, we'll simply use a column in the agenda to vote.  Time will 
be kept by a volunteer. 

## Minutes

The Scrum Master will take notes live in Confluence on the Agenda page for that week. Notes are: Brief descriptions 
of decisions, assigned action items, etc.  They are not meant to be a detailed record of the conversation.

# Resources

* [Lean Coffee](https://medium.com/agile-outside-the-box/lean-coffee-facilitator-s-guide-d79d9f13d0a9) - While run in many ways, this is a reasonable facsimile of what we will be doing.
History

# Example Meeting Page

The below is an example of a meeting page after the meeting has occured. 

* Names have been scrubbed
* Note that a single letter in the vote column is from a single human. Each human gets three votes
* We discuss for 5 minutes then vote on continuing for another 5 minutes.
* We discuss in vote order but you can see there was an override by the manager b/c of vacations and holidays. This is _very rare_
  (i.e. once a month maybe)
* You can see we got to six of the items in about 75 minutes. The others will carry forward to the next week.
  
## Date
18 Dec 2019

## Participants

@KM 
@Boyd Hemphill 
@JG 
@JAd 
@DQ 
@CB 

# Goals

* Use this meeting is to discuss problems the team is facing and get solutions in place.
* Go over action items from the previous weeks team meeting.
* Scrum master  - go over previous meetings action items.


## Discussion topics
| Discussion Order | Votes - 3 each | Item | Presenter | Notes |
| ---------------- | -------------- | ---- | --------- | ----- |
| 2nd | scdb | Batman meeting attendance | KM | The role of our team and batman is to be service oriented to our internal and external customers. Having a day full of meetings, can hinder our customer service to our customers. I suggest we identify if Batman is allowed to skip certain meetings and what meetings shall be required, or if this process should be adopted. |
| 6th | j | Deprecation and Error messages in Ansible runs | @Boyd Hemphill | We get a crap ton of Ansible deprecation notices (and a few errors). This is way too much noise for the signal. Should we address this? |
| | a | Alphabetical is the only order | @Boyd Hemphill | For the formatting of all lists of things (e.g. variables, stack inputs et al) things should be placed in strict alphabetical order. I further propose that when we touch a file we ask if this is the case and make reasonable changes to make it so without obfuscating the actual change we are making. |
| 3rd | sjcd | CloudFormation Parallel Work | KM | We’re growing as a team and it’s already become a blocking issue for people to work on environments in parallel, specifically on production roll-outs. Let’s discuss the issue and potential solutions to bridge the gap between today and the future. |
| 5th | sa | Research Tasks | KM | Since we have new team members, questions have come up around research tasks.  Therefore, let’s ask ourselves, “what do we expect out of research tasks?” |
| | | Ready To Work & Backlog | FS | How do we prioritize the Backlog & the Ready to Work areas of our board? Should Ready to Work tickets be assigned? Is the Ready to Work section for project tickets? |
| 1  (boyd Override) | b | Holiday Vacation / On-Call | KM | Just ensure everyone’s aware of vacations coming up.
| | j | Zoom v. Highfive | @Boyd Hemphill | Sounds like we’ve adopted Zoom.  I’d like to make this formal here then announce this to the org formally so folks know we’ve done so.  Remember that we will need to use highfive for the time being with our peers. |  
| | j | Team Calendar | KM | We need a way for general notices for the team, like vacation. Proposal is to have a team calendar where we can place all this type of information, so our private calendars are not polluted. |
| 4th | bacd | Review Done column in standup | @Jeff Adams | Look at done column during standup for important thingys |

# Previous Action item/s:

* [x] @FS/@CB update Bitbucket transitions
* [x] @JA Move onboarding pieces from previous team charter to own document
* [ ] Team: Consider how/when to implement artifacts for distribution
* [x] @FS schedule a discussion about how we manage and organize the backlog
* [x] @Boyd Hemphill If MS in Ireland accepts, we don't need senior level AWS skills. Intro>mid okay, as long as they have experience with on prem, support, etc. 

# Action item/s:

* [x] @Boyd Hemphill  - create a coding standard document. Have @Casey Buto review it. Concerning variable hierarchy topic. - ticket created DEVOPS-4671: Start Coding Standard Doc
* [x] @FS to check with TS crew to see if there will be a release during the freeze
* [ ] @Boyd Hemphill to write into batman the meeting attendance guidelines.
* [ ] @KM will write up a research task to look into building a pipeline for production releases of our current code base. And another for handling parallel dev efforts. 
* [ ] @FS to reduce time for the tix in done to live on the board to 5 days. 
* [ ] @Boyd Hemphill to update doc on how standup works (kanban and use of the done column)
* [ ] @Boyd Hemphill Alter the story point doc to add info on research task in terms of a time box.  Write up the need for output from the research (confluence doc, tix, or it comes out as a no-op). A research task should include why it is happening and what. 

