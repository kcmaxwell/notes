# Introduction to CI/CD

During this part, you will build a robust *deployment pipeline* to a ready made example project starting in exercise 11.2. You will fork the example project and that will create you a personal copy of the repository. In the last two exercises, you will build another deployment pipeline for some of *your own* previously created apps.

## Getting software to production

Writing software is all well and good, but nothing exists in a vacuum. Eventually, we'll need to deploy the software to production, i.e. give it to the real users. After that, we need to maintain it, release new versions, and work with other people to expand that software.

We've already used GitHub to store our source code, but what happens when we work within a team with more developers?

Many problems may arise when several developers are involved. The software might work just fine on *my computer*, but maybe some of the other developers are using a different operating system or different library versions. It is not uncommon that a code works just fine in one developer's machine, but another developer cannot even get it started. This is often called the "works on my machine" problem.

There are also more involved problems. If two developers are both working on changes and they haven't decided on a way to deploy to production, whose changes get deployed? How would it be possible to prevent one developer's changes from overwriting another's?

In this part, we'll cover ways to work together and build and deploy software in a strictly defined way so that it's clear *exactly* what will happen under any given circumstance.

## Some useful terms

In this part, we'll be using some terms you may not be familiar with, or you may not have a good understanding of. We'll discuss some of these terms here. Even if you are familiar with the terms, give this section a read so when we use the terms in this part, we're on the same page.

### Branches

Git allows multiple copies, streams, or versions of the code to co-exist without overwriting each other. When you first create a repository, you will be looking at the main branch (usually in git, we call this *master* or *main*). This is fine if there's only one developer for a project and that developer only works on one feature at a time.

Branches are useful when this environment becomes more complex. In this context, each developer can have one or more branches. Each branch is effectively a copy of the main branch with some changes that make it diverge from it. Once the feature or change in the branch is ready, it can be *merged* back into the main branch, effectively making that feature or change part of the main software. In this way, each developer can work on their own set of changes and not affect any other developer until the changes are ready.

But once one developer has merged their changes into the main branch, what happens to the other developers' branches? They are now diverging from an older copy of the main branch. How will the developer on the later branch know if their changes are compatible with the current state of the main branch? That is one of the fundamental questions we will be trying to answer in this part.

You can read more about branches from here:
https://www.atlassian.com/git/tutorials/using-branches

### Pull request

In GitHub, merging a branch back to the main branch of software is quite often happening using a mechanism called pull request, where the developer who has done some changes is requesting the changes to be merged to the main branch. Once the pull request, or PR as it's often called, is made or *opened*, another developer checks that all is ok and *merges* the PR.

### Build

The term "build" has different meanings in different languages. In some interpreted languages such as Python or Ruby, there is actually no need for a build step at all.

In general, when we talk about building, we mean preparing software to run on the platform where it's intended to run. This might mean, for example, that if you've written your application in TypeScript, and you intend to run it on Node, then the build step might be transpiling the TypeScript into JavaScript.

This step is much more complicated (and required) in compiled languages such as C and Rust where the code needs to be compiled into an executable.

In part 7, we had a look at webpack, which is the current de facto tool for building a production version of a React or any other frontend JavaScript or TypeScript codebase.

### Deploy

Deployment refers to putting the software where it needs to be for the end-user to use it. In the case of libraries, this may simply mean pushing an npm package to a package archive (such as npmjs.com) where other users can find it and include it in their software.

Deploying a service (such as a web app) can vary in complexity. In part 3, our deployment workflow involved running some scripts manually and pushing the repository code to Fly.io.

In this part, we'll develop a simple "deployment pipeline" that deploys each commit of your code automatically to Fly.io **if** the committed code does not break anything.

Deployments can be significantly more complex, especially if we add requirements such as "the software must be available at all times during the deployment" (zero downtime deployments) or if we have to take things like database migrations into account. We won't cover complex deployments like those in this part, but it's important to know that they exist.

## What is CI?

The strict definition of CI (Continuous Integration) and the way the term is used in the industry are quite different.

Strictly speaking, CI refers to merging developer changes to the main branch often. Wikipedia even helpfully suggests "several times a day". This is usually true, but when we refer to CI in industry, we're usually talking about what happens after the actual merge happens.

We'd likely want to do some of these steps:
- Lint: to keep our code clean and maintainable
- Build: put all of our code together into software
- Test: to ensure we don't break existing features
- Package: Put it all together in an easily movable batch
- Upload/Deploy: Make it available to the world

We'll discuss each of these steps (and when they're suitable) in more detail later. What is important to remember is that this process should be strictly defined.

Usually, strict definitions act as a constraint on creativity/development speed. This, however, should usually not be true for CI. This strictness should be set up in such a way as to allow for easier development and working together. Using a good CI system (such as GitHub Actions that we'll cover in this part) will allow us to do this all automagically.

## Packaging and Deployment as a part of CI

It may be worthwhile to note that packaging and especially deployment are sometimes not considered to fall under the umbrella of CI. We'll add them in here because in the real world, it makes sense to lump it all together. This is partly because they make sense in the context of the flow and pipeline, and partially because these are in face the most likely point of failure.

The packaging is often an area where issues crop up in CI as this isn't something that's usually tested locally. It makes sense to test the packaging of a project during the CI workflow, even if we don't do anything with the resulting package. With some workflows, we may even be testing the already built packages. This assures us that we have tested the code in the same form as what will be deployed to production.

What about deployment, then? We'll talk about consistency and repeatability at length in the coming sections, but we'll mention here that we want a process that always looks the same, whether we're running tests on a development branch or the main branch. In fact, the process may *literally* be the same, with only a check at the end to determine if we are on the main branch and need to do a deployment. In this context, it makes sense to include deployment in the CI process, since we'll be maintaining it at the same time we work on CI.

### Is this CD thing related?

The terms *Continuous Delivery* and *Continuous Deployment* (both of which have the acronym CD) are often used when one talks about CI that also takes care of deployments. In general, we refer to CD as the practice where the main branch is kept deployable at all times. In general, this is also frequently coupled with automated deployments triggered from merges into the main branch.

What about the murky area between CI and CD? If we, for example, have tests that must be run before any new code can be merged to the main branch, is this CI because we're making frequent merges to the main branch, or is it CD because we're making sure that the main branch is always deployable?

So some concepts frequently cross the line between CI and CD, and as we discussed above, deployment sometimes makes sense to consider CD as part of CI. This is why you'll often see references to CI/CD to describe the entire process. We'll use the terms "CI" and "CI/CD" interchangeably in this part.

## Why is it important?

