## These are the things I would like to keep in mind or introduce in the code review process at my workplace.
### Experimental, subject to change.

Main source: https://medium.com/palantir/code-review-best-practices-19e02780015f

### How big should be my PR?
* Ideally - 20 minutes for review, 5 files
* Big PRs can be divided:
  - Make one PR with 'core' functionality and then other PRs with smaller changes related to it. E.g. if the task is
    to display the customers list with filters, then one PR can include displaying the list, and the other for filters.
  - One branch can be made for the task, and from there other branches can be created for more specific parts (task/12345/big-change, task/12345/big-change-api).
    Then, we can have PR for each specific part, that will be merged into main task branch.

### What to focus on?
 * More on logic, than on style, format etc.
 * Try to follow the flow of the code, not just a list of files
 * Distinct in the comments required changes, suggestions and questions
 * Feedback can be given on commit messages and PR's size too! (We all should take care of codebase readability, should we?)
 * Think about how you would solve the problem
 * Is the goal clear and documented? If the jira ticket does not contain full description, maybe it can be added at least in the PR description?
 * **Read the tests**. Are they understandable? Do they cover all the cases?
 * _When you’re done with a code review, indicate to what extent you expect the author to respond to your comments and whether you would like to re-review the CR after the changes have been implemented (e.g., “Feel free to merge after responding to the few minor suggestions” vs. “Please consider my suggestions and let me know when I can take another look.”)._
 
### How to prepare my PRs?
 * Rember about self review and testing before opening PR
 * Respond to all comments (easier to check if something was omitted)
 
