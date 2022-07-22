# Github Actions

## Understanding GitHub Actions

### Overview

* CI/CD platforms
* Allows you to automate your build, test, and deployment pipeline

### The components of Github Actions

* Workflows ⊃ Events ⊃ Jobs
* Can configure a workflow to be triggered
  * A pull request being opened
  * An issue being created
* Job has one or more steps(script or action)
  * Job will run inside VM, container

#### Workflows

* Workflows == Configurable automated process
* Written in yaml
* Manually or defined schedule
* Defined in the `.github/workflows`
  * Can have multiple workflows
    * e.g.) There are A, B, C workflows
      * A - Build and test
      * B - Deploy your application
      * C - Adds label
* Can reuse with reference

#### Events

* Activity that triggers a workflow run
* Activity examples
  * Pull request
  * Opens an issue
  * Pushes a commit to a repository
  * Schedule(by posting REST API, or manually)

#### Jobs

* Set of steps in a workflow that execute on the same runner
  * Each step = shell script
  * Can share data, step <--> step
  * Can configure job's dpendencies with other jobs

#### Actions

* Performs repeated task
* Actions can
  * Pull git repository
  * Set up toolchain
  * Set up authentication to your cloud provider

#### Runners

* Server(virtual machine)
* Each runner can run single job at a time
* Can host your own runners

### Create an example workflow

```yaml
name: learn-github-actions # Optional, appers in action tab
on: [push] # Trigger
jobs: # Groups together all the jobs
  check-bats-version: # Job name
    runs-on: ubuntu-latest # Runner OS
    steps: # Groups all the steps
      - uses: actions/checkout@v3 # Actions repository (stored in public)
      - uses: actions/setup-node@v3
        with:
          node-version: '14'
      - run: npm install -g bats # Execute command
      - run: bats -v
```

## Finding and customizing actions

### Overview

**Actions** are defined in

* The same repository as your workflow file
* Any public repository
* A published Docker container image on Docker Hub

You can keep your actions up to date with Dependabot.

### Adding an action to your workflow

#### Adding an action from the same repository

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      # This step checks out a copy of your repository.
      - uses: actions/checkout@v3
      # This step references the directory that contains the action.
      - uses: ./.github/actions/hello-world-action
```

#### Adding an action from a different repository

```yaml
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: actions/setup-node@v3
```

#### Referencing a container on Docker Hub

```yaml
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: docker://alpine:3.8
```

### Using release management for your custom actions

[link](https://docs.github.com/en/actions/learn-github-actions/finding-and-customizing-actions#using-release-management-for-your-custom-actions)

### Using inputs and outputs with an action

```yaml
name: "Example"
description: "Receives file and generates output"
inputs:
  file-path: # id of input
    description: "Path to test script"
    required: true
    default: "test-file.js"
outputs:
  results-file: # id of output
    description: "Path to results file"
```

## Essential features of GitHub Actions

### Overview

Customization techniques

* Using variables
* Running scripts
* Sharing data and artifacts between jobs

### Using variables in your workflows

```yaml
jobs:
  example-job:
      steps:
        - name: Connect to PostgreSQL
          run: node client.js
          env:
            POSTGRES_HOST: postgres
            POSTGRES_PORT: 5432
```

### Adding scripts to your workflow

Exmaple 1

```yaml
jobs:
  example-job:
    steps:
      - run: npm install -g bats # Run command on the runner
```

Exmaple 2

```yaml
jobs:
  example-job:
    steps:
      - name: Run build script
        run: ./.github/scripts/build.sh # In your repository script
        shell: bash
```

### Sharing data between jobs

Can share job generated files as artifacts in Github.
All actions and workflows called within a run have write access to that run's artifacts.

#### Artifacts

* Created when you build and test your code.
* Binary or package files
* test results
* screenshots,
* log files

Create a file, upload it as an artifact

```yaml
jobs:
  example-job:
    name: Save output
    steps:
      - shell: bash
        run: |
          expr 1 + 1 > output.log
      - name: Upload output file
        uses: actions/upload-artifact@v3
        with:
          name: output-log-file
          path: output.log
```

Download artifact

```yaml
jobs:
  example-job:
    steps:
      - name: Download a single artifact
        uses: actions/download-artifact@v3
        with:
          name: output-log-file
```
