# future-ideas
My ideas about future possibilities in software engineering

## Lightning Deployments
* What: Apply changes to production in less than a second.
* Why: You want to be able to evolve/fix your app as quickly as possible.
* How:
  * Deployment is just uploading a script.
  * Nearly all commits are script changes anyway.
  * Confidence can be gained via staggered canary deployments to robots and humans.

## Command Loop
* What: A design pattern for orchaestrating functional applications
* Why: 
  * Ability to test orcahestration of functional core and imperative shell.
  * Having a uniform interface for unit testing helps tooling.
* How: 
  * An application is a recursive function:
```
function app = (Command cmd) {
  if cmd.next === EXIT return
  return app(process(cmd))
}
```
* Notes:
  * `process` returns the next command based on the current command.
  * ALL unit tests are of form: `Given a command, when it is processed, then a command another is returned`
  * Commands can be imperative or functional. Processing imperative commands should have a cycomatic complexity of one.
  
  
