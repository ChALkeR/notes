_First of all, an affiliation change notice (this is recent):\
I am currently working for [Exodus](https://www.exodus.io/) and am sponsored by them to perform security research._

# Yarn transfered npm credentials over unencrypted http connection

This was reported by me through the [Node.js third-party modules](https://hackerone.com/nodejs-ecosystem) program
and was fixed in the just-released yarn v1.17.3. See also [yarn blog post](https://yarnpkg.com/blog/2019/07/12/recommended-security-update/) on this.

## Vulnerability description

For scoped packages that are listed as `resolved "http://registry.npmjs.org/@`... in `yarn.lock`, affected [yarn](https://www.npmjs.com/package/yarn) versions
trasfer npm credentials (i.e. `_authToken`) over unencrypted http connection.
This allows any MitM (for example, a proxy or a VPN, or a cafeteria Wi-Fi) to sniff out npm credentials,
given that the developer performs `yarn install` on such a `yarn.lock` file.

## Timeline

* 2019-07-12 - Found
* 2019-07-12 11:31 UTC - Reported to [Node.js third-party modules](https://hackerone.com/nodejs-ecosystem) program
* 2019-07-12 12:12 UTC - Triaged
* 2019-07-12 12:19 UTC - Yarn dev joined the conversation
* 2019-07-12 12:47 UTC - Initial response and confirmation from Yarn dev
* 2019-07-12 13:22 UTC - [Fix PR](https://github.com/yarnpkg/yarn/pull/7393) created by Yarn devs
* 2019-07-12 14:19 UTC - Fix landed
* 2019-07-12 15:28 UTC - Fixed version (v1.17.3) released as `rc`
* 2019-07-12 21:42 UTC - Fixed version (v1.17.3) released as `latest`
* 2019-07-13 08:40 UTC - Yarn blog post published

## Impact

Attacker (MitM) being able to:

* Impersonate the affected account
* Publish packages (including those containing malicious code) from the affected account that could also get used by the affected account/company in the future (for protected packages) and by anyone in the ecosystem (for public packages)
* Perform logout and break installs of protected packages
* Unpublish packages from the registry (with limitations)

## Exploitability

A quick search shows that there is a large number of `yarn.lock` files affected by this on GitHub, [some](https://github.com/EC-Nordbund/ec-verwaltungs-app/blob/ab961352d5dd53834a51793d6e2c4bc69a2b22d4/packages/api/yarn.lock#L36) [examples](https://github.com/nujabes403/boilerplate2/blob/61613e526aec02c5dd4227457deb8676d66780d0/yarn.lock#L7).\
There seem to be many of those.

Looks like not only it was possible to craft a `yarn.lock` with a malicious intent, but also this seems to be a common pattern that yarn created itself at some point or under some circumstances and that gets persistent from older versions.

Yarn explanation (from their [blog post](https://yarnpkg.com/blog/2019/07/12/recommended-security-update/)):
> For a few months in 2018, the npm registry [returned http urls instead of the regular https ones](https://npm.community/t/some-packages-have-dist-tarball-as-http-and-not-https/285/40). Although the problem seems to have been corrected earlier this year, the lockfile entries generated during this period may still reference http urls and cause Yarn to send authentication data unencrypted.

This looks very common, mostly coming from `@babel` and `@types` scopes.

This could also be exploited with malicious intent -- i.e. an attacker could construct/modify a `yarn.lock` manually to exploit this vuln, not only as a result of that incident in 2018.

I do not think that these deps are something that is likely to be catched by PR review unless one knows what to look for (examples above show that it was not noticed), so anything in line with "users should have reviewed `yarn.lock` files for this not to happen" does not work as a solution here. The problem is that users probably won't catch that packages being resolved to `http://` could mean that the yarn lock is "compromised". A lot of such existing `yarn.lock` files that no one noticed (even with scoped packages) indicate that.

Some historic examples in facebook repos: [docusaurus](https://github.com/facebook/docusaurus/blob/v1.4.0/yarn.lock#L7) (same file [here](https://github.com/facebook/docusaurus/blob/v1.6.0/v1/yarn.lock#L7)), [jest](https://github.com/facebook/jest/blob/v23.4.0/yarn.lock#L101).  Moreover, [yarnpkg/yarn](https://github.com/yarnpkg/yarn) lockfile itself has a `http://`-resolved dep: https://github.com/yarnpkg/yarn/blob/v1.17.3/yarn.lock#L5646. It's not scoped, but that difference might also be subtle while reviewing a PR. There are more of those in git history.

## What should I do?

* Update yarn to v1.17.3 or later in all your environments.
* Go to [Settings -> Tokens](https://www.npmjs.com/settings/~/tokens) on [npmjs.com](https://www.npmjs.com/) and delete all listed tokens. Relogin where you need.
* Make sure that your `yarn.lock` files do not have entries resolved to `http://` registries.
* Enable 2FA on npm for logins and publishes, if you haven't done so already.

As a side note: periodically rotating your npm tokens (i.e. revoking them and doing a re-login) is a good idea in general.
Also, don't forget to keep track of your tokens through the [Settings -> Tokens](https://www.npmjs.com/settings/~/tokens) page.

## How to check if a certain repo was affected?

Use the following command, for example:
```sh
git log --pretty=format:"%h" -- yarn.lock | xargs -n1 -I{} git show {} -- yarn.lock | grep 'http:.*/@'
```

Example output that indicates that `http://registry.npmjs.org/@types/node` has been used in `jest` at some point:
```console
jest chalker$ git log --pretty=format:"%h" -- yarn.lock | xargs -n1 -I{} git show {} -- yarn.lock | grep 'http:.*/@'
-  resolved "http://registry.npmjs.org/@types/node/-/node-9.4.7.tgz#57d81cd98719df2c9de118f2d5f3b1120dcd7275"
   resolved "http://registry.npmjs.org/@types/node/-/node-9.4.7.tgz#57d81cd98719df2c9de118f2d5f3b1120dcd7275"
   resolved "http://registry.npmjs.org/@types/node/-/node-9.4.7.tgz#57d81cd98719df2c9de118f2d5f3b1120dcd7275"
+  resolved "http://registry.npmjs.org/@types/node/-/node-9.4.7.tgz#57d81cd98719df2c9de118f2d5f3b1120dcd7275"
```

## Steps to reproduce

It is not wise to do that with your active token, and you probably won't need this section, but here it is for anyone
who is interested:

1. Perform an npm login or just write `//registry.npmjs.org/:_authToken=38bb8d1f-a39b-47d1-a78e-3bf0626ff77e` (which is the format npm uses) to ~/.npmrc. Doing this from your own account would leak your npm credentials on next steps, so better just use a placeholder.
2. Create an empty package with a single dependency on `"@babel/core": "^7.5.4"`
3. Perform `yarn install`
4. Replace all occurances of `https://registry.yarnpkg.com` with `http://registry.npmjs.org/` in the generated `yarn.lock`

   Alternatively to steps 2-4 -- just use an already existing yarn.lock with `resolved "http://registry.npmjs.org/@` in it (lots of those on GitHub), but be careful with that.
5. Clear yarn cache and node_modules: `rm -rf ~/.cache/yarn/ node_modules`. Let's assume you just downloaded an affected `yarn.lock` on your clean machine.
6. Start wireshark with `tcp dst port 80` filter.
7. Run `yarn install`

Observed result is attached on a screenshot below:
![](/media/yarn.http.png)

## Suggestions

Apart from the applied fix in yarn v1.17.3, I would ideally like to see the following changes:

* Disallowing `http://` for all repositories by default -- looks like planned for yarn v2.
* Yarn fixing `yarn.lock` files automatically to replace `http://` links to npm registry with `https://` links.
* Automatic revocation of all tokens that were transfered to npm registry via unencrypted connection on npm registry side.

## Footnote

Don't forget to enable 2FA on npm (and everywhere else). Do it now. For both logins and publishes.

---

Published: 2019-07-13 10:30 UTC.

If you have any questions to me, contact me over Gitter ([@ChALkeR](https://gitter.im/ChALkeR)).
