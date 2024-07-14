# workflow-github

To automate adding issues to a GitHub project using GitHub Actions workflows, you can create a workflow file in your repository that triggers on issue creation and adds the new issue to a specified project. Here's a step-by-step guide:

### 1. Create a GitHub Personal Access Token

First, you need a GitHub Personal Access Token (PAT) with the necessary permissions to access the repository and projects.

1. Go to [GitHub Settings](https://github.com/settings/tokens).
2. Click on "Generate new token".
3. Give the token a descriptive name, and select the following scopes:
    - `repo` (full control of private repositories)
    - `workflow` (update GitHub Action workflows)
    - `project` (manage project permissions)
4. Generate the token and save it securely. You will need it for the workflow.

### 2. Store the Token as a Secret

1. Go to your repository on GitHub.
2. Click on `Settings` > `Secrets and variables` > `Actions` > `New repository secret`.
3. Add a new secret named `GH_TOKEN` and paste the Personal Access Token you generated earlier.

### 3. Create the Workflow File

Create a workflow file in your repository to automate the process. 

1. In your repository, navigate to `.github/workflows`.
2. Create a new file named `add-issue-to-project.yml`.

### 4. Define the Workflow

Here’s a sample workflow that adds new issues to a specified project:

```yaml
name: Add Issue to Project

on:
  issues:
    types: [opened]

jobs:
  add_issue_to_project:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v2

      - name: Add Issue to Project
        env:
          GH_TOKEN: ${{ secrets.GH_TOKEN }}
        run: |
          ISSUE_NUMBER=${{ github.event.issue.number }}
          PROJECT_ID=<YOUR_PROJECT_ID>
          COLUMN_ID=<YOUR_COLUMN_ID>
          
          curl -X POST -H "Authorization: token $GH_TOKEN" -H "Accept: application/vnd.github.inertia-preview+json" \
            https://api.github.com/projects/columns/$COLUMN_ID/cards \
            -d '{"content_id":'$ISSUE_NUMBER', "content_type":"Issue"}'
```

### 5. Retrieve Project and Column IDs

To fill in the placeholders `<YOUR_PROJECT_ID>` and `<YOUR_COLUMN_ID>`, you need to get the IDs of the project and the column where you want to add the issues.

1. **Get Project ID**:
    - Go to your project page.
    - Note the project number from the URL, e.g., `https://github.com/your-org/your-repo/projects/1`. The project number is `1`.
    - Use the GitHub API to get the project ID: `https://api.github.com/repos/your-org/your-repo/projects`.
    - The response will contain the ID of the project.

2. **Get Column ID**:
    - Use the GitHub API to get the columns of the project: `https://api.github.com/projects/<PROJECT_ID>/columns`.
    - The response will contain the IDs of the columns within the project.

### Example API Requests

```sh
# Get Project ID
curl -H "Authorization: token YOUR_PERSONAL_ACCESS_TOKEN" \
     -H "Accept: application/vnd.github.inertia-preview+json" \
     https://api.github.com/repos/your-org/your-repo/projects

# Get Column ID
curl -H "Authorization: token YOUR_PERSONAL_ACCESS_TOKEN" \
     -H "Accept: application/vnd.github.inertia-preview+json" \
     https://api.github.com/projects/YOUR_PROJECT_ID/columns
```

### 6. Update Workflow with IDs

Replace `<YOUR_PROJECT_ID>` and `<YOUR_COLUMN_ID>` in the workflow file with the actual IDs obtained from the API responses.

### 7. Commit and Push the Workflow

1. Commit and push the workflow file to your repository:

```sh
git add .github/workflows/add-issue-to-project.yml
git commit -m "Add workflow to add issues to project"
git push origin main
```

Now, every time a new issue is created in the repository, the GitHub Actions workflow will automatically add the issue to the specified project column.
