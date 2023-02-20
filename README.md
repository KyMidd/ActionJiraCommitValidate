# Jira Commit Validate

This Action is capable of reading all commits a branch has that deviate from a base branch, and will derive on each commit the leading ticket number. That ticket number will be checked against Jira using an API call to validate that the ticket exists. 

PRs can be blocked from merging if this API call derives that a Jira ticket doesn't exist for each commit. 

If there is a failure, users should rebase their branch's commit history to contain only valid commits that link real Jira tickets. 

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