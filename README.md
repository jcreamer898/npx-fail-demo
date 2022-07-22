# npx failure in symlink'd monorepo
This example using pnpm.

## Install
```bash
npm -g i pnpm
pnpm i
```

## Explanation
The `packages/all-signs-end` package has a dependency on `rimraf`, therefore it installs it into `packages/all-signs-end/node_modules` and creates a bin in `packages/all-signs-end/node_modules/.bin/rimraf`.

It seems like in `libnpmexec`, there may be a bit of a bug.

The manifest for `rimraf` gets found in the project here...
https://github.com/npm/cli/blob/latest/workspaces/libnpmexec/lib/index.js#L121

So, `needsInstall` is false here.
https://github.com/npm/cli/blob/latest/workspaces/libnpmexec/lib/index.js#L142

Then it appears like the assumption is that the binary being installed will have been put in `${rootDir}/node_modules/.bin` as the code inserts that path into the `PATH`...
https://github.com/npm/cli/blob/latest/workspaces/libnpmexec/lib/index.js#L204

However, the `rimraf` binary it needs to use is actually located at `packages/all-signs-end/node_modules/.bin/rimraf`. 

Then `_run()` calls `npm-run-script`, but it won't find the binary...
https://github.com/npm/cli/blob/latest/workspaces/libnpmexec/lib/index.js#L207

And it will throw the following error.

```
❯❯ npx-fail-demo  09:34 npx rimraf
'rimraf' is not recognized as an internal or external command,
operable program or batch file.
```

This error would probably also occur in `npm` isolated mode, `midgard-yarn-strict`, or any package manager which is not hoisting the `.bin` scripts.