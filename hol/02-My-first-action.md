# 🔨 Hands-on: My first Action

In this hands-on lab you will learn how to create a composite action, pass in parameters and return values to your workflow. And you will learn how to test the action locally with a CI build.

This hands on lab consists of the following steps:
- [Creating the action](#creating-the-action)
- [Testing the action](#testing-the-action)
- Optional: [Release and use the action](#optional-release-and-use-the-action)


## Creating the action

1. Create a new file [`hello-world-composite-action/action.yml`](/../../new/main?filename=hello-world-composite-action%2Faction.yml)

2. Add content to the `action.yml` file follow the
  [instructions](https://docs.github.com/en/actions/tutorials/create-actions/create-a-composite-action)). Have the action run a `Composite` and pass
  in the parameter `who-to-greet` with the default value `world`. Also give back an output that returns the current time.


<details>
  <summary>Solution</summary>

```YAML
name: 'Hello World'
description: 'Greet someone'
inputs:
  who-to-greet:  # id of input
    description: 'Who to greet'
    required: true
    default: 'World'
outputs:
  random-number:
    description: "Random number"
    value: ${{ steps.random-number-generator.outputs.random-number }}
runs:
  using: "composite"
    using: "composite"
  steps:
    - name: Set Greeting
      run: echo "Hello $INPUT_WHO_TO_GREET."
      shell: bash
      env:
        INPUT_WHO_TO_GREET: ${{ inputs.who-to-greet }}

    - name: Random Number Generator
      id: random-number-generator
      run: echo "random-number=$(echo $RANDOM)" >> $GITHUB_OUTPUT
      shell: bash

    - name: Set GitHub Path
      run: echo "$GITHUB_ACTION_PATH" >> $GITHUB_PATH
      shell: bash
      env:
        GITHUB_ACTION_PATH: ${{ github.action_path }}

    - name: Run goodbye.sh
      run: goodbye.sh
      shell: bash
```

</details>

3. Commit the file (`[skip ci]` to not run a build, yet).

4. Create the file [`googbye.sh`](/../../new/main?filename=hello-world-composite-action%goodbye.sh) in the folder. It is a simple bash script that writes a message to the console.

<details>
  <summary>Solution</summary>

```bash
#!/bin/sh -l

echo "echo Goodbye"
```

</details>

5. Make sure you set the executable bit for the script when you commit it.

<details>
  <summary>Solution</summary>

```bash
git add --chmod=+x goodbye.sh
```

</details>

6. Commit the file (`[skip ci]` to not run a build, yet).

## Testing the action

1. To test the action we create a new workflow file [`.github/workflows/hello-world-composite-ci.yml`](/../../new/main?filename=.github%2Fworkflows%2Fhello-world-composite-ci.yml&workflow_template=blank).

2. The workflow should run on every push if the action has changed. Also add a manual trigger to start the build manually.
   Checkout the repository to reference the action locally and without a git reference.

<details>
  <summary>Solution</summary>

```YAML
name: CI Build for Composite Action
on:
  push:
    branches: [ main ]
    paths: [ hello-world-composite-action/** ]
  workflow_dispatch:

jobs:
  test-action:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v6.0.2

      - name: Run my own composite action
        id: hello-action
        uses: ./hello-world-composite-action
        with:
          who-to-greet: '@jessehouwing'

      - name: Output set in the composite action
        run: echo "The random number was ${{ steps.hello-action.outputs.random-number }} when the action said hello"

```

</details>

3. Run the workflow and see how the parameters are passed into the action and back to the workflow.

<img width="600" alt="image" src="https://user-images.githubusercontent.com/5276337/174239255-262a8014-4b66-40df-aa17-6f043f948342.png">

## Optional: Release and use the action

If time permits you can create a release and then use the action in a workflow in another repository.

> **Note**
> You can only publish a GitHub Action that exists in the root of a repository.

1. Create a new public repository `hello-world-composite-action` and copy all the files from [hello-world-composite-action](../hello-world-composite-action) to it.

2. Create a [new release](/../..releases/new) with a tag `v1` and the title `v1`. Click `Generate release notes` and publish the release.

<img width="600" alt="image" src="https://user-images.githubusercontent.com/5276337/174241482-6d3d0c34-9d55-4e3d-86fa-8ac28055cea8.png">

3. Go to Settings > Actions > General > Access, and ensure `Accessible from repositories owned by the user '<your-github-username>` is selected.

4. Create a new repository or use another existing one and create a simple workflow that calls the action your created in version `v1`.

<details>
  <summary>Solution</summary>

```YAML
name: Test
on: [workflow_dispatch]

jobs:
  test-action:
    runs-on: ubuntu-latest
    steps:
      - name: Say hello
        uses: <your-github-username>/hello-world-composit-action@v1
        with:
          who-to-greet: '@octocat'
```

</details>

## Summary

In this hands-on lab you've learned how to create a composite action, pass in parameters, return values to your workflow, and to test the action locally with a CI build.

You can continue now with [Staged deployments](03-Staged-deployments.md).
