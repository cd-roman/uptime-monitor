# Website Uptime Monitor Action

A GitHub Action that monitors websites' uptime, logs results, and creates issues when sites are down. All managed entirely within your GitHub repository, with no external servers or cloud services required.

## Features

- ✅ Monitor multiple websites concurrently
- 📊 Log response times and status codes
- 🚨 Automatically create GitHub issues for failures
- 🌍 Timezone-aware timestamps
- ⚙️ Configurable timeouts and success codes
- 📝 Commit logs back to repository

## Getting Started

If you’re new to GitHub Actions or prefer a detailed, step-by-step guide, check out **[GETTING_STARTED](docs/GETTING_STARTED.md)** guide. Otherwise, continue here for a quick setup.

## Usage

```yaml
name: Website Uptime Monitor
on:
  schedule:
    - cron: '0 */12 * * 1-5'  # run automatically every 12 hours, Monday–Friday
  workflow_dispatch:

permissions:
  contents: write
  issues: write

jobs:
  uptime-check:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Monitor websites
        uses: cd-roman/uptime-monitor@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          sites: |
            https://example.com
            https://api.example.com/health
            https://status.example.com
          timezone: "Europe/Paris"
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `sites` | Newline-separated URLs to monitor | Yes | - |
| `timezone` | IANA timezone for timestamps | No | `UTC` |
| `log-file` | Path to log file | No | `uptime-monitor-results.log` |
| `timeout` | Request timeout in seconds | No | `15` |
| `success-codes` | HTTP codes considered successful | No | `200,201,202,204` |
| `max-concurrent` | Max concurrent requests | No | `5` |
| `create-issue` | Create GitHub issue on failures | No | `true` |

## Outputs

| Output | Description |
|--------|-------------|
| `failed-sites` | Number of sites that failed |
| `total-sites` | Total number of sites checked |
| `log-file-path` | Path to the generated log file |

## Permissions

```yaml
permissions:
  contents: write  # To commit log files
  issues: write    # To create issues on failures
```

## Advanced configuration

```yaml
name: Website Uptime Monitor
on:
  schedule:
    - cron: '0 */12 * * 1-5'  # run automatically every 12 hours, Monday–Friday
  workflow_dispatch:

permissions:
  contents: write    # for committing logs
  issues: write      # for opening issues

jobs:
  uptime-check:
    runs-on: ubuntu-latest
    timeout-minutes: 20    # cancel the job if it runs over 20 minutes

    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Monitor websites
        uses: cd-roman/uptime-monitor@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          # 1) List of sites, newline-separated
          sites: |
            https://example.com
            https://api.example.com/health
            https://status.example.com

          # 2) IANA timezone for log timestamps
          timezone: "America/New_York"

          # 3) Path to write the log file (relative to repo root)
          log-file: "logs/uptime-monitor.log"

          # 4) Commit message when pushing the updated log
          commit-message: "📊 Update uptime log"

          # 5) Per-request timeout (seconds)
          timeout: "15"

          # 6) Which HTTP codes count as “success”
          success-codes: "200,201,202,204,301,302"

          # 7) Max number of concurrent requests
          max-concurrent: "5"

          # 8) Whether to auto-create an issue on non-success code
          create-issue: "true"
```

## Troubleshooting & FAQs


### Q: Why isn’t my log file committing?

Ensure your workflow declares:

```yaml
permissions:
  contents: write
env:
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
Verify log-file is a path within the repository and not excluded by .gitignore.

---

### Q: My job hung or ran past the expected time.

Add a timeout-minutes: under your job to cap total runtime:

```yaml
jobs:
  uptime-check:
    timeout-minutes: 20
```
Try lowering the `timeout` value (in seconds) or reducing `max-concurrent` to avoid long-running or overlapping fetch requests.

---

### Q: How do I skip issue creation but still log failures?

Set `create-issue: "false"` in your step’s `with:` block. The log will still record failures, but no issue will be opened.

---

### Q: I get network or DNS errors for some URLs.

The action treats a failed fetch() as status 0 and logs “Error: Timeout” or the exception message.

You can adjust timeout (e.g. 5 seconds) or disable issue creation for noisy endpoints.

---

### Q: My configured success-codes aren’t recognized.

Provide a comma-separated list without spaces, e.g. "200,204,301".

The action parses each code with parseInt; invalid entries are ignored.

---

### Q: I’m not receiving email notifications when the action creates an issue.

The action opens issues using the GITHUB_TOKEN, so they’re authored by the GitHub Actions bot and you won’t be notified unless you’re subscribed to issue events or participate in the thread.

To receive emails for new issues, go to your repository, click Watch > Custom, select Issues, and then Apply. Once subscribed, you’ll get an email each time the action generates an issue.

---

### Q: I see a 6-hour job timeout error.

By default, GitHub cancels jobs after 6 hours. Use timeout-minutes: X in your workflow to shorten this and avoid runaway bills.

---

## 🙌 Contributing

Contributions are welcome! Refer to [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on getting started.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

## 💬 Support & feedback

If you have any questions, found a bug, want to request a feature, or need help with setup, please open an issue.

---

✨ **Happy monitoring!**  
