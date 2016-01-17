# Let's fix Node.js Buffer API.

*Note: I am aware of various opinions on this issue. This post covers most of them in the [Q/A]() section. Please, read this whole document first before posting comments or expressing your opinion anywhere, it covers some issues that you might have missed.*

*Do not forget that all the relevant issues are not votings, so posting plain «+1», «me too»-s, or «I hate this» comments is generally counter-productive. If you have anything to say or propose, please, do that in a constructive manner.*

*Everything written below mostly describes my own point of view and proposal, the consensus has not been reached yet and there are no specific plans to implement this (at the time of writing this post).*

**Tl;dr: jump to [the proposed solution](#the-proposed-solution).** _Still, I strongly recommend reading the whole note before discussing it or expressing your opinion._

*Note that the current behaviour of `Buffer()` is a by-design decision and does not contain unexpected bugs on the Node.js side. The fact that `Buffer(number)` could contain sensitive data is properly [documented](https://nodejs.org/api/buffer.html#buffer_new_buffer_size), the issues come from the potential errors in user code that this API is causing easy to make.*

--

## Previous materials

Note: none of those describes any previously unknown vulnerability in Node.js, this is how the API was designed. The problem is in the fact that it's very easy to use the current API in an unsafe way, and there were various libraries hitting that (and probably many more still are).

 1. [Node.js Buffer knows everything](./Buffer-knows-everything.md), describing the possible consequences of using unitialized `Buffer`s.
 2. [Node.js issue #4660](https://github.com/nodejs/node/issues/4660) — Buffer(number) is unsafe.
 3. [Node.js pull #2586](https://github.com/nodejs/node/pull/2586) — buffer: introduce zero-fill option in constructor.
 4. [Node.js pull #2574](https://github.com/nodejs/node/pull/2574) — doc: minor clarification in buffer.markdown.
 5. [Node.js archive pull #9104](https://github.com/nodejs/node-v0.x-archive/pull/9104) — Documentation update about Buffer initialization.
 6. [`ws` release](https://github.com/websockets/ws/releases/tag/1.0.1) fixing it's vulnerability caused by inaccurate `Buffer(number)` usage.

   *Note: their release note contains severe mistakes, please check [this nodesecurity advisory](https://nodesecurity.io/advisories/67) instead.*
 7. [Node Security Project advisory 67](https://nodesecurity.io/advisories/67) on the above-mentioned `ws` issue.
 8. [Node Security Project advisory 68](https://nodesecurity.io/advisories/68) on the `bittorent-dht` inaccurate `Buffer(number)` usage.
 9. Private discussions.
 10. [Mongoose vulnerability](https://github.com/Automattic/mongoose/issues/3764) — assigning a number to the property that is `Buffer`-typed saves unitialized memory block to the DB. [POC](https://gist.github.com/ChALkeR/d4a8055625221b6e65f0).

## Is there a problem?

Yes, there is. In fact, there are two issues.

Note: while one can say that both of those are the fault of libraries and module authors, there is no point blaming anyone and trying to argue whoose fault is something, we should try best to make the ecosystem more secure and take steps in that direction. If it's possible to build the API in such a way that would reduce the security risks of the ecosystem modules — it should be done.

##### First one — inaccurate usage of `Buffer(number)` when it's not really needed.

Atm, the only method one can use to create a `Buffer` of a predefined length is `Buffer(number)`.

Note that the person using this method might not be aware of the security risks of leaking such a buffer, and might not be aware that he or she has to call `.fill(0)` on it to be on the safe side. Not everyone even reads the documentation (remember — no blaming, that's not constructive).

A better situation would be if the default method that an inexperienced coder will find and use will be the safe one, and the unsafe and more faster one should be used only by developers that actually read the docs, understand all the implications when optimizing for speed, and are confident that they are not leaking anything.

The possible outcome of inaccurate `Buffer(number)` usage is leaking everything, including http traffic, configuration, source code files, private crypto keys, passwords, etc. (That is not theoretical — see [this](./Buffer-knows-everything.md) for code samples).

##### Second one — accidental hitting of `Buffer(number)` when `Buffer(value)` was meant to be used instead.

This is exactly what happened to `ws` (see materials [above](#previous-materials)).

This is not some kind of a rare edge case (thanks to @joepie91 for clarifying that) — the only prerequisites for this to be exploited are:
 1. Accepting typed user input that could be a string or a number. Parsing some JSON received from the user does exactly that, as an example.
 2. Passing a value from that input to `Buffer(value)`.
 3. Failing to validate such input. I have lost count long ago how often does this happen.


While this gets catastrophic with non-zero-filled unsafe buffers that could contain anything, it would still be dangerous if those buffers were zero-filled.

For example, and attacker could make the server to quickly allocate very large buffers by just sending small numbers to it, and cause a denial of service.

The possible outcome of this is leaking everything when combined with the previous issue, and DoS, crashes, or bugs if the previous issue is fixed.

## Things that should be noted

At the first glance, it looks like the solution is quite straightforward — zero-fill all the `Buffer`s, but not everything is that simple. In fact, that would only make things _worse even from the security point of view_ short-term and will leave us with an insufficient API long-term.

When trying to fix the API in the most sane way, one should start with a question:

##### Considering everything that we know now, how would the API look like if it was designed anew, putting all the current ecosystem and situation aside?

_Zero-filling all the buffers does not answer this, because non-zero-filled buffers are required for experienced developers for performance reasons. When people are quite confident that they can handle `malloc()` in a safe way — it's better to give them that instrument and not argue that it shouldn't be used under any circumstances. Doing otherwise will only make the situation worse by making them resort to using native addons._

Now, once we answer the first question, the second question would be:

##### How do we get there in the best way?

Such way should try hard not to break existing code, should fix everything asap, and should avoid common pitfails.

_To illustrate such a pitfail, I would recall the «zero-fill `Buffer(number)`» statement once again._

_Making `Buffer(number)` zero-filled will solve the first part of the issue long-term, but will cause havoc short-term. For example, if Alice writes a lib, sees that with Node.js 6.x, 7.x etc `Buffer(number)` is zero-filled, and does not zero-fill the buffer in her code (she doesn't want to zero-fill it twice!), and Bob uses that lib under Node.js 5.x where `Buffer(number)` is not zero-filled by default — it will cause more damage that it solves. Even if zero-filling gets merged into 4.x or even 0.10.x/0.12.x — take into an account that Bob could use an older version._

_**Making `Buffer(number)` zero-filled would result in a complete disaster to everyone who is using an older version of Node.js.**_

_You can't just forget those users, security people should probably know how hard is it to get everyone to update._

_Documenting that in a way «zero-fills since 6.x, zero-fill it twice if you use a lower version» would not help, Alice could think «I don't need 5.x» and publish a lib that does not manually zero-fill `Buffer`s, but some other person could use that lib under 5.x and will be affected._

_There is no way around that — if `Buffer(number)` becomes zero-filled by default, the user will have to choose between zero-filling it two times in newer versions of Node.js or not zero-filling it at all in less recent versions of Node.js (and the third option would be — write some ugly version-detection code). Guess what will the user choose?_

So, the way to zero-filling `Buffers` by default is not nice in any sense.

## The proposed solution

*Note: the new API method names are not fixed (chosen) atm, I do not have a hard opinion on those. They could be changed after further discussion.*

##### First part — replace `Buffer(number)` with two methods, one of them producing a safe zero-filled buffer and another one a «fast» unsafe buffer.

 1. Introduce `Buffer.unsafe(number)` to allocate a non-zero filled Buffer, document it, try hard to note the possible security consequences of careless usage.
 2. Introduce `Buffer.safe(number)` to allocate a zero-filled Buffer.
 3. Soft-deprecate `Buffer(number)` (first in doc for at least a whole major release, perhaps), tell people to use `Buffer.safe(number)` instead.
 4. Introduce a command-line flag for opt-in to zero-filling all `Buffer(number)` calls, so that the topmost app could enforce zero-filling all Buffers. I think this point was already discussed and supported. I would recommend that flag to be clearly named to show that it only deals with `Buffer(number)`, not with `Buffer.unsafe(number)`. E.g. `--safe-buffer-number`.
 5. Hard-deprecate `Buffer(number)` in some future release, point people to `Buffer.safe(number)`. Perhaps make `Buffer(number)` zero-filled by default at the same time (but don't let people rely on that) and remove the above-mentioned command-line flag, because `Buffer(number)` being hard-deprecated would mean that the usage is low and no new code is expected to use that.

Note that the current target is specifying our future API and getting points 1-4 merged to the next major version. Everything else, including discussion on when to do actual hard-deprecation (point 5) and discussion on whether or not and how to backport points 1-4 are technical details that could be covered later and is currently out of scope.

Let's get our future API fixed first (e.g. merging 1-4 to the master branch), then discuss the exact roadmap on all the other things.

##### Second part — replace `Buffer(value)` with a separate method.
The primary intention of this is to better split between `Buffer(value)` and `Buffer(number)`, so that people won't end up accidentally calling `Buffer(number)` when they only wanted to to deal with `Buffer(value)` (like `ws` did).

 1. Introduce `Buffer.from(value)` that will do everything `Buffer(value)` did minus the `Buffer(number)`, so that people can use that method to be 100% sure that they won't accidentally hit `Buffer(number)`.
 2. Soft-deprecate `Buffer(value)` and point people to `Buffer.from(value)`.

This could be done on a separate schedule from the first part.

Thanks to @mafintosh for [noticing this](https://github.com/nodejs/node/issues/4660#issuecomment-171784186).

Note that even zero-filling `Buffer(number)` by default will not remove the need in this change, because `Buffer(number)` could be still used to cause a DoS in a simple way if the code is accidentally calling `Buffer(number)` instead of `Buffer(value)`.

The hard deprecation of `Buffer(number)` won't fix the second issue by itself — developers wouldn't notice the deprecation warning until it gets hit (which could be already late).

The only situation that will make this part unneeded will be the complete removal of `Buffer(number)`, and I estimate the probability of that happening anytime soon as 0.

While `Buffer(number)` remains present, the only way how the second issue could be fixed is by introducing a separate method that can't accidentally call a completely different one like `Buffer(value)` does.

## Q/A

_Note: there are a lot of repetitions in this section, that's mainly because people are asking very similar questions over and over again. I tried hard to cover those._

 1. **But it's not broken, what do you mean by «fixing it»?**
 
 There are two main problems with `Buffer()` now:
  1. Many people are not aware that `Buffer(number)` is unsafe and contains important things, it's unobvious that people should use `Buffer(number).fill(0)` to be on the safe side. The reason for this is that the method that a developed would undoubtedly use to allocate a `Buffer` of a fixed size has an unobvious pitfall.
  2. Someone could accidentally hit `Buffer(number)` when they want to call `Buffer(value)` — that's exactly what happened with the `ws` module. That's because two _very_ different methods are grouped under one name.
 
 You could argue that those problems are not with `Buffer`, but with libraries and authors — in this case, see point 15 below.
 
 2. **Why not just deprecate `Buffer` completely?**
  
  `Buffer` is actively used both internally and by a whole lot of modules out there, and they have various reasons to use `Buffer` over typed arrays. Thus, deprecating it will not happen anytime soon.
  
  Also note that deprecating `Buffer` completely will not provide users with a quick way to fix their code, while the current proposal will.
 
 3. **Why not just make `Buffer(number)` zero-fill everything by default?**
 
 While I first thought that making `Buffer(number)` zero-filled by default is a good idea, I do not think that now.

 Making `Buffer(number)` zero-filled will solve the main issue long-term, but will cause havoc short-term. For example, if Alice writes a lib, sees that with Node.js 6.x, 7.x etc `Buffer(number)` is zero-filled, and does not zero-fill the buffer in her code (she doesn't want to zero-fill it twice!), and Bob uses that lib under Node.js 5.x where `Buffer(number)` is not zero-filled by default — it will cause more damage that it solves.

 Documenting that in a way «zero-fills since 6.x, zero-fill it twice if you use a lower version» would not help, Alice could think «I don't need 5.x» and publish a lib that does not manually zero-fill `Buffer`s, but some other person could use that lib under 5.x and will be affected. There is no way around that — if `Buffer(number)` becomes zero-filled by default, the user will have to choose between zero-filling it two times in newer versions of Node.js or not zero-filling it at all in less recent versions of Node.js (and the third option would be — write some ugly version-detection code). Guess what will the user choose?
 
 Also note that this doesn't do anything with people accidentally hitting `Buffer(number)` instead of `Buffer(value)` (like `ws` did), and that could used to perform a DoS even if the resulting `Buffer` is zero-filled.
 
 4. **Why not just build a safe Buffer module and tell people to use that from npm?**
 
 This will not fix the ecosystem. Most people would either not hear about the shiny new module or will not use it.
 
 Also, the new module does not give any benefits to just improving core `Buffer` API — either the migration would be unobvious or it will have various issues noted here.
 
 Moving `Buffer` to a module is orthogonal to this issue and is out of scope of the current proposal.
 
 5. **It has worked like that for a long time, that is the documented behaviour of `Buffer`. Let's keep it that way.**
 
 Please, re-read the [reasoning behind this](#is-there-a-problem). Also, see point 15 below.
 
 6. **This issue was already raised multiple times, and each time the solution was not to change anything. What's different now?**
  
  With several vulnerabilities in popular packages that were recently discovered (including `ws`), this is the correct time to raise this issue again. Also, I believe that the current proposal has chances to be actually agreed upon, and that we can come to a resolution that will fix the `Buffer` API.
 
 7. **Don't break my `Buffer`s!**
 
 This proposal does not break anything immediately, it just proposes the future sane API and a soft (documentation-only) deprecation of `Buffer(number)` for the time being. How exactly this new API will be backported and when will the hard deprecation happen is out of scope of this proposal.
 
 8. **Why not redirect people to `Buffer.unsafe(number)` from the `Buffer(number)` deprecation message?**
 
 `Buffer.unsafe(number)` should be used only by those who are absolutely sure that they need it and are not leaking anything. 
  
  Pointing people from `Buffer(number)` to `Buffer.safe(number)` instead of `Buffer.unsafe(number)` is needed only because this way people who do not care or are not sure what they are doing will get the correct method. People who are experienced enough to work with unitialized Buffers and who care about performance would be able to actually read the docs and find `Buffer.unsafe(number)`. Anyways, it's unsafe to work with unitialized buffers without reading the docs on those first (those will contain more warnings).
 
 9. **Why not just an opt-in (e.g. command-line flag) for zero-filling all the `Buffer`s?**
 
 An opt-in will not fix the ecosystem — most setups will not know about it or will ignore it.
 
 Also, it will result in severe performance penalty — already safe code will receive double-zeroed `Buffer`s, hot code paths that were using unitialized buffers on a purpose will also receive zeroed `Buffer`s. Thus, an opt-in will introduce speed-vs-security tradeoff, and most managers will choose speed instead of security.
 
 Thanks to @joepie91 for [a nice clarification](https://github.com/nodejs/node/issues/4660#issuecomment-171449786) about this.
 
 10. **Why not just an opt-out (e.g. command-line flag) for zero-filling all the `Buffer`s?**
 
 See point 3. Also, there is no reason to introduce an opt-out that will definitely break things once people begin to rely upon `Buffer(number)` being zero-filled (and they will do that). Also, the speed-vs-security tradeoff argument from point 9 is still valid here.
 
 11. **Why not just a new API method that will construct zero-filled `Buffer`s?**
 
 That isn't better that suggesting everyone to use `Buffer(number).fill(0)` in any way. It does not fix existing code, it does not ensure that everyone gets the fix, it does not stop people from unsafe usage of `Buffer(number)` (and from hitting it accidentally). In fact, it's does not improve anything compared to the current situation. Note that everyone is already used to `Buffer(number)` and will likely not know about the new API.
 
 12. **Why is the `Buffer.safe(number)` needed? Could we deprecate `Buffer(number)` and add just the unsafe `Buffer.allocate(number)`? Everyone could use `.fill(0)` on those.**
 
 Don't forget that people would generally follow the path of least resistance, and that path should be the safe one. So the most straightforward and recommended way of allocating `Buffer`s should do that in a safe way to ensure that users use that by default, and resort to allocating unsafe `Buffer`s only if they are absolutely sure what they are doing and definitely need it. `Buffer.allocate(number).fill(0)` won't be obvious for everyone even if documented, and some people will forget that. Also, if `.fill(0)` is done later on a separate line — it would be harder to automatically check that (eg. using `grep`).

 13. **Why not make the API look like `Buffer.allocate(number, isSafe=true)` instead of introducing two separate methods?**

  The documentation on those methods should be separate, and it's very important. While `Buffer.safe(number)` could be documented as usually, `Buffer.unsafe(number)` documentation should describe all the dangers of using unitialized `Buffer`s. 
  
  Also, that flag changes the behaviour of a function in an unobvious way (I have seen people failing to understand all implications of using unitialized buffers). The names for these functions should better be self-describing.
  
  Merging two different methods with different behaviour into a one that changes it's behaviour depending on a flag isn't a good design move.
 
 14. **I am quite confident that I am experienced enough to use `Buffer(number)` in a safe way, and I absolutely need it.**
 
 Good for you, but this is actually hurting various projects where authors are using `Buffer(number)` inaccurately or hitting it accidentally. In fact, it might even affect you if you are using some of those libraries through a chain of dependencies. 
 
 Unitialized `Buffer` allocation is not going anywhere, you will have enough time to update your code to use the new safer API.
 
 15. **Everyone knows that `Buffer(number)` allocates raw memory, there is no problem with that!**
 
  I have seen people who were not aware, found issues in code, reported those, and talked with such people over Gitter. I remember someone being surprised. Also, at least one of Node.js active collaborators was surprised by the fact that `Buffer(number)` is not zero filled, afaik. Also, I guess @evilpacket have seen many such people.
 
 And don't forget that careless usage of `Buffer(number)` is just one part of the problem, the second part of the problem is hitting `Buffer(number)` by an accident  — that's what happened with `ws`. That's because two _very_ different methods are grouped under one name.
 
 While it is true that everything of those could be fixed on the packages level and those are technically bugs in some packages, it's the `Buffer` API design that allows that and makes that much more common, so yes, there is a problem.

 Remember — there is no point blaming anyone and trying to argue whose fault is something, we should try best to make the ecosystem more secure and take steps in that direction.

 16. **That will make things considerably slower, don't do that!**
 
 Please, read the proposal more carefully. The current proposal will not make anything slower — it will not change the `Buffer(number)` behaviour, just slowly deprecate it in favour of the new API that will provide the same functionality.
 
 It won't remove the alternative to use fast unitialized `Buffers` when the library author is sure what he or she is doing and knowingly wants it.
 
 It will give the library author the choice between using safe zero-filled `Buffer`s or potentially unsafe and fast non-zero-filled `Buffer`s, and will do that in a more obvious way (as it should have been done from the start).

 17. **Why deprecating `Buffer(value)`?**
 
 While deprecating `Buffer(value)` is not a crucial element of this proposal, it's still pretty important.
 
 Until `Buffer(number)` gets eradicated (which probably won't happen anytime soon) anyone could accidentally hit `Buffer(number)` when he or she wanted to call `Buffer(value)` (which happened with `ws`). That could be unnoticed even during the hard deprecation period if it's on a code path that needs special conditions to be executed (but could still be accidentally or maliciously reached).
 
 Note that even if that `Buffer(number)` will be zero-filled by this time, it could be still used to cause a DoS in a simple way.
 
 18. **How should we call the new API methods? E.g. should we call them `Buffer.alloc()`/`Buffer.zalloc()`?**
 
 While I do not have a hard opinion on the new method names, they should be definitely called in such a way so that an unexperienced developer will choose the safest of them without even looking into the docs.
 
 If you want `Buffer.alloc()` (or `Buffer.allocate()`) to be present, then it should allocate zero-filled buffers, and the non zero-filled one should be called something like `Buffer.allocUnsafe()`, `Buffer.allocRaw()`, or anything else like that.
 
 Most people should use the zero-filled variant. Only people who are confident what they are doing and who actually read the docs should use the non-zero-filled variant.
 
 If we fail to provide this, then there would still be a lot of people inadvertently using the non-secure version even when they don't actually need it — just because it was the one they found first.
 
 Taking the above considerations into an account, I am opposed to calling these new methods something like `Buffer.alloc`/`Buffer.zalloc` or `Buffer.alloc`/`Buffer.calloc`.
 
 Once again — the path of least resistance should be the secure one.

 19. **Can we make `Buffer` zero-filled by default and move the old behaviour to something like `UnsafeBuffer`?**
 
 That is esentially the same as zero-filling `Buffer(number)`, which has multiple problems that were covered above several times.
 
 See [things that should be noted](#things-that-should-be-noted) and point 3 of the current Q/A.

--

Published: 2016-01-15.

If you have any questions to me, contact me over Gitter (@ChALkeR) or IRC (ChALkeR@freenode).
