# Getting Started: Website Uptime Monitor GitHub Action

Welcome! This guide will help you set up the Website Uptime Monitor Action in your GitHub repository, even if you have little or no experience with GitHub Actions.

---

## What is this Action?

The Website Uptime Monitor is a GitHub Action that automatically checks if your websites are up and running. It logs the results and can create GitHub issues if a site is down.

Below is a step-by-step instruction on setting up your workflow and start monitoring your websites.

---

## Step 1: Prepare Your Repository

You can use this action in **any GitHub repository** (new or existing).

### For a New Repository

1. Go to [GitHub](https://github.com/) and create a new repository.
2. Clone the repository to your computer (optional).

### For an Existing Repository

Just continue with your chosen repository.

---

## Step 2: Create the Workflow File

1. In your repository, create a folder called `.github/workflows` if it doesn't exist.

2. Inside `.github/workflows`, create a new file named `uptime-monitor.yml`.

---

## Step 3: Add the Workflow Configuration

Copy and paste the following example into your `uptime-monitor.yml` file. You can tweak the schedule, inputs, and any other configuration options later to suit your needs:

```yaml
name: Website Uptime Monitor
on:
  schedule:
    - cron: '0 8 * * *'  # Runs every day at 8:00 UTC. Adjust schedule or remove entirely if you don't need it
  workflow_dispatch:

permissions:
  contents: write
  issues: write

jobs:
  uptime-check:
    runs-on: ubuntu-latest
    timeout-minutes: 20    # cancel the job if it runs over 20 minutes

    steps:
      - uses: actions/checkout@v6

      - name: Monitor websites
        uses: cd-roman/uptime-monitor@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          sites: |
            https://example.com
            https://api.example.com/example
          timezone: "UTC"
```

**Tip:**  
- The `schedule` section makes the check run automatically every day. You can change the schedule or remove if entirely if you only want to run the action manually.
- Generate your custom schedule by using, for example, this [cron expression generator](https://crontab.cronhub.io/)
- The `workflow_dispatch` lets you run it manually from the GitHub Actions tab in your repository.

> ⚠️**IMPORTANT NOTE:** 
Please be aware that running this Action may incur charges from GitHub. Check your usage under **Settings > Billing & Licensing > Usage**.  
>
> As of June 2025, usage in public repositories is free. Private repositories include a limited number of free minutes monthly. Once you exceed your free minutes amount, each workflow run will be billed by GitHub.  
>
> **It is your responsibility to check GitHub’s billing changes.**

---

## Step 4: Customize the Inputs

You can adjust the action’s settings using the `with:` block. Here’s what each input means:

| Input           | Description                                                                 | Required | Default                        |
|-----------------|-----------------------------------------------------------------------------|----------|---------------------------------|
| `sites`         | Newline-separated list of URLs to monitor                                   | Yes      | -                               |
| `timezone`      | IANA timezone for timestamps (e.g., "America/New_York")                     | No       | `UTC`                           |
| `log-file`      | Path to log file (relative to repo root)                                    | No       | `uptime-monitor-results.log`     |
| `commit-message`| Commit message for log file updates                                         | No       | `🔍 Uptime monitoring results`   |
| `timeout`       | Request timeout in seconds                                                  | No       | `15`                            |
| `success-codes` | Comma-separated HTTP status codes considered successful                     | No       | `200,201,202,204`               |
| `max-concurrent`| Maximum number of concurrent requests                                       | No       | `5`                             |
| `create-issue`  | Create GitHub issue on failures (`true` or `false`)                         | No       | `true`                          |

**Example with more options:**

```yaml
      - name: Monitor websites
        uses: cd-roman/uptime-monitor-action@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          sites: |
            https://example.com
            https://api.example.com/health
          timezone: "America/New_York"
          log-file: "logs/uptime-monitor.log"
          commit-message: "📊 Update uptime log"
          timeout: "15"
          success-codes: "200,201,204"
          max-concurrent: "5"
          create-issue: "false"
```

> ⚠️**Additional note about `timeout-minutes`:** 
> Don’t confuse it with the `timeout` input, which specifies the request timeout in seconds!
>
> The `timeout-minutes` setting controls how long GitHub will allow the workflow job to run before automatically cancelling it. The maximum value you can set is **360 minutes (6 hours)**. Increasing this limit means your workflow could run (and consume billed minutes) for a much longer time, which may result in higher usage costs—especially in private repositories. Unless you have a specific need (such as monitoring a very large number of sites or handling slow endpoints), it’s best to keep this value low. **Do not set a high timeout unless you are sure it’s necessary and you understand the potential billing impact.**

---

## Step 5: Make sure your log file isn’t ignored

Your `.gitignore` may include a rule like `*.log`, which will block the action workflow from creating commits with the updated log file. Here’s how to handle it in different scenarios:

**Scenario 1. No `*.log` entry**
   If you don’t have `*.log` in your `.gitignore`, you’re all set. Continue to the next step.

**Scenario 2. You’re ignoring all `.log` files but don’t need that rule**  
   In this case, remove the `*.log` line from your `.gitignore` (only if it won’t affect other parts of your project).

**Scenario 3. You want to ignore other logs but keep the uptime log**  
   Keep `*.log` in your `.gitignore`, but add an exception for the log file, created by the uptime monitor:
   ```gitignore
   *.log
   !uptime-monitor-results.log
   ```
  This allows the action to commit its results file while still ignoring all the other `.log` files.

  Please note that the default log file is `uptime-monitor-results.log`. If you change it with a custom path in your configuration file, you need to add it to the `.gitignore` file in Scenario 3 as well.

---

## Step 6: Commit and Push

1. Save your changes.
2. Commit the new workflow file (and changes in .gitignore):
   ```sh
   git add .
   git commit -m "Add Website Uptime Monitor workflow"
   git push
   ```
3. Go to the **Actions** tab in your GitHub repository, select your workflow from the list, and run it manually to ensure it works correctly.

---

## Step 7: Check the Results

- The action will log results to the file you specified (default: `uptime-monitor-results.log` in the root of your project).
- If a site is down, and `create-issue` is `true`, a GitHub issue will be created automatically.
- **Note:** Each workflow run creates a new commit for the log file. During active development, pull frequently by using `git pull` to stay in sync, or consider running this action in a separate repository to keep extra commits out of your main project.

---

## Outputs

After the action runs, it provides these outputs:

| Output           | Description                                 |
|------------------|---------------------------------------------|
| `failed-sites`   | Number of sites that failed                 |
| `total-sites`    | Total number of sites checked               |
| `log-file-path`  | Path to the generated log file              |

You can use these outputs in later steps of your workflow if needed.

To use the outputs from your action in the later steps, you need to add an id to the step running your action:

```yaml
      - name: Monitor websites
      # add this line
      id: monitor 
```

To see the outputs from your action in the workflow logs, you can add another step:

```yaml
      - name: Show action outputs
        run: |
          echo "Failed sites: ${{ steps.monitor.outputs.failed-sites }}"
          echo "Total sites: ${{ steps.monitor.outputs.total-sites }}"
          echo "Log file path: ${{ steps.monitor.outputs.log-file-path }}"
```

---

## Troubleshooting

- **Log file not committed?**  
  Make sure your workflow has `permissions: contents: write` and uses the correct `GITHUB_TOKEN`.

- **No issues created?**  
  Ensure `permissions: issues: write` is set and `create-issue` is `"true"`.

- **Timeouts or errors?**  
  Try lowering the `timeout` value or reducing `max-concurrent`.

---

## Need More Help?

- See the main [README.md](../README.md) for advanced usage and FAQs.
- Open an issue in the repository if you get stuck.

---

✨ **You’re all set! Your websites will now be monitored automatically.**
