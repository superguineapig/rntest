# rntest

This repo is a "reproducer" for an issue filed with the `react-native` repository.

## Description

This represents a monorepo using `yarn workspaces`

There are 2 independent sample react native apps located in the `packages` directory.

The issue being demonstrated is the improper resolution of App-local dependencies by the `codegen` tools when invoked through the typical `pod install` process.

In this case, **App1** and **App2** delcare different versions of the same dependency.

**App1** uses an older version. **App2** uses a newer version.

yarn will hoist the older dependency to the root of the monorepo

```
/
  node_modules/
    *here*
```

The newer dependency is added to **App2** package directory
```
/
  packages/
    App2/
      node_modules/
        *here*
```

When invoking a `pod install` from `/packages/App2/ios` the underlying `codegen` (javascript) utility improperly sources the dependency from the repository **root** as opposed to the local **App2** package containing the correct version of the library.

This is made evident (coincidentally) via terminal output for codegen indicating a *deprecation warning* of the sourced library.

The old version of the lib is using deprecated codegen keys in its package.json; whereas the new version is using the appropriate syntax. Therfore, the warning from codegen reveals it has sourced the wrong dependency.

### Note
The dependency being used to test this issue is `@react-native-community/geolocation`

This issue is **NOT** about that library!

This issue appears to be related to the use of `require.resolve` in [generate-artifacts-executor.js](https://github.com/facebook/react-native/blob/8d3c4fb47514e0599a64dcb7197c26c75f9b0b7a/packages/react-native/scripts/codegen/generate-artifacts-executor.js#L241) when the project is hosted in a monorepo with locally installed node_module deps.

Node's require resolution is dependent on how the module was imported rather than the cwd of the running process. The module is imported via a ruby script invoking a node shell process.

## A Possible fix
A fix that I have tested locally is to replace `require.resolve` with the `find-up` utility (also used in other react native code) in order to traverse the directory tree upward from the specific project, allowing it to find local `node_modules` before reaching the root.

See FIXME for a patched version of `generate-artifacts-executor.js` Note, the `projectRoot` path needs to be threaded through to the `findExternalLibraries` function in order to always have a deterministic search start for the current project.
