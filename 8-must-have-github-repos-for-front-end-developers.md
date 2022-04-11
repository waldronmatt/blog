---
title: 8 Must-Have GitHub Repos for Front-end Developers
published: true
description: Tips and tricks to set up tools, automate workflows, and standardize projects
tags: codenewbie, beginners, webdev, productivity
cover_image: https://images.unsplash.com/photo-1618401471353-b98afee0b2eb?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1188&q=80
---

# 8 Must-Have GitHub Repos for Front-end Developers

One of the most challenging aspects of front-end development is maintaining configurations and dependencies used in the projects. In this guide, we will cover essential repository tips and tricks to help set up your tools, automate your workflow, and standardize your projects. At the end, we'll use everything we've covered to market your skills.

### Why Write This Article?

This is the article I wish I had when I started front-end development. I've gone through countless ways of how to best organize and maintain my projects. This article is the result of much learning and experimentation that has culminated in an efficient workflow that is useful for me and I hope that it can be useful for you too.

**Note**: While this article is for beginners, I do cover a lot of advanced topics that are covered quickly at a very high level. Don't feel discouraged if you don't understand everything right away. It's okay! Remember to take everything step-by-step.

## Prerequisites

If you haven't created a GitHub account yet, you can follow the link [here](https://github.com/join) to get started.

