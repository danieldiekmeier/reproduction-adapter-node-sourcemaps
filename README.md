# SvelteKit Reproduction

When you turn on sourcemaps in Vite, SvelteKit will generate sourcemaps even for the server. This is great! But during the build, `@sveltejs/adapter-node` moves the generated files, for example from `.svelte-kit/output/server/foo.js` to `.svelte-kit/adapter-node/foo.js`, breaking the relative paths to the sources in the sourcemaps.

This is the code that moves the files: https://github.com/sveltejs/kit/blob/51f3e668ca5c433de0215cdfd627fc06fd030d47/packages/kit/src/core/adapt/builder.js#L207-L209

I noticed this because I wanted to setup Sentry's new SvelteKit integration, but the sourcemaps would just not work. Sentry is actually using [sorcery](https://www.npmjs.com/package/sorcery) to try to flatten all the sourcemaps from the different steps of the build: https://github.com/getsentry/sentry-javascript/blob/41fef4b10f3a644179b77985f00f8696c908539f/packages/sveltekit/src/vite/sourceMaps.ts#L136-L139

This almost works, except that `adapter-node` moves the files, thus breaking the chain.

A fix could be to update the source maps while copying them. (But maybe there are better options.)

## Steps to reproduce

```sh
pnpm i
pnpm build
diff .svelte-kit/output/server/chunks/index.js.map .svelte-kit/adapter-node/chunks/index.js.map
# nothing will be printed, which is the problem: it's the same file
```
