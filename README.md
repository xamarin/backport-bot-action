# Xamarin Backport Bot

Note: this is used for internal use only.

Contains an action that's responsible for running an Azure DevOps build which backports a PR based on one branch to another branch.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft
trademarks or logos is subject to and must follow
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.


## Testing

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
