# Taming TypeScript autoimport in a monorepo

## Context

TypeScript's VSCode plugin doesn't use transitive dependencies for autoimport.

This means that, in a given subpackage, you have to list all the dependencies you want it to be able to autoimport.

That is, TypeScript will look in your `package.json`, _not_ your `node_modules` folder to know what to autoimport.

Below is a workflow that manages this for you with ease, without having clashing versions or trouble installing.

## Instructions

```sh
yarn add -D -W lerna
yarn add -D -W lerna-update-wizard
yarn lerna init
```

Add this in `lerna.json`:

```json
{
  "packages": ["packages/*", "apps/*"],
  "version": "0.0.0",
  "npmClient": "yarn",
  "useWorkspaces": true
}
```

This setup assumes we have a layout like this:

- `packages/`
- `apps/`

Adjust the `packages` field in `lerna.json` to match your folders.

Same goes for the `package.json` in the root of your repo:

```json
{
  "workspaces": ["packages/*", "apps/*"]
}
```

You can also add a convenience script to your root `package.json` to install packages:

```json
{
  "workspaces": ["packages/*", "apps/*"],
  "scripts": { "plus": "lernaupdate", "p": "lernaupdate" }
}
```

And use it like so:

```sh
yarn plus
# wait for the prompt to ask you which package
```

## Install or update a package

Instead of `yarn add` inside of packages, use `yarn lernaupdate` at the root of your repo. This will be more efficient too, since it only needs to run `yarn install` once under the hood.

Then, type the package you want to install/update.

When it asks you which packages you want to put it in, check each one.

For every package that isn't an app (i.e. in your subpackages), select `devDependency`. If you're using an open source project, this may be different, so you should know what kind of dependency it is. But if you're just using a monorepo for an internal app for organization purposes, `devDependency` should be fine.

## Confirm it worked

Say you just installed `dripsy` at `^3.5.3`.

```sh
yarn why dripsy
```

This should give you `^3.5.3`, hoisted by each package you installed it in. But there shouldn't be any duplicates.

Now, open a file in one of your subpackages. You should be able to autoimport from `dripsy`.

## Related

I found this solution after asking a question here: https://github.com/microsoft/TypeScript/pull/38923#issuecomment-970509969

Thanks to [@andrewbranch](https://github.com/microsoft/TypeScript/pull/38923#issuecomment-970585424) for helping me understand how TS deals with transitive dependencies.
