---
title: "Adding blog comments to your static site with utterances"
date: "2021-11-06"
updated: "2021-11-06"
categories:
  - "web"
  - "svelte"
  - "javascript"
coverImage: "utterances.png"
coverWidth: 16
coverHeight: 9
excerpt: The web world is full of tradeoffs. Going from a CMS to a static site keeps things simple—but that simplicity comes with costs. Luckily, giving up comments on your blog doesn't have to be one of them.
---
<script>
  import Callout from '$lib/components/Callout.svelte'
  import SideNote from '$lib/components/SideNote.svelte'
</script>

The web world is full of tradeoffs. As I wrote in my post about [moving away from WordPress](/blog/goodbye-wordpress), going from a CMS to a static site keeps things simple. That simplicity, however, comes with costs—one of which is the ability to have comments on blog posts. 

I had to throw away all the existing comments on my blog when I moved away from WordPress. (Not that there were a lot; most of them were from ages ago, and on my [Pantone post](/blog/pantone), which somehow retains considerable SEO juice.) Due to the nature of static sites generally not having a database or a server to process data, there are few good, simple ways to allow user comments.

There are plenty of options out there to solve this problem, of varying degrees of simplicity. But I've recently settled on a nifty little GitHub-based library called [utterances](https://utteranc.es).


## What is utterances, and what does it do?

Have you ever been on a website and found a Facebook-style comments section, which allowed you to add comments to the content using your Facebook login?

If so, just know that utterances is _that_, but with GitHub instead of Facebook.