Before we begin, I recommend learning Git version control; an essential tool to manage your code and track changes. More information can be found [here](https://git-scm.com/). A cheat sheet can be found [here](https://about.gitlab.com/images/press/git-cheat-sheet.pdf) that covers the bulk of commands you'll use on a regular basis starting out.

## 1. The Dotfiles Repository

If you've started working on projects, you've likely come across files like `.gitignore`, `.editorconfig`, etc.

The `.gitignore` file contains a list of files/directories you want ignored from version control. The `.editorconfig` file contains configurations to maintain consistent coding styles between code editors.

For me personally, these files don't change often from project to project. I want a way to store these files, copy them easily to new projects, and maintain them using version control.

The simplest solution is to create a regular repository and commit the dotfiles to GitHub. Simply copy/paste the files to a new project when you want to use them. In some cases, however, other dotfiles might get read by tools in a specific directory; commonly your home directory on your machine.

For these situations, it might be useful to create a **bare repository**. A bare repository is a repository that contains only the `.git` folder; the folder containing version control history while a regular git repo contains both the `.git` and project files.

Why is this important? With a bare repository, we can leave the dotfiles where they are so that they'll get picked up by tools that use them while our version control history lives in a separate folder.

Want to create your own? You can [follow the directions here](https://github.com/waldronmatt/dotfiles) in my personal dotfiles repo. I have instructions to set up for `linux`, `mac`, and `windows` systems. If you run into issues with my directions, please [submit an issue](https://github.com/waldronmatt/dotfiles/issues)!

In my personal dotfiles, I opted to keep things relatively simple. I have a `.gitignore`, a global `gitignore`, `.editorconfig`, a default MIT `LICENSE` I use for projects, and a basic `README` template. I also use a simple script to automate my bare repo commands and file copying.

There are a near-infinite number of ways to configure your own dotfiles. In fact, there is a huge community built around them. This [dotfiles resource](https://dotfiles.github.io/) is your one-stop guide for everything dotfiles.

## 2. The `.extra` Repository

Popularized by [this css-tricks article](https://css-tricks.com/using-dotfiles-for-managing-development-and-many-other-magical-things/#aa-protect-your-git-credentials-extra), the `.extra` repository is useful for hiding information you don't want to share publicly in one/several files in a private repo.

Examples of files you might consider making private include `.gitconfig` which contains sensitive information such as your personal email. If this file was in your public `.dotfiles` repository and someone forked and used it, they could accidentally commit changes as you.

In my personal `.extra` repo I have my `.gitconfig` and a resume file containing contact information I don't want public.

Keep in mind that these files won't benefit from your main `.dotfiles` setup, but more importantly, your sensitive configurations will be protected.

## 3. The `.github` Repository

When creating repositories, it's important to include community files which are a standard set of files that provide additional context, help, and safety to users or collaborators. You may have already seen examples of these files in other projects such as `CONTRIBUTING` and `CODE_OF_CONDUCT`. More information on community health files can be [found here](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file#about-default-community-health-files).

Creating a new GitHub repository and naming it `.github` creates a special repository where you can add default community health files for your repositories.

Once created, GitHub will automatically apply these files from the `.github` repo to all current and future public repositories. You can override this behavior on specific repositories if you create a community health file with the same name in the same directory.

You can easily create these community health files using the [amazing github template](https://github.com/dec0dOS/amazing-github-template) generator for your `.github` or generate for individual repositories.

My [personal .github repositories](https://github.com/waldronmatt/.github) uses modified files from that generator tool. When I go to the `Insights` tab, then `Community Standards` tab on the left on my public repositories, I can see that my files meet GitHub's recommended community standards found [here](https://opensource.guide/).

## 4. The Shareable Config Monorepo

One of the largest challenges in front-end development is managing dependencies and tooling configurations for projects. For example, you might use linters such as `prettier`, `eslint`, `stylelint`, and `commitlint` to help your code and commits conform to best practices.

Additional tools such as `Browserslist`, a tool to define which browsers are supported, `Postcss`, a tool to automate CSS operations, `Jest`, a JavaScript testing framework, `TypeScript`, a superset of JavaScript that adds static typing, `Webpack`, a JavaScript module bundler, and many others; all use configurations that can introduce a fair amount of complexity and time to set up for projects.

You can create your own shareable configurations to make this process easier. The idea centers around identifying your most commonly used settings for each tool and publish your own package to the NPM registry so you can install it in projects. Check out [this guide](https://zellwk.com/blog/publish-to-npm/) to learn how. This is great, but if we have a lot of tools to set up and publish individually to NPM, this creates a lot of overhead to maintain.

Here's where the monorepo comes in. A **monorepo** is just like a regular repository, except that it holds many individual projects. Monorepos are an ideal tool to use because you can manage all of your configurations in one place. A good introductory article on the subject can be found [here](https://medium.com/@jsilvax/a-workflow-guide-for-lerna-with-yarn-workspaces-60f97481149d).

My [shareable-configs](https://github.com/waldronmatt/shareable-configs) repository is a monorepo of all my shareable configurations for projects. I use `Lerna` and `Yarn Workspaces` to optimize my workflow and dependency management between packages. `Lerna` and `GitHub Actions` are used in conjunction to automatically version and deploy to the NPM registry and create GitHub releases. GitHub actions is also used to automate dependency updates and discover vulnerabilities. In each shareable configuration, I have a `postinstall` script that will auto-generate configuration files after I install them. You can fork and use my monorepo for your own shareable configs.

## 5. The Mighty Template Repository

So far, we've covered how to manage dotfiles, standardize community guideline files, and automate management of shareable configurations.
We'll use everything we've created to set up a repository we can use as a template for new projects. After you're done, enable the setting `Template Repository` under the `Settings` section of your repository so you can create new projects in GitHub using the template you created.

You can check out / use my personal [Webpack Template](https://github.com/waldronmatt/webpack-template). It is a comprehensive repository with server and serverless builds for Express and Netlify with automatic linting, versioning, GitHub releases, vulnerability scanning, and dependency management.

## 6. The Personal Portfolio Repository

No GitHub profile would be complete without your own personal website to show off your skills. With a lot of the hard work out of the way, we can create a new repository using the new template created in the previous step.

The rest is up to you! Your personal portfolio is a great way to showcase your skills in a unique way.

You can check out my [personal portfolio](https://github.com/waldronmatt/webpack-template) as an example that uses my [webpack template](https://github.com/waldronmatt/waldronmatthew.com) as a base. I use Netlify for hosting. Every time I make an update and deploy to my `main` branch, Netlify will run my build script and publish changes automatically.

## 7. The Resume Repository

I've spent countless hours on resume formatting over the years. After researching how to automate this process I discovered [jsonresume](https://jsonresume.org/), an open source initiative to create JSON-based resumes. Simply create a `.json` file containing your resume information, install the `resume-cli` package, [pick a theme](https://jsonresume.org/themes/), and then watch your resume generate into `.html` or `.pdf` automatically.

My [personal resume repository](https://github.com/waldronmatt/resume) uses a modified version of the [onepage theme](https://www.npmjs.com/package/jsonresume-theme-onepage) where I added support for a `Certifications` section. I also have an untracked file containing private contact information which is backed up on my private `.extra` repository.

## 8. The GitHub `README.md` Repository

Now that you've created your personal portfolio and resume, you should have a good idea of how you can best summarize your skills and accomplishments to recruiters. A good way to do this is to create a GitHub repository using the same name as your GitHub account name. This will unlock a special repository that will be viewable on your GitHub profile.

Have fun with this and show the world what you can do! You can see my [personal readme](https://github.com/waldronmatt/waldronmatt) in action here. For inspiration, check out [this awesome collection of readme repositories](https://github.com/abhisheknaiidu/awesome-github-profile-readme).

## Conclusion

That's it! I hope you enjoyed the article. You can find examples of the repositories covered on my [GitHub](https://github.com/waldronmatt).
