# Deployment

Having written a nice application, it's time to think about how we're going to deploy it to the use of real users.

In part 3 of this course, we did this by simply running a single command from terminal to get the code up and running on the servers of Fly.io.

It is pretty simple to release software in Fly.io and Heroku, but it still contains risks: nothing prevents us from accidentally pushing broken code to production.

Next, we're going to look at the principles of making a deployment safely, and some of the principles of deploying software on both a small and large scale.

## Anything that can go wrong...

We'd like to define some rules about how our deployment process should work but before that, we have to look at some constraints of reality.

Murphy's law states that: "Anything that can go wrong will go wrong."

It's important to remember this when we plan out our deployment system. Some of the things we'll need to consider could include:
- What if my PC crashes or hangs during deployment?
- I'm connected to the server and deploying over the Internet, what happens if my Internet connection dies?
- What happens if any specific instruction in my deployment script/system fails?
- What happens if, for whatever reason, my software doesn't work as expected on the server I'm deploying to? Can I roll back to a previous version?
- What happens if a user does an HTTP request to our software just before we do deployment (we didn't have time to send a response to the user)?

These are just a small selection of what can go wrong during a deployment, or rather, things that we should plan for. Regardless of what happens, our deployment system should **never** leave our software in a broken state. We should also always know (or be easily able to find out) what state a deployment is in.

Another important rule to remember when it comes to deployments (and CI in general) is: "Silent failures are **very** bad!"

This doesn't mean that failures need to be shown to the users of the software, it means we need to be aware if anything goes wrong. If we are aware of a problem, we can fix it, if the deployment system doesn't give any errors but fails, we may end up in a state where we believe we have fixed a critical bug but the deployment failed, leaving the bug in our production environment and us unaware of the situation.

## What does a good deployment system do?

Defining definitive rules or requirements for a deployment system is difficult. Let's try anyway. Our deployment system should:
- be able to fail gracefully at **any** step of the deployment.
- **never** leave our sofware in a broken state.
- let us know when a failure has happened. It's more important to notify about failure than about success.
- allow us to roll back to a previous deployment. Preferably, this rollback should be easier to do and less prone to failure than a full deployment. Of course, the best option would be an automatic rollback in case of deployment failures.
- handle the situation where a user makes an HTTP request just before/during a deployment.
- make sure that the software we are deploying meets the requirements we have set for this (e.g. don't deploy if tests haven't been run).

Let's define some things we **want** in this hypothetical deployment system too:
- We would like it to be fast
- We'd like to have no downtime during the deployment (this is distinct from the requirement we have for handling user requests just before/during the deployment).
