# action-release

A GitHub Action to release tags.
This is a composite action that combines other actions, like:

- metcalfc/changelog-generator
- softprops/action-gh-release
- peter-evans/find-comment
- chuhlomin/render-template
- peter-evans/create-or-update-comment

This action is mean to execute when new git tags are pushed (see how to create tags
with [action-git-tag](https://github.com/GRESB/action-git-tag)).
When invoked this action will create a change log, and a release (including the change log and user supplied input).
If a PR number has been supplied it will look for a comment in that PR that contains a tag comment header.
If the PR comment is found it will be updated, otherwise a new one is created with information about the release.

## Inputs

| Input              | Description                                                                                 | Required | Default                                                                               |
|--------------------|---------------------------------------------------------------------------------------------|----------|---------------------------------------------------------------------------------------|
| name               | The name of the release.                                                                    | true     |                                                                                       |
| is-final           | Whether the release is a final release (eg v1.2.3).                                         | true     |                                                                                       |
| pr-number          | The PR number of the tag is meant for a release candidate.                                  | false    |                                                                                       |
| tag-comment-header | The header on the PR tag comment, used to update the comment and to find existing comments. | false    | '## Tag created'                                                                      |
| body               | The text to post on the release.                                                            | true     |                                                                                       |
| workflow-run-url   | The url of the workflow run.                                                                | false    | '${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}' |
| github-token       | The GitHub token used for creating the tag.                                                 | true     |                                                                                       |

## Usage

The default `secrets.GITHUB_TOKEN` is able to create the release and comment on the pull request.
The generation of the release changelog requires a full checkout of the repository.

To create pull request comments, the action needs a template under `.github/templates/tag-comment.md`, with the
following variables.

```md
{{ .header }}

Tag: `{{ .tag }}`

{{ .body }}

{{ .footer }}
```

The template can contain any other static text.

### Create a final release 

```yaml
name: Release Final Tag


on:
  push:
    tags:
      - 'v[0-9]+.[0-9]+.[0-9]+'


jobs:
  tag:
    name: Release git tag
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - name: Read Pushed Tag
        id: tag
        uses: GRESB/action-git-tag@main
        with:
          read: true
          ref: ${{ github.ref }}
      - name: Release
        uses: ./
        with:
          name: ${{ steps.tag.outputs.tag }}
          is-final: ${{ steps.tag.outputs.is-final }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          body: |
            ### Artifacts

            - Container image:

              ```some-container-image:${{ steps.tag.outputs.tag }}```
```

### Create a release candidate for a pull request

```yaml
name: Release Candidate Tag - Pull Request


on:
  push:
    tags:
      - 'v[0-9]+.[0-9]+.[0-9]+-pr1234-rc.2'


jobs:
  tag:
    name: Release git tag
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - name: Read Pushed Tag
        id: tag
        uses: GRESB/action-git-tag@main
        with:
          read: true
          ref: ${{ github.ref }}
      - name: Release
        uses: ./
        with:
          name: ${{ steps.tag.outputs.tag }}
          is-final: ${{ steps.tag.outputs.is-final }}
          pr-number: ${{ steps.tag.outputs.pr-number }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          body: |
            ### Artifacts

            - Container image:

              ```some-container-image:${{ steps.tag.outputs.tag }}```
```

### Create a release 
