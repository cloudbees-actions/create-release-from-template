# Create a Release from a template on a remote CloudBees CD/RO instance
## Description
This is one of serveral GitHub Actions provided on an "as-is" basis by CloudBees that enable users to write GitHub Action Workflows that send work to an external CloudBees CD/RO instance. This Action enables workflows to create a Release from an existing template on a remote CloudBees CD/RO instance.
## Intended audience
For teams utilizing GitHub Actions for build and continuous integration, the CloudBees CD/RO Actions provide a mechanism for releasing software in a secure, governed and auditable manner with deep visibility, as is required in regulated industries, for example. Platform or shared services teams can build reusable content in the CloudBees CD/RO platform that conforms to company standards and removes the burden of release automation from the application teams.
## Prerequisites
CloudBees CD/RO is an enterprise "on-premise" product that automates software delivery processes, including production deployments and releases. To use utilize this GitHub Action, it is necessary to have access to a CloudBees CD/RO instance, in particular, 
- A CloudBees CD/RO instance that GitHub Actions can access through REST calls (TCP port 443)
- A valid API token for the CloudBees CD/RO instance. A token can be generated from the _Access Token_ link on the user profile page of the CloudBees CD/RO user interface; see [Manage access tokens via the UI](https://docs.cloudbees.com/docs/cloudbees-cd/latest/intro/sign-in-cd#_manage_access_tokens_via_the_ui) documentation for details.
These values should be stored as GitHub Action secrets to be referenced securely in a GitHub Actions workflow.
## Usage
The CloudBees CD/RO GitHub Actions are called from _steps_ in a GitHub Actions _workflow_. The following workflow extract illustrates how to create a Release from an existing template on a remote CloudBees CD/RO instance.
```yaml
steps:
  - name: Create release
    env:
      CDRO_URL: ${{ secrets.CDRO_URL }}
      CDRO_TOKEN: ${{ secrets.CDRO_TOKEN }}
    uses: cloudbees-github-actions/create-release-from-template@v1
    with:
      templateProjectName: Electric Cloud
      templateCatalogName: Releases
      templateName: Multi-stage CD Release
      actualParameter: |
        releaseProjectName: GHA Test
        releaseName: GHA Release from template
        releaseDescription: desc
        releaseStartDate: 2023-07-31
        releaseEndDate: 2023-08-02
```
### Inputs
| Name                   | Description                                                            | Required |
|------------------------|------------------------------------------------------------------------|----------|
| templateProjectName    | Project name of the release template                                   | yes      |
| templateCatalogName    | Template catalog name                                                  | yes      |
| templateName           | Release template name                                                  | yes      |
| actualParameter        | Parameters as list of key-value pairs                                  | no       |
| ignore-unverified-cert | Ignore unverified SSL certificate                                      | no       |
### Outputs
| Name                   | Description                                                            |
|------------------------|------------------------------------------------------------------------|
| response               | The JSON data structure emited by the API call. This data can be parsed to retrieve individual values from the response, for example, `${{ fromJson(steps.start-release.outputs.response).flowRuntime.flowRuntimeId }}` where `start-release` is the name of a previous step and `.flowRuntime.flowRuntimeId` is the selector for the release pipeline runtime ID. |
### Secrets and Variables
The following GitHub secrets are needed to run the Action. These can be set in the _Secrets and variable_ section of the workflow repository _Settings_ tab.
| Name                   | Description                                                            | Required |
|------------------------|------------------------------------------------------------------------|----------|
| CDRO_URL               | CloudBees CD/RO server URL, e.g., `https://my-cdro.net` or `https://74.125.134.147` | yes |
| CDRO_TOKEN             | CloudBees CD/RO API Access token                                       | yes      |
## Examples
### Create release from an existing template
1. Set up secrets in the repository settings for Actions. In the GitHub repository, select the _Settings_ tab, _Secrets_, _Variables_, and _Actions_. Use the _New Repository_ button to create the CDRO_URL and CDRO_TOKEN secrets.
2. Create a new workflow file in the `.github/workflows` directory, for example, `create-and-start-release-from-template.yml`:
```yaml
name: Create and start release from template

on:
  workflow_dispatch:
env:
  CDRO_URL: ${{ secrets.CDRO_URL }}
  CDRO_TOKEN: ${{ secrets.CDRO_TOKEN }}
jobs:
  create-and-start-release-from-template:
    runs-on: ubuntu-latest

    steps:

      - name: Create release
        uses: cloudbees-github-actions/create-release-from-template@v1
        with:
          templateProjectName: Electric Cloud
          templateCatalogName: Releases
          templateName: Multi-stage CD Release
          actualParameter:
            projectName: GHA Test
            releaseName: GHA Release from template
            pipelineParameter: |
              releaseProjectName: Default
              releaseName: SSC Generated
              releaseDescription: desc
              releaseStartDate: 2023-06-15
              releaseEndDate: "2023-06-20"
     
      - name: Start release
        uses: cloudbees-github-actions/start-release@v1
        with:
          projectName: Default
          releaseName: SSC Generated
```
3. Go to the GitHub `Actions` tab and run the workflow `Create and start release from template`
## Sample build and release repository
The [CloudBees CD/RO GitHub Actions Demonstration Repository](https://github.com/cloudbees-demos/gh-actions-demo) illustrates how to implement a build and release workflow with the CloudBees CD/RO GitHub Actions.
## License
The scripts and documentation in this project are released under the MIT License.
## Documentation
For more details about the CloudBees CD/RO product, view the [online documentation](https://docs.cloudbees.com/docs/cloudbees-cd/latest/github-actions/).
