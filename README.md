# future-ideas
My ideas about a possible future of software engineering. I feel these are all achievable with today's technology.

## High-level ideas
* Deploy your code to a production environment in less than a second.
* Remove all environments except for production (even local development).
* Automatically generate comprehensive fuzz/boundary/load tests that continuously execute.
* Keep historic suite of failing tests that continuously execute.
* Your IDE visualises production data flowing through your application's code paths (shows density etc).
* You can "breakpoint" live production data (non-blocking, a replica of the real data that flowed through).
* Code in your IDE automatically updates with other people's changes.
* Production incidents will automatically generate pull request with a failing unit test.
* The application will attempt to self-heal by figuring out how to pass the failing test.

## Concepts to achieve this

## The Command Loop
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
  
  
