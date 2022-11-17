---
title: Supercharge your Build Tooling
published: true
description: Supercharge your build tooling through automation and monorepo management.
tags: tutorial, tooling, javascript, monorepo
series: Supercharge your Build Tooling
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/f1nhmuqu16f9z29wks20.jpg
---

# Supercharge your Build Tooling

Have you ever wondered if there was an easier way to manage your tooling across projects?

With so many tools in the frontend ecosystem to set up, it can be really time consuming and difficult to manage everything; taking time away you could be spending on actual development.

In this series, we'll cover approaches for managing tooling and how you can implement this yourself to increase your productivity. I'll be using [my own personal setup](https://github.com/waldronmatt/shareable-configs) throughout the series as a reference.

## Frontend Tooling

Before we dive in, let's consider popular tooling a developer will need for frontend projects and how important it will be to manage them more efficiently.

- `Git Hooks` help automate the development lifecycle and tools like `Husky` help make setting this up easier across different systems.
- Tools like `Commitizen` help standardize commit message rules with user friendly prompts and `Commitlint` to lint commit messages
- `Lint-Staged` helps automate code linting on changed files after commits
- `Stylelint` lints styling, `Postcss` provides a variety of plugins to transform css, and `Sass` helps scale `css` with helpful features.
- `TypeScript` provides type-safety for projects
- `Eslint` lints your `JavaScript` and `TypeScript` code.
- `Prettier` enforces consistent styling and across your entire codebase and provides linting
- `Jest` is a popular testing framework
- `Browserslist` defines your supported browsers/devices
- `Secretlint` helps prevent accidently committing credentials
- Tools like `HTMLHint` and `Markdownlint` lint `html` and `md` files
- `Semantic Release` automates versioning and publishing

That was a lot! But our tooling isn't limited to the list above. You might be using a framework or a completely different setup that will require its own set up of tools which is why it's important to think of ways to make managing this easier.

## Extendable Configurations

Luckily for us, many of the tools above provide ways to create our own custom configurations and extend them by reference.

For example, we might define our own `eslint` configuration:

**`index.js`**

```js
module.exports = {
  // my custom eslint setup
};
```

But how do we reference this in our own projects? We'll want to publish this somewhere like `npm` so we can download anywhere and install in our project.

In our project's `.eslintrc.js` file we can define our configuration like so:

**`.eslintrc.js`**

```js
module.exports = {
  extends: "@[unique-scoped-name]/eslint-config",
};
```

[See eslint's guide](https://eslint.org/docs/latest/developer-guide/shareable-configs) for more information.

## Versioning and Publishing

So far so good, but what if we need to make a change to our shareable configuration? We'll don't want to manually bump our `package.json` version and manually publish out to `npm`.

Let's automate! We can use a tool like `semantic-release` to do this for us:

**`.github/workflows/release.yml`**

```yml
...
- name: Release
  env:
	GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  run: npx semantic-release
```

And in our `.releaserc.json` file:

**`.releaserc.json`**

```json
{
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/changelog",
    "@semantic-release/npm",
    "@semantic-release/github",
    "@semantic-release/git"
  ],
  "branches": "main"
}
```

In this setup, we have semantic release automating versioning, publishing to `npm`, and changelog generation to help us keep track of changes.

## Monorepos

For publishing one or two packages, this setup works well. But what if we have several configurations to maintain? This setup will be hard to scale. We'll need to rethink our approach.

This is where monorepos come to save the day. A monorepo is like a regular repo, except that it can hold many projects. If we have a lot of configurations we can keep and maintain them in the same repository.

What if we want to automate versioning and publishing? Using `semantic-release` won't work in a monorepo setup. But with a monorepo tool like `Lerna` we can leverage it to version and publish multiple changed packages.

We can also use a tool called `Yarn Workspaces` to optimize and link different packages together. While `Lerna` can do this too, `Yarn Workspaces` offers improved performance.

## Conclusion

After reviewing our options, a more ideal setup for managing multiple configurations involves using a monorepo structure to manage, version, and publish our packages.

A common monorepo structure for our configurations will look like the following:

```bash
/
	.github/
		# github CI/CD files
	packages/
		eslint-config/
		stylelint-config/
		...
	package.json
	lerna.json
	...
```

In part 2 of this series, we'll go into detail on how to set up this repository. I'll be basing this off my own [personal shareable configuration monorepo](https://github.com/waldronmatt/shareable-configs). Follow for updates on new article releases!
