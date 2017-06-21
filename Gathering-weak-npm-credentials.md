# Gathering weak npm credentials

In this post, I speak about three ways of gathering credentials — bruteforce attack, known accounts leaks from other sources (not npm), and npm credentials leaks on GitHub (and other places). _The last one was already covered in the [previous post](Do-not-underestimate-credentials-leaks.md), but it's still a valid source nowadays nevertheless._

Also check out the npm, Inc [blog post](http://blog.npmjs.org/post/161515829950/credentials-resets) about this, if you haven't seen it already.

## Warning — if your password was revoked by npm recently, read this

This is not a false alarm — your password being revoked basically means that I was able to obtain it by some of the means described in this note (though neither of those involve npm directly). Basically any other person with an internet access (including malicious players) can also do that.

**If you are still using that revoked password anywhere — change it everywhere.**

If your token/password was revoked, it means that at least one of these cases happened:
1. You packaged your npm credentials inside an npm package — in this case, npm now revokes your credentials automatically.
2. You published your token/password online yourself, e.g. uploaded it to a public GitHub repo, saved it in the CI logs, pasted to GitHub Gist, or did something similar.
3. You were using a very weak password — though the extent to what I tried matching that depended on the overall downloads/month that you control, all of those were at the top part of weak password lists.
4. You were reusing an old password that leaked from another site (e.g. through breaches, fishing, or anything) and your login+password or email+password combination is present in the public databases that basically anyone could download, and that could be used by malicious players.

**Once again — that is not a false alarm, change that password on every other site where you use it.** I was able to obtain it in cleartext by working with publicly available data.

And once again: **npm wasn't breached — your credentials came up from somewhere else**. You could check [haveibeenpwned.com](https://haveibeenpwned.com/) to get a hint on that.

Note that apart a few accounts that asked for that, I did not point npm to the specific leaks through other services — some of those are sensitive, and because of that I didn't want to include specific sources for everyone. That said, I included the links to specific files on GitHub for npm credentials leaks through GitHub, and I labeled the weak/dictionary passwords as being weak — so npm support should have that data.

If you still have questions that you _really_ want to be answered after checking with [haveibeenpwned.com](https://haveibeenpwned.com/) — contact me in Gitter, either in the [public room](https://gitter.im/chalker-notes) for common questions or [privately](https://gitter.im/ChALkeR). I will not be able to respond by email on this, and it will be harder for me to respond in IRC (Gitter accounts are linked to GitHub, so I can be reasonably sure to whom I am speaking).

## For everyone else

This article is in fact good news — don't be scared of this too much (you should rotate your own passwords, though).

**This is about the mitigated risk, to my knowledge no real harm has been done.**

npm became a safer place over this month. The credentials were preemptively revoked — those that were published somewhere by users, or those that leaked from other services and were reused by users at npm.

The main task of this post is to show the dangers of using weak/reused/leaked passwords, promote changing passwords to safer ones, and specify further ways of how the situation could be improved at npm registry side.

## Results

In total, there were 66876 public packages from 15495 accounts directly affected — about 13% of the whole npm ecosystem.

Taking dependencies into an account, to my estimations about 52% of the ecosystem was affected — i.e. that number of packages install affected ones along with them through dependency chains.

### Overall
* In total, I found **15568 valid credentials for 15495 accounts** since this May.
* Of those, 15343 accounts have published something (I was targeting only those for everything but npm credentials leaks). The total number of such accounts on npm was 125665, so that gives us **12% of accounts with leaked or weak credentials**.
* The total number of directly affected packages was **66876 — 13% of the ecosystem**.
* The total percentage of indirectly affected packages is estimated to be about **52% of the ecosystem** — that is, including packages affected by dependencies.
* I obtained accounts of **4 users from the top-20 list**.
* Of the affected accounts, **40 users had more than 10 million downloads/month (each)**. _For comparison, `express` package has 13 million downloads/month atm._ **13 users had more than 50 million downloads/month**.
* One of the passwords with access to publish [koa](https://www.npmjs.com/package/koa) **was literally «`password`»**.
* One of the users directly controlling more than 20 million downloads/month chose to improve their previously revoked leaked password by adding a `!` to it at the end.
* One of those 4 users from the top-20 list set their password back to the leaked one shortly after it was reset (so it got reset again).
* At least one password was significantly inappropriate — to the extent that one wouldn't want that to be linked to them online and could be publicly blamed in that case (i.e. not just a swearword). [Don't use offensive passwords](https://medium.com/@malcomvetter/offensive-passwords-451371ccd02e) — those could (and in this case were) leaked to the public in cleartext.
* **662 users had password «`123456`», 168 — «`123`», 115 — «`password`»**.
* **1409 users (1%) used their username as their password**, in its original form, without any modifications.
* **10% of users reused their leaked passwords**: 9.7% — directly, and 0.6% — with very minor modifications.
* Total downloads/month of the unique packages which I got myself publish access to was 1 946 302 172, that's **20% of the total number of d/m** directly.

### Packages

I will not include the full list (it won't fit here), nor will I include all the top packages (as some packages are owned by a single user, and listing them here will point at specific users), but all of the following packages were **directly affected** (i.e., I got myself publish permissions to those):

[**debug**](https://www.npmjs.com/package/debug),
[**qs**](https://www.npmjs.com/package/qs),
[supports-color](https://www.npmjs.com/package/supports-color),
[**yargs**](https://www.npmjs.com/package/yargs),
[commander](https://www.npmjs.com/package/commander),
[**request**](https://www.npmjs.com/package/request),
[strip-ansi](https://www.npmjs.com/package/strip-ansi),
[**chalk**](https://www.npmjs.com/package/chalk),
[**form-data**](https://www.npmjs.com/package/form-data),
[mime](https://www.npmjs.com/package/mime),
[tunnel-agent](https://www.npmjs.com/package/tunnel-agent),
[extend](https://www.npmjs.com/package/extend),
[delayed-stream](https://www.npmjs.com/package/delayed-stream),
[combined-stream](https://www.npmjs.com/package/combined-stream),
[forever-agent](https://www.npmjs.com/package/forever-agent),
[concat-stream](https://www.npmjs.com/package/concat-stream),
[vinyl](https://www.npmjs.com/package/vinyl),
[**co**](https://www.npmjs.com/package/co),
[**express**](https://www.npmjs.com/package/express),
[escape-html](https://www.npmjs.com/package/escape-html),
[path-to-regexp](https://www.npmjs.com/package/path-to-regexp),
[component-emitter](https://www.npmjs.com/package/component-emitter),
[**moment**](https://www.npmjs.com/package/moment),
[**ws**](https://www.npmjs.com/package/ws),
[handlebars](https://www.npmjs.com/package/handlebars),
[**connect**](https://www.npmjs.com/package/connect),
[escodegen](https://www.npmjs.com/package/escodegen),
[**got**](https://www.npmjs.com/package/got),
[gulp-util](https://www.npmjs.com/package/gulp-util),
[ultron](https://www.npmjs.com/package/ultron),
[http-proxy](https://www.npmjs.com/package/http-proxy),
[dom-serializer](https://www.npmjs.com/package/dom-serializer),
[url-parse](https://www.npmjs.com/package/url-parse),
[vinyl-fs](https://www.npmjs.com/package/vinyl-fs),
[configstore](https://www.npmjs.com/package/configstore),
[coa](https://www.npmjs.com/package/coa),
[csso](https://www.npmjs.com/package/csso),
[formidable](https://www.npmjs.com/package/formidable),
[color](https://www.npmjs.com/package/color),
[**winston**](https://www.npmjs.com/package/winston),
[node-**sass**](https://www.npmjs.com/package/node-sass),
[**react**](https://www.npmjs.com/package/react),
[react-dom](https://www.npmjs.com/package/react-dom),
[rx](https://www.npmjs.com/package/rx),
[postcss-calc](https://www.npmjs.com/package/postcss-calc),
[superagent](https://www.npmjs.com/package/superagent),
[basic-auth](https://www.npmjs.com/package/basic-auth),
[**cheerio**](https://www.npmjs.com/package/cheerio),
[jsdom](https://www.npmjs.com/package/jsdom),
[**gulp**](https://www.npmjs.com/package/gulp),
[sinon](https://www.npmjs.com/package/sinon),
[useragent](https://www.npmjs.com/package/useragent),
[deprecated](https://www.npmjs.com/package/deprecated),
[**browserify**](https://www.npmjs.com/package/browserify),
[redux](https://www.npmjs.com/package/redux),
[array-equal](https://www.npmjs.com/package/array-equal),
[**bower**](https://www.npmjs.com/package/bower),
[**jshint**](https://www.npmjs.com/package/jshint),
[**jasmine**](https://www.npmjs.com/package/jasmine),
[global](https://www.npmjs.com/package/global),
[**mongoose**](https://www.npmjs.com/package/mongoose),
[vhost](https://www.npmjs.com/package/vhost),
[imagemin](https://www.npmjs.com/package/imagemin),
[highlight.js](https://www.npmjs.com/package/highlight.js),
[tape](https://www.npmjs.com/package/tape),
[rollup](https://www.npmjs.com/package/rollup),
[gulp-**less**](https://www.npmjs.com/package/gulp-less),
[rework](https://www.npmjs.com/package/rework),
[ionic](https://www.npmjs.com/package/ionic),
[**normalize.css**](https://www.npmjs.com/package/normalize.css),
[n](https://www.npmjs.com/package/n),
[**react-native**](https://www.npmjs.com/package/react-native),
[**ember**-cli](https://www.npmjs.com/package/ember-cli),
[yeoman-generator](https://www.npmjs.com/package/yeoman-generator),
[**nunjucks**](https://www.npmjs.com/package/nunjucks),
[**koa**](https://www.npmjs.com/package/koa),
[modernizr](https://www.npmjs.com/package/modernizr),
[**yo**](https://www.npmjs.com/package/yo),
[mongoskin](https://www.npmjs.com/package/mongoskin),
and a lot more.

As [previously](Do-not-underestimate-credentials-leaks.md), I guess you can recognize some of those.

The list above is nowhere near being complete, I just selected some packages from it.
Overall, 46 packages have more than 10 million downloads/month (of those, 22 are listed above), 282 have more than 1 million downloads/month (of those, 62 are listed above).

_Note that some of those packages have had the number of owners reduced since then, so attempting to guess the affected users by that is likely to give wrong results. I specifically excluded packages owned by single users from the list — there were significant ones. Also, npm registry and website both sometimes return the owners list incorrectly, and that is the case for at least some of the above-mentioned packages._

I got publish access to [component-emitter](https://www.npmjs.com/package/component-emitter) (and a lot more related) via 10 independent users, to [domify](https://www.npmjs.com/package/domify) — via 9 users, [array-parallel](https://www.npmjs.com/package/array-parallel) and a lot of [mongodb-js](https://github.com/mongodb-js) components — via 5 users each, [superagent](https://www.npmjs.com/package/superagent) — via 3 users, [cheerio](https://www.npmjs.com/package/cheerio), [browserify](https://www.npmjs.com/package/browserify), [koa](https://www.npmjs.com/package/koa), [mongoose](https://www.npmjs.com/package/mongoose), [modernizr](https://www.npmjs.com/package/modernizr), [react](https://www.npmjs.com/package/react), [tape](https://www.npmjs.com/package/tape), [winston](https://www.npmjs.com/package/winston), [ws](https://www.npmjs.com/package/ws) — via 2 users each. I'm also not listing all of those, 1819 packages in total were accessible through more than one user, 38 of those with more than 1 millon downloads/month, 7 — with more than 10 million downloads/month.

### Detailed
* Bruteforce attack using very weak passwords gave me 5470 total packages from 2545 accounts.
* [Utilizing](https://twitter.com/slatestarcodex/status/854382497261596676) datasets from known public leaks gave me 57112 total packages from 12150 accounts (directly).
* Fuzzing the passwords from those known public leaks a bit (appending numbers, replacing other company names with «`npm`», etc) gave me 4786 packages from 732 accounts.
* New npm credentials leaks (GitHub, Google, etc) gave me 582 total packages from 120 accounts.

#### Leaks of npm credentials on GitHub and other places

These are some of the packages that I got myself publish access to by collecting npm credentials leaked to various places recently: [conventional-changelog](https://www.npmjs.com/package/conventional-changelog), [fetch-mock](https://www.npmjs.com/package/fetch-mock), [sweetalert2](https://www.npmjs.com/package/sweetalert2).

In total, there were 120 accounts and 582 packages.

This is mostly covered in the [previous post](Do-not-underestimate-credentials-leaks.md) — I made a tool to automatically gather those using the GitHub API, and validate gathered results on npm `/whoami`. I shared that tool with npm, Inc.

Some of those were manually collected (i.e. not on GitHub) — for example, npm tokens appearing in CI.

#### Bruteforce

I gathered 2545 accounts from bruteforce, controlling 5470 total packages and 217 690 203 d/m directly.

I got direct publish access to a number of packages, including, but not limited to, the following ones (ordered by downloads/month):

`████`, [form-data](https://www.npmjs.com/package/form-data), `████████`, [path-to-regexp](https://www.npmjs.com/package/path-to-regexp), [component-emitter](https://www.npmjs.com/package/component-emitter), [merge-descriptors](https://www.npmjs.com/package/merge-descriptors), `████████`, `████████`, …

That list also included (unordered): [global](https://www.npmjs.com/package/global), [koa](https://www.npmjs.com/package/koa) (and other `koa`-related), [path-to-regexp](https://www.npmjs.com/package/path-to-regexp), [configstore](https://www.npmjs.com/package/configstore), [yo](https://www.npmjs.com/package/yo), [yeoman-generator](https://www.npmjs.com/package/yeoman-generator) (and other `yeoman`-related), [moment](https://www.npmjs.com/package/moment), [mongoose](https://www.npmjs.com/package/mongoose) (and other related), [v8-profiler](https://www.npmjs.com/package/v8-profiler), [node-inspector](https://www.npmjs.com/package/node-inspector), [mongodb-extended-json](https://www.npmjs.com/package/mongodb-extended-json) and other mongodb-js related, [mz](https://www.npmjs.com/package/mz), [cheerio](https://www.npmjs.com/package/cheerio), etc.

One of the accounts with publish access to `koa` had a password `password`, literally.

One of the users controlling ~2 million package downloads / month had their npm username as a password.

The bruteforcer activity was noticed an prevented (via an IP ban) after approx 1½ days and a little less than 3 million requests to `/whoami`, but that was enough to bruteforce the top accounts — I gathered 35 accounts using this bruteforce before my IP got blocked, totaling to approx 151 000 000 package downloads per month (69% of the total bruteforce impact). I could have continued from another IP, but I notified npm at that point and coordinated with them — one of the initial questions was how fast would they notice the attack. Since then, those endpoints became better monitored and I was notified that a stricter ratelimit is now imposed. Note that a real attacker [would have started](#qa) with checking leaked credentials, and would have been able to gather most if not all of the high-impact ones in that time.

#### Reused passwords leaked from other services

This gave me 57112 total packages from 12150 accounts directly, totaling to 1 693 349 422 downloads/month.

As that is the most part of the data — I will not repeat the affected packages list.

The datasets used are publicly available — I found them in Google.

Another thing necessary to mention is that bruteforce protection wouldn't have been able to protect against this kind of attack — most of the account matching was done offline, and many accounts matched with high confidence (i.e. by email). A very small number of requests to `/whoami` could have given most of the top impact accounts gathered there — that wouldn't have been noticed or prevented in due time.

#### Fuzzed passwords

By «fuzzing» I mean performing modifications to passwords gained from third-party leaks and attempting the modified password on npm.

Fuzzing included (among other attempted modifications): changing the capitalization, attempting/removing digits (and other symbols) at the end, replacing company names with `npm`, appending/prepending `@npm` (and others), and other various changes. That was quickly achievable because there were already known potential passwords matched for each account — and there were not too many of those, so even multiplying those by one or two orders of magnitude (depending on the account significance) was possible.

That gave me 4786 packages from 732 accounts. The top package gained that way had 10 millon downloads/month, and the total was 171 174 218 d/m.

[react](https://www.npmjs.com/package/react), [redux](https://www.npmjs.com/package/redux), [browserify](https://www.npmjs.com/package/browserify), [rollup](https://www.npmjs.com/package/rollup), [sinon](https://www.npmjs.com/package/sinon), and other packages were directly affected.

#### Technical details

I obtained all the used datasets from public sources. There were some minor issues with that (some of the archives are plain broken or incomplete), but nevertheless those are easily reachable.

I have written my own tooling to work with the datasets, to match those with npm accounts, and to use that as the input for the bruteforcer. That is basically a script that parses the file, looks for usernames/emails, splits those in prioritized buckets depending on the user impact (in downloads/month) and the confidence of the password (was it matched by email or name), and decodes the passwords if needed.

I have used Node.js for that tooling, it worked pretty well. I also (as usual) used the excellent [lz4](https://github.com/pierrec/node-lz4) module to keep the datasets compressed on disk and for keeping the scans fast.

## Common problems on the users side that lead to this

 1. Using unsafe passwords. Duh.
 2. Reusing passwords on other services.
 3. Not rotating passwords, i.e. it often happened that a user set a weak password initially when there was nothing important in the account thinking that it doesn't matter much, and then forgot to change it when it started mattering.
 4. Adding publish permissions to users who don't really need it.
    At least one user was surprised that they had access to something with a considerable number of downloads/month and told me that they don't remember requesting it — that user just filed some patches to the package and did some other stuff, but never actually published it (and probably wasn't supposed to).
    I have often seen a large number of users having publish access to various packages without an actual need for that.
    Perhaps that is done to get their avatar displayed next to a package? I have no other ideas.

Also note that what makes this more dangerous is some security mechanisms (e.g. 2FA) not currently being present in npm registry — see below for more info.

## What have been done by npm, Inc

* Basic auth is going to be limited soon: [link](http://blog.npmjs.org/post/160809090595/basic-auth-to-be-limited-soon). This should make npm credential leaks via GitHub/etc. less harmful.
* Those leaked credentials that I was able to obtain were reset: [link](http://blog.npmjs.org/post/161515829950/credentials-resets).
* Password rules were implemented — too short and known weak/common passwords are not allowed anymore: [link](https://www.npmjs.com/password).
* 2FA is still in progress.

## How things could be further improved on the npm side

See also [Short-term package manager wishlist](https://github.com/ChALkeR/notes/blob/master/Short-term-package-manager-wishlist.md) from the last year, which already covers most of these recommendations.

This list is ordered, by impact (from higher to lower) combined with estimated complexity of implementation (from less complex to more complex).

 1. **Partial** A proper bruteforce protection mechanism. Getting this one correct is hard, see [here](https://www.owasp.org/index.php/Blocking_Brute_Force_Attacks) for explanation, but even significantly reducing the reachable speed could greatly improve things, combined with other methods.
 1. **Done** ~Monitor overall failed auth requests — that should almost immediately give a hint that a bruteforce attack is running.~
 1. Notify package authors when a new version of a package they own is packaged — both on the [npmjs.com](https://www.npmjs.com) website (similar to GitHub notifications, for example) and through the email (the email should have an out-out, of course).
 1. **Done** ~Check for password weakness upon registration and when changing the password. Too short, known weak (from the top-used lists), and passwords containing username of a significant part of it (case-insensitive) should be rejected.~
 1. The existing users should be taken through this check. Perhaps the users with different impact in downloads/month should be treated differently, and while it will probably be enough to just send warnings to users without any packages at all, it's probably worth to actually reset weak password for accounts who have publish access to significantly popular and crucial packages. This has to be redone from time to time (note that the password lists, the rules, the download stats and the user-package relations will be changing over time).
 1. **[Soon](http://blog.npmjs.org/post/160809090595/basic-auth-to-be-limited-soon)** ~Deprecate password-based authorization (that is, `_password`/`_auth`) in the client in favor of token (`_authToken`) one — [#9866](https://github.com/npm/npm/issues/9866).~
 1. **[Not needed anymore](http://blog.npmjs.org/post/160809090595/basic-auth-to-be-limited-soon)** ~Implement an opt-out from password-based auth at the server for the users. The `login` endpoint could be excluded from the opt-out, given that it (and the website) will have much stricter limits and bruteforce protection than the other endpoints. That also will make it sure that the users aren't locked out of their accounts by the bruteforce protection as long as they are using auth tokens. _Another way would be to also limit the `/login` and to allow creating tokens from the website only — but to my estimation that would make things significantly more complex, and having users to manually copy-paste tokens could be in fact **worse** from the security point of view._~
 1. Add another way of being somehow displayed with an avatar at the package page, but without the publish access.
 1. 2FA for the users and allowing to enforce 2FA requirement for package-level access. This is the change that would greatly improve the security, it's being listed at the bottom here only because it requires significantly more work to be implemented.
 
As 2FA on the client seems to be complex and will likely take long time to implement, here is another plan instead of 2FA on the clients, which could be easier to achieve, without having 2FA on the client (note: doesn't cancel the need in bruteforce protection and other entries from the above list):
* **[Soon](http://blog.npmjs.org/post/160809090595/basic-auth-to-be-limited-soon)** ~~The client just needs the password-based auth deprecation everywhere but the `/login`, all other changes need to be applied only to the website~~
* Opt-in confirmation of all published packages through the website
* Opt-in 2FA on the website
* Opt-in 2FA/confirmation requirements for packages/teams
* Send notifications on package publishes, both website and email (email with a opt-out)
* Send notifications when a package was pushed but not confirmed or rejected (so not published) for half an hour
 
## What users should do on this

* If you have an account on npm:
  * Make sure that your password is not weak: i.e. not short, not trivial, not a dictionary one and does not include your username.
  * Rotate your passwords.
  * Rotate your auth tokens, i.e. delete them from the [website](https://www.npmjs.com/settings/tokens) and relogin again.
  * Don't reuse the same password on other sites.
  * Make sure that you are not using password-based auth in `.npmrc` — `_authToken` is good, `_auth` and `_password` are bad.
  * Check **all** your logins and emails on [haveibeenpwned.com](https://haveibeenpwned.com). Make sure that you don't forget about historic emails.
  * Check if you have publish access to any popular packages on npm. _Note: some users that I talked to were not aware that they are added to some popular packages._
* If you have popular packages published on npm:
  * Make sure that the list of people who have the permissions to publish it is not unnecessary huge and does not include unrelated people who are not active, not going to publish that package, or otherwise don't need those permissions.
  * Re-check the published versions of those popular packages — do those include any versons that you are not aware of? _I do not expect that and haven't seen that, but that is always a good thing to check, as it is theoretically possible if someone malicious got hold of credentials of anyone with publish access_.
  * Contact npm support if unsure.
* If you don't host any packages on npm and are just installing stuff from it (and don't have an account) — there is nothing you should do about that, everything is fine for you.

## Q&A

1. **What data was accessed?**
   I did not attempt to access anything apart from the `/whoami` endpoint.
2. **What about private packages? How many of them were there?**
   I am measuring only public packages, as I did not check anything but the `/whoami` endpoint and have no idea how many private packages are there.
3. **What about users without any packages?**
   Those were not included in anything but the npm credentials leaks on GitHub and other websites.
   I did not attempt to match those with leaked accounts from other sites or to bruteforce passwords for them.
   One of the reasons for that is because those are not very important, and another is because I don't have a list of those.    I needed a list of users (with their emails) to match with leaked accounts from other websites, and I gathered that from the dump of all public packages (`byField.json`).
4. **Was this attack really possible?**
   This was pretty close to being a real attack — I did not use any internal knowledge and launched that without coordination to check what I could get. I also used only publicly available data — i.e. npm registry API and downloaded leaked datasets found in Google.
   
   My attack simulation had a few differences from how a real malicious attacker would behave:
   * I did not attempt to hide. A real attacker could have used a distributed botnet (there are even ways to get that literally for free) for performing those requests. I used a single IP to see how fast would the attack be noticed (though it changed a few times back and forth with the location change).
   * I did not start with the leaks — those took time to prepare, and I wanted to prevent the potential damage asap. A real malicious attacker would have prepared the full lists offline first, prioritized them, and started with high-confidence high-impact passwords (instead of bruteforce like I did to estimate how fast an attack would have been noticed).
   * Once noticed and blocked, I did not attempt to switch the IP, continue the attack by other means, or using those accounts — I contacted npm, Inc. instead and proceeded in coordination with them. The primary reason for not telling from the start was to observe the prevention mechanism in real life (so I could record and report issues with that).
   * No real harm was done — i.e. no payload was launched. The target was to explore and prevent the feasibility of gaining control over npm accounts, and nothing beyond it. A real attacker would have continued this, bootstrapping a real malicious attack with those gathered credentials.
4. **You shouldn't have downloaded those datasets with my password.**
   Tell that to a real attacker which could have done that instead.
   Those passwords are already public, and rechecking them and preventing the potential catastrophic damage for a lot of users (which could have been done if someone uploaded malware to all or any of those) is significantly more important that not looking at those leaked passwords (which have been already looked at by a lot of people).
5. **My password was reset but I don't know where it was leaked.**
   If your password was «qwerty», «12345» or anything from the top of the common password lists — duh.
   Anyway — check **all** your emails (and your npm username) at [haveibeenpwned.com](https://haveibeenpwned.com/).
   If that is green (or no entry includes passwords or hashed passwords) — most likely, it came from GitHub or other places, contact npm support, they should have that data.
   Note that because of the fuzzed passwords, you password does not have to be _exactly_ the same as the leaked one, it just has to be *highly similar*. E.g. I replaced «adobe» with «npm» in passwords, added/removed digits at the end, etc.
   If still wondering after doing all of the above — you can contact me in Gitter, either in the [public room](https://gitter.im/chalker-notes) for common questions or [privately](https://gitter.im/ChALkeR).
6. **I reuse my password but it wasn't reset — am I safe?**
   No such check performed could give you a guarantee against being affected. Rotate your passwords and do not reuse them. Using highly similar passwords (e.g. `random@adobe`, `Random`, and `random@npm`) also counts as reusing.

---

Published: 2017-06-21 20:30 UTC.

If you have any questions to me, contact me over [Gitter](https://gitter.im/ChALkeR) (@ChALkeR) or IRC (ChALkeR@freenode).
