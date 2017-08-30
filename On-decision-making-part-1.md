_Note: this is a draft dated from 4-5 May 2017. I was going to finish it before publishing, but got busy with other issues (including ecosystem security). As of today, this has been lying under the hood for too long, so I'm publishing it as-is._

_Do not consider this a final proposal of some sort, this version was intended to be just a draft to outline some things and gather initial feedback._

_Nothing, except this note at the very top, has changed in this draft since 5 May 2017, some things might have changed a bit since then._

_I now (August 2017) resumed working on this, but anything further will be published in a separate note._

_All opinions expressed are personal._

# On the Node.js CTC decision-making process :sparkles:

There are some rough edges in the decision-making process, e.g.: GitHub being a non-ideal place for making decisions when there is a significant disagreement. Also the potential (aka expected) problems with getting a quorum with the growth of the CTC, and other stuff.

This document tries to outline all those and what we could do with that.

The options list is not extensive, there are just a few mentioned here.
It doesn't mean that I'm personally against other options — most likely I just didn't think of them.
If anyone has better ideas — any suggestions are welcomed (including an argumented «everything is good, don't touch anything»). :smile:

### Basic use-cases for involving CTC
* Require a CTC member review.
  Example: semver-major PRs.
* Inform CTC by pinging them, so that they are less likely to miss it and will voice their concerns, if needed.
  Example: a PR that could in theory be controversial, but does not have a very high probability of that (i.e. no one raised concerns yet).
* Require a solution with a >50% CTC consent, either by directly calling a vote or by requesting a discussion first.
  Example: concerns raised, someone expressed a negative vote, and there are people who still want to pursue that.
* Require a discussion by the CTC.
  Not exactly sure how this differs from the above, but this is what is happening in many cases.

### Rules
 * Consensus-seeking
 * Either a consensus of everyone involved (i.e. no objections for some resolution), or a vote for some resoltion with >50% of non-abstaining CTC members.

### Controversial topics
 * Buffer API
 * ES modules
 * Some private discussions
 * Promise-based API
 * Naming various API
 * Some other stuff

### Problems
 * Options and arguments have to be repeated over and over again.
    This both makes the thread hard to read and makes people who voice their opinions over and over again tired of that.
    In the worst case, we just have everyone repeating their concerns again and again and be unhappy with solutions others proposed.
    In fact, that does move us forward, but that is happening slower than it could be.
 * With the growth of the CTC, it may be hard to reach consensus, with people not being able to be present on all the calls and not commenting on GitHub for some reason.
 * Some questions that we expect to be controversial but that no one voiced any concerns against are passing too quicky.
   Everyone performs their own quick estimation, and if no one has any serious concerns at that time — the question is passed. In fact, we could have a blind spot for negative consequences of some of those.
 * _(minor)_ Technically, it's possible to bring in an issue for a resolution/vote almost immediately before a CTC meeting and get a resolution there either from consensus of present members or a vote. The problem here is that if this goes really quickly, some input from non-present CTC members could be missing, and if they were present — opinions of other memebers could have changed through a regular discussion.
   I think that this is minor, as we don't actually jump to voting without a prior discussion.
   That said, we _could_ specify a time slot that an issue should have been labeled as `ctc-review` before being brought to the meeting (or a similar way to inform CTC members in advance for private topics).
 * ~`ctc-agenda` rules assume that it could be assigned only when the consensus-seeking process among CTC memebrs have previously failed. It doesn't look that that is accurately followed. Most likely it's a documentation issue. See [#11325](https://github.com/nodejs/node/issues/11325).~ _Upd 2017-10-30: this has been fixed._

### What should be noted when changing anything
  * In most cases, the established process works good — that should not be broken.
  * GitHub is a good place for discussions until those start being exceedingly huge.
    For most cases, it's fine, and we shouldn't move most of the discussions elsewhere — everyone here is familiar with GitHub and hence it's easier to gather new opinions here.
  * Calls are great at collecting various data/concerns from all the participants and making sure that other participants are aware of that.
  * A quick estimation on whether the question is controversial or not is expected to give good results — so we could rely on that.
  * The rules are quite good, and the process should be kept consensus-seeking — that is a great thing. The intention here is to only improve the details of that process.
  * We shouldn't get more voting — voting means that the consensus-seeking process was aborted due to the lack of time.
    A consensus is [always possible](https://en.wikipedia.org/wiki/Aumann%27s_agreement_theorem), and one of the purposes behind trying to improve the process is to save time spent on the decision-making for complex issues, meaning that we will be able to move further towards a consensus and will get better resolutions. We should aim to less voting and more consensus.

### Effectiveness
 * On the TSC repo, an opinion was voiced that about half of the activity is dedicated to meta
 * The call meetings are still required, as that's the quickest way to discuss things, and those are productive for decision-making on controversial questions
 * Questions that have no concerns and are not expected to be controversial probably shouldn't be brought to the meeting.
 * Some questions are quickly moved back to GitHub. Perhaps that needs a slight process change, if that happens often.

### What could be done?
  * Once a certian threshold is reached (not necessary formally defined) — list all the opinions somewhere, giving everyone an opportinity to evaluate that (e.g. a quick -1/0/+1) with an explanation of their vote
    * List all known options
    * Allow any member to add new options at any time, even after the current ones were estimated
    * Collect votes -1/0/+1 for every option. Those should contain an explanation why (a minimal would do).
    * Once/in case of one for the reasons in the explanation behind the vote is proven to be false — the person should reevaluate it. _(That happens, and that is a part of the normal process)._
    * Seek for the uncontroversial or the best option.
    * Uncontroversial option should probaby require a devils advocate.
  * Try strategies for resolving disagreement (double crux?), when the regular discussion failed or is taking too long?
  * Devils advocate for questions that have no concerns voiced, but were expected to be controversial?
  * Decide what to do with abstains — e.g. two weeks without a notice could count as automatic abstain. It could be recorded differently, but counted as an abstain for the votes formula. As it limits us and slows down the process — we should discourage people defaulting to that somehow — e.g. we could count the number of automatic abstains per several months, perhaps, and use that a data point? _Note: **this** topic is expected to be controversial by me._ It shouldn't opress the people who are e.g. absent for a while, though.
  * Give people an ability to opt-out for a while (e.g. sick leave or a vacation), effectively abstaining their votes on everyting for that period unless noted otherwise. This is basically a soft version of a temporal removal of a CTC memeber — it does have lower negative effects, though, and less churn.
  * About the discussions quickly being moved back to GitHub, there are several things we could do (listing without advocating for any of those atm, just exploring):
    * That could be done async prior to the meeting — that will save a discussion slot. That has drawbacks and will require a process change.
    * We could add e.g. an `delayed-ctc-agenda` tag that would turn into a `ctc-agenda` after several days if not revoked by the person who added, and will have different rules allowing everyone assigning that (the ctc-agenda has strict rules that are not followed btw) — that could solve two problems.
    * Alternatively — allow all Collabolators to use `ctc-agenda` at will, but have a minimal delay of two days between the labelling and the meeting, (i.e. defer to the next meeting if the label has been just added), and ask for a confirmation if the discussion is still wanted before the actual meeting.

### Specific actions

This mostly duplicates the previous section, but is a bit more specific.

 1. Automated abstaining of everyone who did not comment after a time period (e.g. two weeks, the exact time could be discussed separately) after being pinged personally or as the CTC group
 2. Opt-in to abstaining by default for a time period (vacation or sick leave)
 3. Investigate methods to collect and record options and opinions (Google Spreadsheets)?
 4. Investigate methods of resolving disagreement — would something like that be useful here?
 5. Differentiate between issues that are expected to be controversial and not, treat them differently — the former could need a devils advocate, while the latter could be brought back to GitHub even before the meeting happens.
 6. ~Discuss what to do with CTC-related labels.~ _Upd 2017-10-30: not needed anymore._
