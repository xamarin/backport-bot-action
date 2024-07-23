# Xamarin Backport Bot

Note: this is used for internal use only.

Contains an action that's responsible for running an Azure DevOps build which backports a PR based on one branch to another branch.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft
trademarks or logos is subject to and must follow
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.

## Onboarding

A federated credential needs to be added for each repo where the Backport bot runs.  Peform the following steps to add the federated credential representing a repo to the [VSEng-AzureDevOps-Xamarin-BackportBot-Identity](https://ms.portal.azure.com/#@microsoft.onmicrosoft.com/resource/subscriptions/7b4817ae-218f-464a-bab1-a9df2d99e1e5/resourceGroups/AzureDevOps/providers/Microsoft.ManagedIdentity/userAssignedIdentities/VSEng-AzureDevOps-Xamarin-BackportBot-Identity/overview) managed identity
- Navigate to the [federated credentials](https://ms.portal.azure.com/#@microsoft.onmicrosoft.com/resource/subscriptions/7b4817ae-218f-464a-bab1-a9df2d99e1e5/resourceGroups/AzureDevOps/providers/Microsoft.ManagedIdentity/userAssignedIdentities/VSEng-AzureDevOps-Xamarin-BackportBot-Identity/federatedcredentials) section for `VSEng-AzureDevOps-Xamarin-BackportBot-Identity`
- Click `+ Add Credential`
- Select `Github Actions deploying Azure resources` from the `Federated credential scenario` dropdown
- Fill out the rest of the form as follows. Leave any field not specified below set to its default value.
  - Organization: Name of the GitHub organziation such as `xamarin`
  - Repository: Name of a repo within the organization such as `xamarin-macios`
  - Entity: Select `Branch` from the dropdown
  - Branch: `main` (or the name of the default branch for the repo)
  - Name: Enter a name using the naming format of `[Oranization]--[repo-name]--main-branch` such as `xamarin--xamarin-macios--main-branch`

## Testing

To test `backport-bot.yml` pipeline changes, you will need to edit the `.github/workflows/backport-action.yml` file and temporarily update the trigger target branch from `main` to the name of your topic branch here:<br>
https://github.com/xamarin/backport-bot-action/blob/b6cb9b0c1b51a5a03ba79cea51ee8602f24afe26/.github/workflows/backport-action.yml#L147

For example change the `refName` setting from `refs/heads/main` to your topic branch such as `refs/heads/[TOPIC-BRANCH]`.  You should also add warning comment on the same line as a reminder to undo the change prior to merging your PR to main as follows:
```
refName = "refs/heads/[TOPIC-BRANCH]"  # UNDONE: DO NOT MERGE TO MAIN: Revert to using the main branch after testing
```

You can test changes from your topic branch (PR branch) by using the `v1.0-test` tag. Associate the tag with your latest changes by performing the following steps

```
git checkout [TOPIC_BRANCH_NAME]
git tag -f v1.0-test
git push --tags --force
```

To exercise your changes associated with the `v1.0-test` tag, use the following test PR
https://github.com/xamarin/backport-bot-action/pull/11

Apply the following comment to the PR

```
@gitbot backport backport-target
```

See the [test PR](https://github.com/xamarin/backport-bot-action/pull/11) for additional guidance & details

## Deploying

Release management of the backport bot action is done though git tags. At present, the current release can be found at `v1.1`.

To deploy your changes, please:

1. Update the tag locally with:
```bash
git tag -f $TAG_NAME
```

2. Push the tag to the remote (https://github.com/xamarin/backport-bot-action):
```bash
git push --tags
```
Please note that for updating the `v1.1` tag (or any other tag you want to push if it already exists), you will need to `push --force` to overwrite the existing tag.

In order to pick up the changes for Staging, make sure that the backport trigger YAML in your target repo (usually found at https://github.com/xamarin/$REPO_NAME/blob/main/.github/workflows/backport-trigger.yml) points to your desired tag.

For example, https://github.com/xamarin/.github/blob/main/.github/workflows/backport-trigger.yml#L13, the `uses` line should be updated as follows:
```yaml
      - uses: xamarin/backport-bot-action@$TAG_NAME
```
where `$TAG_NAME` is filled in with the actual name of the tag.

You can list tags by executing the following command

```
git tag
```

To view the contents of a tag execute the following command:

```
git show $TAG_NAME
```

## Resources

|Resource type|Resource|Resource group|
|:---|:---|:--|
|Backport Build|[Xamarin Backport Bot](https://devdiv.visualstudio.com/DevDiv/_build?definitionId=13834)|N/A|
|Telemetry|[PipelineTelemetry](https://ms.portal.azure.com#@72f988bf-86f1-41af-91ab-2d7cd011db47/blade/Microsoft_OperationsManagementSuite_Workspace/Logs.ReactView/resourceId/%2Fsubscriptions%2F26b9b438-7fe8-482f-b732-ea99c70f2abb%2FresourceGroups%2FPipelineTelemetry%2Fproviders%2Fmicrosoft.insights%2Fcomponents%2FPipelineTelemetry/source/LogsBlade.AnalyticsShareLinkToQuery/q/H4sIAAAAAAAAA6VT0UrDMBR931dc%252BrIWhqATUUYfHFPYi47NH0jbOxeXJjG5cU7EbzfN2tkx6gTz1sM5N%252BeekwokmGPGLE4LSOF8eHNxNeoJj45ZvtbKUI1fDy93OMtz1MQygfeMC2fQQtoDf3JnSZV3byjJBuATNis0CJKVCJaYIbvhtIJo7Lgo4InZNVQzsIhqPr4TygLCV3UCcYLL4IEUlxTvbpnwEqXlStqzhsMlJw9Mi2RwqD8p9pIDv%252B1b03YOzHuLSVkyXD4fD3vweyaQfkF0q7XYwoxRvopAGfiL5lGjhJkTwvfx6tBS1LjSRr1gTs02o95B0l0p93cpL5zvCwss%252BmGaN9NB3DVxijXzAGdCbFtzm96CNPtHZdlf62rR52idoCDpynj8w6uVllBXyf8qC9Xs4806HsV%252BezCNE86XEHfxwyNqFuUyPv6dkoEPu%252BYv1lxrn%252FGgvWxlyrqyZIZ%252FIOTK%252BagSyLa1g2%252FFm1yz0wMAAA%253D%253D/timespan/P7D)|[PipelineTelemetry](https://ms.portal.azure.com/#@microsoft.onmicrosoft.com/resource/subscriptions/26b9b438-7fe8-482f-b732-ea99c70f2abb/resourceGroups/pipelinetelemetry/overview)|
|Managed identity|[VSEng-AzureDevOps-Xamarin-BackportBot-Identity](https://ms.portal.azure.com/#@microsoft.onmicrosoft.com/resource/subscriptions/7b4817ae-218f-464a-bab1-a9df2d99e1e5/resourceGroups/AzureDevOps/providers/Microsoft.ManagedIdentity/userAssignedIdentities/VSEng-AzureDevOps-Xamarin-BackportBot-Identity/overview)|[AzureDevOps](https://ms.portal.azure.com/#@microsoft.onmicrosoft.com/resource/subscriptions/7b4817ae-218f-464a-bab1-a9df2d99e1e5/resourceGroups/AzureDevOps/overview)|
