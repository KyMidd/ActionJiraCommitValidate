# Jira Commit Validate

This Action reads all commits on a PR branch that deviate from a base branch, and checks each commit for a leading ticket number (at the beginning). It then checks to make sure that ticket number exists in a Jira instance you specify to make sure the ticket exists. 

You can (and should) block PRs from being merged if this Action fails - you can do so by creating branch policies to protect branches and requiring this Action succeeds in order to merge. 

If there is a failure, users should rebase their branch's commit history to contain only valid commits that link real Jira tickets. A great command to do this is: `git reset --soft  && git commit -a && git rebase master`. 

There's more information on this Action at Medium here: https://medium.com/@kymidd/lets-do-devops-github-action-check-if-each-commit-contains-valid-jira-ticket-id-63693def70a8

# Example call of this action

```
name: Jira Commit Checker

# Run on push to any branch 
on: [pull_request]

jobs:
  Jira_Commit_Checker:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
      with:
        fetch-depth: 0
        ref: '${{ github.event.pull_request.base.ref }}'

    - name: Prep
      run: |
        # Checkout branch
        git checkout -q ${{ github.event.pull_request.head.ref }}
        
        # Set variables
        BASE_BRANCH=${{ github.event.pull_request.base.ref }}

        # Write BASE_BRANCH to GITHUB_ENV so commit checker can use
        echo "BASE_BRANCH=$BASE_BRANCH" | tee -a $GITHUB_ENV

    - name: Jira Commit Checker
      id: jira_commit_checker
      uses: KyMidd/ActionJiraCommitValidate@v1
      with:
        jira-username: ${{ secrets.JIRA_COMMIT_CHECKER_USERNAME }}
        jira-api-token: ${{ secrets.JIRA_COMMIT_CHECKER_API_TOKEN }}
        JIRA_URL: 'https://foo.atlassian.net'
        BASE_BRANCH: ${{ env.BASE_BRANCH }}
```