The utterances [documentation and demo](https://utteranc.es/) covers the topic pretty well. (In fact, it's so concise, this post will likely be much longer than the whole site.) But to summarize: utterances is a tiny script that runs on the page to display a comments form, along with any comments that have been made on the page already. Behind the scenes, this is all powered by GitHub--and specifically, GitHub issues.

When a user creates a new comment (which, it should be noted, they must be logged into GitHub to do), if there are no comments yet, utterances will create a new GitHub issue for the current page, and the user's comment becomes the first comment on that issue. Any new comments will appear as further comments on the same issue (so that you ideally only have one "issue" per route). Whenever your page loads, utterance will go and grab all of those comments and plop them onto the page in sequential order.

<Callout>
Utterances adds GitHub-powered comments to your site, simply and easily.
</Callout>

You don't really need to know anything about GitHub issues, or even that GitHub issues is the engine under the hood. (After all, these comments aren't really issues at all; they're just a convenient way to store data associated with your repo and in the same place as your code.)

All you need to know is: utterances adds GitHub-powered comments to your site, simply and easily.


## How to set up utterances

Again, the [utterances site](https://utteranc.es/) covers this nicely, so I'll just hit the high notes here:

1. **Make sure your site's GitHub repo is public, not private.** Since utterances is really GitHub issues under the hood, the public needs to be able to read those issues in order for it to work.

2. **Be sure to [enable the utterances app](https://github.com/apps/utterances) in GitHub**. You have the choice of whether to enable it for _all_ of your repos, or to pick and choose. (I personally just selected this site's repo only, but the choice is up to you.)

  Note that you also need to be sure that issues are enabled in the repo's settings, particularly if the repo in question is a fork of another one. That option can be found on the first page in the repo's "Settings" tab, near the top.

<SideNote>
Even if you choose to enable the utterances app for all your repos, you'll still have to put the utterances code on each site individually; simply enabling the app doesn't do anything on its own.
</SideNote>

3. **Finally, add the script snippet to your site**. We'll dig into this a bit more next, since--while not too complex--it's the area that gave me the most trouble.


### Adding utterances to the page

The last step of the process is to add a small script (which, you may be happy to know, includes no tracking) wherever the comments form should appear. That script will look a little something like this:

```html
<script
  src="https://utteranc.es/client.js"
  repo="github-name/repo-name"
  issue-term="pathname"
  theme="github-light"
  crossorigin="anonymous"
  async>
</script>
```

The `theme` option controls the appearance of the form, and the `issue-term` controls how the issue will be named in your repo. There are others, too, such as auto-tagging. Once more, be sure to have a look at the [utterances site](https://utteranc.es/) for full details.

But in any case, the idea is: you'll state your preferred options inline as part of a script tag, and then drop that script into the HTML wherever you want your comments to appear. 

This is simple enough when actually working with straight HTML, but it poses a small challenge in SvelteKit (and likely in some other frameworks, too), where `<script>` tags have special meaning. Most frameworks have rules about where you can just sling `script` tags. In fact, the Svelte compiler will yell at you for it.

The workaround is a _little_ verbose, but simple enough; the more Svelte-y way of handling it (and in fact, utterances itself) was brought to my attention via this tweet from [@sarah11918](https://twitter.com/sarah11918):

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Spent my morning with <a href="https://twitter.com/jjcollinsworth?ref_src=twsrc%5Etfw">@jjcollinsworth</a> &#39;s <a href="https://t.co/DEaQ4JJ79m">https://t.co/DEaQ4JJ79m</a> and WHAT a treat! (Appreciated React comps approach.)<br><br>It&#39;s the perfect pairing for trying out <a href="https://twitter.com/james_r_perkins?ref_src=twsrc%5Etfw">@james_r_perkins</a> &#39;s instructions for adding comments to an <a href="https://twitter.com/astrodotbuild?ref_src=twsrc%5Etfw">@astrodotbuild</a> blog using <a href="https://twitter.com/sveltejs?ref_src=twsrc%5Etfw">@sveltejs</a>! <a href="https://t.co/xOn3GgUXwR">https://t.co/xOn3GgUXwR</a></p>&mdash; Sarah Rainsberger (@sarah11918) <a href="https://twitter.com/sarah11918/status/1456636003968561154?ref_src=twsrc%5Etfw">November 5, 2021</a></blockquote>

(Fun how a tweet about a post comes full circle and becomes a new post, eh?)

Credit to James Perkins for [the approach from his blog](https://www.jamesperkins.dev/post/supercharge-your-astro-blog), in which he uses the `onMount` hook to create a script, set its properties, and inject it just before a target `<div>`, all inside of a Svelte `<Comments />` component. This allows the component itself to be placed anywhere you'd like the comments form to show up.

I followed the spirit of his example closely, but changed it in a few ways. Here's my finished comments component (slightly simplified to remove some code irrelevant to this post):

```svelte
<!-- Comments.svelte -->
<script>
  import { onMount } from 'svelte'
  import { prefersDarkMode } from '$lib/data/store'

  const siteTheme = $prefersDarkMode ? 'github-dark' : 'github-light'

  const options = {
    src: 'https://utteranc.es/client.js',
    repo: 'josh-collinsworth/joco-sveltekit',
    label: 'comments',
    crossorigin: 'anonymous',
    theme: siteTheme,
    async: '',
    'issue-term': 'pathname',
  }

  onMount(() => {
    const utteranceScript = document.createElement('script')
    const tag = document.getElementById('utterances-comments')
  
    for (const prop in options) {
      utteranceScript.setAttribute(prop, options[prop])
    }

    tag.appendChild(utteranceScript)
  })
</script>

<div id="utterances-comments" />
```

The main differences are:

* I prefer to abstract the script attributes to an `options` object (and also, prefer descriptive variable names). While this makes the code longer, I feel it also makes it more readable (or at least, less repetitive);

* Since my site has two themes, I dynamically set the GitHub theme based on the user's site-level preference. (This site _does_ detect and respect the user's dark mode preference by default, but _also_ allows them to override it, just in case they like the opposite version here. So OS preference may or may not be site preference); and

* Finally, I put the script itself _inside_ the target `div`, rather than before it. This is mostly just to avoid having an empty div lying around, but it could also potentially help with styling. (The comments form itself is in an `iframe`, so you can't style it directly regardless, but at least this way you can have control over the wrapping `div`.)

To restate/emphasize, since I'm talking about somebody else's code here: this is all just personal preference. Both versions have tradeoffs, and either is perfectly fine.


## Utterances drawbacks

It's hard to complain about such a simple and effective solution, but as with all things, this approach comes with tradeoffs.

The main thing to remember is: since this is all powered by GitHub comments under the hood, a user needs a GitHub login in order to comment. I decided that's fine in my case (at least for now), since this blog is increasingly development-focused, but your needs and audience may vary. This probably wouldn't be a good approach for a non-technical audience.

Also, unfortunately, there's no commenting on other comments or threading.

Finally, I suppose you could consider it a drawback that your comments management moves to GitHub. Personally, I like that my comments are now hosted with the code itself, but I can see where going into GitHub to manage them could be undesirable in some cases. At the very least, it means you have less control over approving and moderating comments that you might with, say, WordPress. (That said, however: GitHub almost certainly has much better control over spam issues and comments than I'd ever be able to devise.)


## Conclusion

As you've seen, I'll be using utterances on this site for at least the time being. I like the idea of having comments here rather than trying to direct users to a contact form (though both work well). If you also like the idea or found this post useful, consider trying it out yourself. (Or, you know…leave a comment.) 😉