# Do not underestimate credentials leaks.

While one might think that leaking credentials is pretty much a noob mistake, and if you are feeling yourself being safe because you are an experienced developer, that’s a wrong impression leading to underestimation of the problem.

I performed a rather simple credentials search using only two methods: wget/untar/grep on npm packages and GitHub Search.
The queries were pretty simple, it did not take much time, and that’s a quite obvious thing to do.
Do you remember yourself laughing at «begin rsa private key» search results over GitHub and people who publish that?

## So, here are the results

I obtained direct access to the following packages (publishing on npm or/and GitHub push access), you might have heard of some of them:
* [Express](https://npmjs.org/package/express),
* [EventEmitter2](https://npmjs.org/package/eventemitter2),
* [mime-types](https://npmjs.org/package/mime-types),
* [semver](https://npmjs.org/package/semver),
* [npm](https://npmjs.org/package/npm),
* [fstream](https://npmjs.org/package/fstream),
* [cookie](https://npmjs.org/package/cookie) (and [cookies](https://npmjs.org/package/cookies)),
* [Bower](https://npmjs.org/package/bower),
* [Component](https://npmjs.org/package/component) (double, via two unrelated users),
* [Connect](https://npmjs.org/package/connect), [koa](https://npmjs.org/package/koa), [co](https://npmjs.org/package/co), [tar](https://npmjs.org/package/tar), [css](https://npmjs.org/package/css), [gm](https://npmjs.org/package/gm), [csrf](https://npmjs.org/package/csrf), [keygrip](https://npmjs.org/package/keygrip), [jcarousel](https://npmjs.org/package/jcarousel), [serialport](https://npmjs.org/package/serialport), [basic-auth](https://npmjs.org/package/basic-auth), [lru-cache](https://npmjs.com/package/lru-cache), [inflight](https://npmjs.com/package/inflight), [mochify](https://npmjs.org/package/mochify), [denodeify](https://npmjs.org/package/denodeify), and a lot of other packages.

Note that some packages were exposed more than once via unrelated users.

Some packages were indirectly affected — the packages that are not affected by themselves, but that depend on affected packages, so that installing such package installs an affected package.
As an example: [Grunt](https://www.npmjs.com/package/grunt) depends on EventEmmiter2, so it was also indirectly affected.
[Request](https://npmjs.com/package/request), [Browserify](https://npmjs.com/package/browserify), [grunt](https://npmjs.com/package/grunt), [grunt-cli](https://npmjs.com/package/grunt-cli), [cordova](https://npmjs.com/package/cordova), [forever](https://npmjs.com/package/forever), [less](https://npmjs.com/package/less), [pm2](https://npmjs.com/package/pm2), [node-gyp](https://npmjs.com/package/node-gyp), [karma](https://npmjs.com/package/karma), [yo](https://npmjs.com/package/yo), [strongloop](https://npmjs.com/package/strongloop), [glob](https://npmjs.com/package/glob), [mocha](https://npmjs.com/package/mocha), [sails](https://npmjs.com/package/sails), and a lot of other packages were all also indirectly affected (via deps).
Most of that is chained — for example, [request](https://npmjs.com/package/request) depends on [mime-types](https://npmjs.com/package/mime-types), and a lot of packages depend on [request](https://npmjs.com/package/request).

Taking indirectly affected (via deps) packages into an account, it could be seen that from the 15 [packages people 'npm install' a lot](https://www.npmjs.com/#explicit) only two were not affected.

Overall, there were around 90-135 npm passwords, around 40-80 npm tokens, around 100 GitHub tokens, and a few GitHub passwords. Those are the ones that I found, scans done by the corresponding teams most probably found more of them.

## Common sources of leaks
_This is not an comprehensive list of course, just a few examples:_

1. Dotfiles: various things, for example:
 1. `.ssh` private keys (duh) — if you are publishing _that_, you are in great trouble.
 2. `.npmrc` (duh) — npm auth.
 3. `.muttrc` (duh) — email auth data.
 4. sublime text config (wut?) — GitHub oAuth tokens.
 5. `config.json` (duh) — GitHub oAuth tokens, supposedly from Composer.
 6. `.gitconfig` — (duh) — GitHub auth / oAuth tokens.
 7. `.netrc` — (duh) passwords.
2. Per-package stuff: both inside npm packages and on public git repos (not only limited to Github):
 1. `.npmrc` (duh) — npm auth. Always excluded from npm packages nowdays if you are using a recent 2.x/3.x npm version.
 2. `package.json` / `Gemfile` / `Gemfile.lock` (wut?) — people sometimes put their GitHub oAuth tokens inside them in a repo url. Ow!
 3. `config.gypi` — saves the build process environment (wut?). Old npm clients [leaking their credentials](https://github.com/npm/npm/releases/tag/v2.14.1) into the environment of child processes (in some cases) makes this even more dangerous.
 4. `travis.yml` (ow) — could contain sensitive stuff like API keys (including npm logins/passwords or authTokens). Not everyone uses the [encryption](http://docs.travis-ci.com/user/encryption-keys/). They should.
 5. `config.json` (duh) — GitHub oAuth tokens, supposedly from Composer. Also present in `composer.json`, `.travis.composer.config.json`, etc., and some configuration scripts.
3. Websites/scripts configuration:
 1. DB auth data (duh), frequently in a form of a connection string.
 2. GitHub oAuth tokens (wut?). People use tokens with write permissions to all their repos on their webpages, just to fetch some stuff from Github.
 3. GitHub passwords (wut?). People use that in their scripts or configuration.
4. Logs.
 
 All sort of stuff could get into logs, if something fails to take care of that. It's better to inspect logs before publishing them. Logs can save your passwords, the environment variables, http logs could save auth data, some logs can save what you type or search. _Note: those are not theoretical examples, I saw all of those on GitHub._ Be extremely careful when publishing logs.

## Specific examples

### Express, mime-types, Component, …
What: npm password, npm token.

Where: npm.

Exposed through: credentials leak through config.gypi in one package saving environment variables and npm leaking it’s credentials to the env of child processes in some cases.

Fixed: user changed password and revoked the token. On npm side — by both not leaking credentials to the env (client-side) and automatically revoking packaged tokens/passwords (server-side).

Window: December 2014 — September 2015.

### npm, semver, …
What: npm password.

Where: GitHub Gist.

Exposed through: credentials leak through a published log file saving environment variables and npm leaking it’s credentials to the env of child processes in some cases.

Fixed: user changed password and revoked the token. On npm side — by both not leaking credentials to the env (client-side) and automatically revoking packaged tokens/passwords (server-side).

Window: April 2015 — October 2015.

### Bower
What: GitHub oAuth token (with push access to all the repos in the org, including the website).

Where: GitHub.

Exposed through: GitHub oAuth token leak through a Gemfile in another repo.

Fixed: user revoked the token. On GitHub side — by automatically revoking publically commited tokens.

Window: April 2015 — September 2015.

### EventEmitter2, Component (again), …

What: npm password and GitHub password.

Where: GitHub.

Exposed through: Password leak in dotfiles. The password was shared between npm and github.

Fixed: user changed passwords.

Window: First half of 2013 — October 2015.

### JCarousel

What: GitHub password.

Where: npm.

Exposed through: Password leak through config file of an utility (that shouldn’t be included) in two packages.

Fixed: user changed passwords.

Window: April 2014 — August 2015.

--

There were a lot of other packages, I am not including all of them.

## The main point of this

Being an experienced or a certified Node.js developer, being an _Node.js expert consultant_, having over 500 packages on npm or having yourself in the top few answerers on StackOverflow in your field, being big, being a CEO, having tens of thousands stars on GitHub or followers on Twitter, working for Intel, Google, or npm itself, studying/researching at Berkley, Harvard or MIT, having kids — nothing of that is going to secure you from credentials leakage by itself.

So, the solution for this problem is to:

1. On npm side — credentials leakage into the environment was fixed.
2. On npm and GitHub side — automatic search for their auth data in public packages/repositories at the time of publishing/committing/making a private repo public, then revoking the leaked tokens and resetting leaked passwords. This is done now, and GitHub was already doing that earlier, but their filters were misconfigured.
3. On your side — **check your packages and repositories**. Especially (but not limited to) dotfiles. If you share your dotfiles, review every file there.
4. If you found that your npm password has been made public, the only solution for the moment is to contact support (after resetting the password, of course). _See Q/A below._
5. If you have leaked your password and/or token — review all packages and/or repos that you have access to (not only limited to your own repos). Note that push access on GitHub means that the commit that could be pushed from your account could be authored by either you or _another person_, so reviewing only your own commits is not enough. _See Q/A below._

**Check your stuff.** _Now._

And keep it safe from now on.

## Q/A

_Note: I'm not affiliated with GitHub or npm, Inc., so everything below is pretty much my personal opinion._

1. **Was I affected?**
   * If you did receive an email from npm/GitHub support (and your password/tokens was reset/revoked) — you were affected, if you did receive an email from me — you were also affected.
   
     Please, don't re-use the same password to «fool» the robot while restoring your password — this will result in your account being vulnerable. *Yes, several people have done that.*
   * If you didn't receive any emails and your password/tokens did not get reset/revoked — that means that your account was not found to be leaking credentials, but _that does not guarantee anything_. You should better check your stuff yourself.
   
     I targeted mainly GitHub/npm credentials, and my analysis wasn't comprehensive.
 
     npm performed a search for npm credentials in packages published on npmjs.org (and they are now running it for all new packages). GitHub catches tokens that are made public on GitHub.
     
     There is no guarantee that you (or anyone else) did not commit your email password or your db access string to either npm or GitHub. You should take care about that yourself.
2. **Is my npm version fine?**
 
 Credentials leak to the into the environment of child processes was fixed in npm [v2.14.1](https://github.com/npm/npm/releases/tag/v2.14.1).
 
 If your version is older that that (i.e. if you use Node.js v0.10 or v0.12.x lower than v0.12.8 and haven't updated `npm` manually) — you should update to the latest `npm` version in either `3.x` or `2.x` branch.
 
 _Note:_ This only affects users that are using an account at some registry (i.e. are registered at npmjs.org).
2. **I was affected to credentials leak, what should I do?**
 1. **My npm `authToken` was leaked. What should I do now?**
    * Revoke the affected authToken using `npm logout` on the machine where it was used (if that was not done for you already by npm support).
    * Fix the source of the leak to make sure that you don't expose your `authToken` again.
    * After that, review any new versions of packages that you have access to and that were published from your account.
      
      Note that this is not only limited to the latest version, the attacker could publish a `1.0.4` version if there are `1.0.3` and `2.0.1` in an attempt to make the interference less noticeable (especially if 1.x branch is still being used more than 2.x).
    * When in doubt — contact npm support.
 2. **My npm password was leaked. What should I do now?**
    * Change your password and contact npm support in order to revoke all your oAuth tokens (if that was not done already).
      
      Revoking all the oAuth tokens is necessary and can't be done by yourself — an attacker could have created an oAuth token that you are not aware of and that you can't see or revoke, and that token could still be functional.
    * If the password was shared with other accounts — secure those too, by resetting the password on those accounts and performing the other necessary actions.
    * Fix the source of the leak to make sure that you don't expose your password again.
    * After that, review any new versions of packages that you have access to and that were published from your account.
      
      Note that this is not only limited to the latest version, the attacker could publish a `1.0.4` version if there are `1.0.3` and `2.0.1` in an attempt to make the interference less noticeable  (especially if 1.x branch is still being used more than 2.x).
    * When in doubt — contact npm support.
 3. **My GitHub token was leaked. What should I do now?**
    * Check the scopes of the leaked token. If there are no scopes — it can't do any harm.
    * If the list of scopes is not empty — revoke the affected oAuth token (if that was not done for you already by GitHub support).
    * Fix the source of the leak to make sure that you don't expose your tokens again.
    * If there was an `repo` (or `public_repo`) scope — review if any changes were made to repos (or public repos) that you have access to.
      
      Note that changes that were possibly made from your account are not limited to commits authored by you, they could be authored by anyone.
    * When in doubt — contact GitHub support.
 4. **My GitHub password was leaked. What should I do now?**
    * Change your password, revoke your oAuth tokens (if that was not done for you already by GitHub support).
    * If possible — enable 2FA.
    * If the password was shared with other accounts — secure those too, by resetting the password on those accounts and performing the other necessary actions.
    * Fix the source of the leak to make sure that you don't expose your password again.
    * After that, review if any changes were made to repos that you have access to.
    
      Note that changes that were possibly made from your account are not limited to commits authored by you, they could be authored by anyone.
    * When in doubt — contact GitHub support.
4. **How come no one noticed this earlier?**
 
 No idea. Perhaps no one checked.
5. **Publishing that information about leaked tokens is bad, you shouldn't have done that.**
 
 Well, _not publishing_ this is much worse. People should be aware of dangers this brings, and should know what to do in case of credentials leaks.
 
 Also, people should better check what they publish, and this is an attmpt to make them more cautious.
 
 I'm not pointing onto specific accounts or people. Please, don't try to find and accuse specific people based on the package names that I mentioned — that's not polite and will do no good, also the access configuration was changed for some packages since then, so you will probably get wrong results.
6. **What lessons should we learn here?**
 1. You should better watch your credentials.
 2. Don't underestimate credentials leaks (well, as well as other security risks).
 3. Don't make assumptions, better check. It turned out that some people weren't aware that npm packages everything in the current dir that's not ignored, some people somewhy were under an impression that it follows .git. _On a side note: I saw all kinds of irrelevant stuff being packaged._
 4. Plaintext passwords are bad. No, seriously. There were much more passwords leaked than tokens, and that's because people still have their password in .npmrc.
  **If you have `_auth` or `_password` in your `.npmrc` — remove it**, and perform an `npm login` to generate an `_authToken` instead.
 5. npm lacks a way to list or revoke all the authTokens. That is bad for the reasons described above.
    
    I was told that this is being worked on.
 6. There are a lot of people with publishing access to some popular packages, and I doubt that's really needed.
    
    I suspect that one of the reasons for this is to include some persons into the «Contributors» list on package page.
    
    I was told by npm team that a more fine-grained access level control is being worked on.
 7. Pretty much harm could be done by only a few leaked credentials. There are a number of extremely popular packages that pretty much everything depends upon.
    
    I'm not talking about npm/Express now, but about mime-types, fstream, EventEmitter2, semver, cookie, etc. Updates in those packages, even in the old branches, should not be left unnoticed.
    
    Perhaps some visual view of version updates (not only the latest ones, but all branches) of the top downloaded packages would be nice.
7. **Why was April such a bad month for the ecosystem security?**
 
 No idea, really. Looks like coincidence to me.
8. **What about automatic tools or pre-commit hooks that scan for credentials? Should I write/use one?**
 
 While using an automatic checker would do no direct harm, most probably it would just give you a false sense of security, but will not protect you from the actual leak.
 
 There are many ways how various credentials could leak, and a general purpose tool aimed to catch all kinds of credentials (even targeted exclusively towards leaks trough git repos or npm packages) would be quite complex and will inevitably be subject to both false negatives and false positives.
 
 Instead, you should better review stuff that you publish. That includes commit review, package contents review before publishing them, config files review, logs review before sharing them. If you have a team of several developers — it would be better to educate your devs more and make each commit go through an independent review instead of forcing your team to use some automatic tool. Also, don't forget about checking package contents.

--

Special thanks to GitHub and npm teams for taking the issue seriously and patching their server side to automatically invalidate leaked tokens/passwords when they are published on their own service (GitHub repos for GitHub oAuth tokens, npm packages for npm tokens and passwords).
_Note: GitHub has already been doing that, but it seems that their filter has not been catching all the cases._

--

Published: 2015-12-04.

Historic revisions (which are not in this repo): https://gist.github.com/ChALkeR/7485b79f6ef15768a309/revisions?page=2

If you have any questions to me, contact me over [Gitter](https://gitter.im/ChALkeR) (@ChALkeR) or IRC (ChALkeR@freenode).
