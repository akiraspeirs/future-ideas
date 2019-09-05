# future-ideas
My ideas about a possible future of software engineering. I feel these are all achievable with today's technology.

## High-level ideas
* Your IDE visualises production data flowing through your application's code paths (shows density etc).
* You can "breakpoint" live production data (non-blocking, a replica of the real data that flowed through).
* Production incidents will automatically generate pull request with a failing unit test.
* The application will attempt to self-heal by figuring out how to pass the failing test.
* Automatically generate fuzz/boundary/load tests that continuously execute.
* Keep historic suite of failing tests that continuously execute.
* Deploy your code to a production environment in less than a second.
* Remove all environments except for production (even local development).
* Code in your IDE automatically updates with other people's changes.

## Concepts to achieve this

### The Command Loop
* What: A pattern for standardising the processing and testing of data.
* Why: 
  * Facilitates automatic generation test data.
  * Facilitates visualisation of production data within the IDE.
* How: 
  * The top level of an application must follow the following convention:
```
interface Command {
  data: data,
  next: () => Command
  toString: () => string
  fromString: (string) => Command
}

function app = (Command cmd) {
  log(cmd.toString())
  if !cmd.next return
  return app(cmd.next())
}

app(initialCmd)
```
* Notes:
  * Application recursively executes commands until there is no next command to execute.
  * All commands (except the final command) must have a next function that returns the next Command.
  * ALL unit tests are of form: `Given a command with data, when next is called, it returns a new command with expected data`.
  * Commands can be imperative or functional.
  * Imperative commands should have no branching logic.
  * The application logs all commands it executes (removing sensitive data).
  * Commands have a `fromString` function that allows commands to be reproduced from logs.
  
### Robot Canaries
* What: A staggered canary process that starts with a swarm of robots.
* Why:
  * Give a large level of confidence to deploy anything to a production environment (including broken code).
  * Robots will replace E2E tests, but we don't need to write tests, just monitor errors like with all other users.
  * Scales horizontally to more robots.
* How:
  * Use immutable architecture to deploy complete and independant stacks into production with a Blue/Green strategy.
  * Use a staged canary process using robot whitelisting > 5% > 50% > 100% (or something).
  * Revert to previous stack if error rates are too high at any stage of canary process.
 
  
