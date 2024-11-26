
## Environment variables


### Defining variables

- At workflow level

```YAML
 env:
  NODE_ENV: production
  API_URL: https://api.example.com
```

- At job level

```YAML
 jobs:
  example-job:
    runs-on: ubuntu-latest
    env:
      JOB_SPECIFIC_VAR: value
    steps:
      - name: Print job variable
        run: echo "Job Variable: $JOB_SPECIFIC_VAR"
```

- At step level

```YAML
 steps:
  - name: Define a variable
    run: |
      echo "Step-specific variable: $MY_VAR"
    env:
      MY_VAR: "StepValue"
```

- Using secrets
Sensitive data should always be stored as GitHub Secrets for security purposes. Secrets can be referenced as environment variables.


```YAML
jobs:
  example-job:
    runs-on: ubuntu-latest
    steps:
      - name: Access secret
        env:
          API_KEY: ${{ secrets.API_KEY }}
        run: echo "Using API Key: $API_KEY"
```


### Accessing variables

1. Environment variables are accessed in scripts using platform-specific syntax:
 - Linux/macOS: Use $VARIABLE_NAME.
 - Windows: Use %VARIABLE_NAME%.

```YAML
# Example in case of a workflow step
- name: Access environment variable
  run: echo "Environment variable: $API_URL"
```

 2. From workflow expressions (Github actions)

```YAML
steps:
  - name: Use in expression
    run: echo "The environment is ${{ env.NODE_ENV }}"
```

## Cache dependencies

```YAML
...
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v4
      - name: Cache dependencies
        uses: actions/cache@v4
        with:
          path: ~/.npm # ficheiros ou pastas do que se quer guardar em cache~
          # identifica a cache
          # a key deve ter um valor que seja parcialmente dinamico, para que a key mude sempre que as dependencias mudarem
          key: deps-nome-modules-${{ hashFiles('**/package-lock.json')}} 

```

## Download and upload artifacts

```YAML
...
    steps:
     ...
     - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: dist-files  # artifacts name
          path: dist
```


```YAML
 ...
    steps:
      - name: Get build artifacts #download the previous artifacts
        uses: actions/download-artifact@v4 # this action will download the artifact and unzip here
        with: 
          name: dist-files 
          # path: dist # onde quero colocar os ficheiros do zip. Se nao indicar path vai ser na diretoria atual (root)
      - name: Output
        run: ls
      - name: Output js file name
        run: echo "${{ needs.build.outputs.script-file}}" #O needs context é usado para aceder aos outputs de todos os jobs
      - name: Deploy
        run: echo "Deploying"
```


## Controlling Execution Flow & Job execution
- on job or step, via *if* field:

```YAML
...

jobs:
  test:
    ...
    steps:
	    ...
      - name: Test code
	      id: run-test-step # to identify the step, so ot could be used on next steps
        run: npm run test
      - name: Upload test report
	      if: failure() && steps.run-test-step.outcome == 'failure" # this allow to run this step only if previous step fails
	      uses: actions/upload-artifact@v4
	      with:
		      name: test-report
		      path: test.json
```

