# beats me

[![Netlify Status](https://api.netlify.com/api/v1/badges/ce63dce5-a334-4834-95bc-675fe4c234eb/deploy-status)](https://app.netlify.com/sites/beats-me/deploys)

This is a small proof of concept app that wraps up a signal processing method of beat detection (most common lowpass frequency) in a small client-side web-app. It's nothing new - mostly me trying out Svelte + Tailwind for the first time, and seeing how efficient this algorithm can get (within a second or so for a reasonable song length, _in JavaScript_). It's also surely not a great example of good coding practice, etc.

Relevant resources:

- libraries used: [SvelteKit](https://kit.svelte.dev/), [Tailwind](https://tailwindcss.com/), [web-audio-beat-detector](https://www.npmjs.com/package/web-audio-beat-detector)
- related blog posts: [Joe Sullivan's original post](http://joesul.li/van/beat-detection-using-web-audio/), [José M. Pérez's follow-up](https://jmperezperez.com/blog/bpm-detection-javascript/)

All the packaged songs are ones that I've arranged in GarageBand; I'm intentionally not uploading copyrighted music.

Eventually, I'm thinking of:

- re-implementing the node package myself
- re-implementing the algorithm in WASM (but this seems unnecessary)
- putting it up behind a server, so other clients can access it on-demand

thanks for taking a look :)
