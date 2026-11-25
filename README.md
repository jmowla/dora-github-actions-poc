# Datadog DORA GitHub Actions PoC

Deploy a healthy release, a breaking release, and a hotfix to see how Datadog records delivery and recovery. Each run copies a health fixture to a temporary file on the GitHub-hosted runner.
The workflow reports the deployment before checking the deployed fixture. The breaking release therefore produces a deployment event and a failed smoke check.

## Set up

1. Put this directory's contents, including `.github/`, at the root of a GitHub.com repository with `main` as its default branch.
2. In Datadog DORA settings, enable **datadog-ci CLI / API** as a deployment source. Connect the test repository through the Datadog GitHub integration and verify that **Pull Request Information**
   meets its requirements. In DORA change failure detection, check that the hotfix PR label rule is enabled.
3. Add the `DD_API_KEY` GitHub Actions secret. Set the `DD_SITE` Actions variable if your site differs from `datadoghq.com`. The workflow defaults to service `dora-github-actions-poc` and
   environment `dora-poc`. Set Actions variables `DORA_SERVICE` and `DORA_ENV` if you need distinct values for your test.

[Datadog DORA setup](https://docs.datadoghq.com/delivery_performance/dora_metrics/setup/)

[Change failure detection rules](https://docs.datadoghq.com/delivery_performance/dora_metrics/change_failure_detection/)

## Run the three releases

In **Actions → Datadog DORA PoC → Run workflow**, select `main`. Leave **Send the deployment event to Datadog** unchecked for a dry run that prints the payload.
Check it for each of the three live runs.

| Release | Change on `main` | Expected workflow result |
| --- | --- | --- |
| Healthy | Start with the supplied `health.txt` containing `ok`. | Deploy and smoke check pass. |
| Breaking | Merge a PR that changes `health.txt` to `broken`. | Deploy and event submission pass. The smoke check fails. |
| Hotfix | Merge a new PR that restores `ok`. Add the `hotfix` label before merging. | Deploy and smoke check pass. |

Run each release before merging the next one. For a visible recovery interval, wait a minute after the breaking run before dispatching the hotfix. Start a new workflow run for each release.

## Check Datadog

Filter DORA Metrics by the PoC service and environment. In the deployment list, look for three events with different commit SHAs. The first event establishes the baseline for change lead time.
Check that subsequent events show their commits and PRs. Open **View Change Failures** and verify that the breaking deployment is flagged and linked to the hotfix as a rollforward.

With those three deployments in the filtered window, expect one change failure out of three. Compare the reported recovery time with the elapsed time between completion of the breaking deployment
and completion of the hotfix deployment. Datadog classifies the failure from the hotfix rule. The smoke check gives you a separate record of when the simulated service was unhealthy.

[Datadog metric calculations](https://docs.datadoghq.com/delivery_performance/dora_metrics/calculation/)
