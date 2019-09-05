# future-ideas
My ideas about a possible direction of software development. I feel these are relatively achievable with today's technology.

## High-level goals
* Your IDE visualises live production data flowing through your application's code paths.
* You can "breakpoint" live production data (non-blocking, a replica of the real data that flowed through).
* Production incidents will automatically generate a pull request with a failing unit test.
* The application will attempt to self-heal by figuring out how to pass the failing test.
* Automatically generate fuzz/boundary tests that continuously execute.
* Keep a historic suite of failing tests that continuously execute.
* Deploy your code to a production environment in less than a second.
* Remove all environments except for production (even local development).

## Concepts to achieve this

### The Command Loop
* What: A pattern for standardising the processing and testing of data.
* Why: 
  * Facilitates automatic generation test data.
  * Facilitates standardised system to consume test data.
  * Facilitates visualisation of production data within the IDE.
* How: 
  * The top level of an application must follow the following convention:
```
//Main
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

//Test
function testCommand = (Command cmd, Command expectedNextCmd) {
  expectEquals(cmd.next(), expectedNextCmd)
}
//Run tests from templated data...
//Run tests from test data directories.
```
* Notes:
  * Application recursively executes commands until there is no next command to execute.
  * All commands (except the final command) must have a next function that returns the next Command.
  * ALL tests are of form: `Given a command (with data), when next is called, it returns the expected command (with data)`.
  * Tests can be driven from templated data or external data files.
  * Commands can be imperative or functional.
  * Imperative commands should have no branching logic.
  * The application logs all commands it executes (removing sensitive data).
  * Commands have a `fromString` function that allows commands to be reproduced from logs.
  * Use equivalient loop for languages without tail call optimisation.
  
### Robot Canaries
* What: A staggered canary process that starts with a swarm of robots.
* Why:
  * Give a large level of confidence to deploy anything to a production environment (including broken code).
  * We don't need to write any E2E tests. We just monitor errors like with all other users.
  * Scales horizontally to more robots.
  * Makes projects have robust error monitoring.
* How:
  * Use immutable architecture to deploy complete and independant stacks into production with a Blue/Green strategy.
  * Some robots test core user paths, others use randomised input.
  * Use a staggered canary process using robot whitelisting > 5% > 50% > 100% (or something).
  * Cancel rollout if error rates are too high at any stage of canary process.

## How we achieve the high-level goals
### Your IDE visualises live production data flowing through your application's code paths.
* A service will interpret all logged production commands and produced aggregated data about how often each command is called.
* An IDE plugin wil pull down this data and visualise this to the user.
* Logs will include a process identifier so you can visualise how commands flow on to other commands.
* Future iterations can look at how to represent branching within a command.

### You can "breakpoint" live production data (non-blocking, a replica of the real data that flowed through).
* An IDE plugin will pull down the latest available log for any given command.
* The plugin will use the command's fromString function to build the same command that was logged.
* The plugin will trigger the command's process function and pause execution.

### Production incidents will automatically generate a pull request with a failing unit test.
* A monitoring system will capture commands that produce an error and pass it to a service.
* The service will create a branch off master.
* The service will add comments to the branch with all information of the incident and links to logs.
* The service will commit a file with the failing log to the testData directory. Tests are data driven from files, so this will automatically be executed as a failing test.

### The application will attempt to self-heal by figuring out how to pass the failing test.
* After creating a failing test a self-healing service will be triggered.
* The self-healing service will perform code mutations on the failing command and attempt to make all tests for the command pass.
* If all tests pass it will add a comment using the mutated code and ask a developer to review, tweak and commit the fix.

### Automatically generate fuzz/boundary tests that continuously execute.
* Command data will have annotations to describe what sort of data they expect.
* A continuously running service will check out latest master branch commits, and generate random/boundary test data files based on the command data annotations.
* The unit tests will automatically pull in this data and we will be running fuzz/boundary tests continuously in the brackground.

### Keep a historic suite of failing tests that continuously execute.
* Commands that encountered production errors will be saved in a persistent data source.
* A continuously running service will check out latest master branch commits, and run these tests.

### Deploy your code to a production environment in less than a second.
* We don't run any tests before deploying!
* Errors will be captured by robot canaries users before any human users access the data.
* This ensures our error monitoring is extermely robust.

### Remove all environments except for production (even local development).
* If you want to test code, deploy a personal stack into production.
* You can use production data breakpointing and the other tools described above to debug in a live production environment.
* No more "but it works in my environment".
