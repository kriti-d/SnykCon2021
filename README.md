# SnykCon2021
Using Snyk Effectively  on Github

## Github Action Scripts Used

This describes the use cases and the scripts used in the SnykCon talk. All of these workflow use [Snyk Actions](https://github.com/snyk/actions) to execute the desired use cases.


### Open Source Delta Check
This workflow lets you block pipelines only if new vulnerabilities are introduced. It uses the [Snyk Delta](https://github.com/kriti-d/snyk-delta-check) tool to do the comparison with an already existing monitored projects to show results.

```bash
jobs:
  security:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [12.x]
    steps:
      - uses: actions/checkout@master
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}
      - name: Installing snyk-delta and dependencies
        run: npm i -g snyk-delta
      - uses: snyk/actions/setup@master
      - name: Snyk Test
        run: snyk test --json --print-deps | snyk-delta
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

### Code Scanning Alerts for Snyk Code (SAST)
This workflow tests your application for SAST vulnerabities and then presents them in the Secuirty tab of Github. It provides in-line details of where the vulnerability is found and provides details and guidance to fix.

```bash
jobs:
  snyk:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: snyk/actions/setup@master
      - name: Snyk Code Test
        continue-on-error: true
        run: snyk code test --sarif > snyk_sarif
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN}}
      - name: Upload results to Github Code Scanning
        uses: github/codeql-action/upload-sarif@v1
        with:
          sarif_file: snyk_sarif
```


### Container Monitor Results
This Github Action lets you inspect your image for vulnerabilities, and creates a project on your Snyk Account with the available base image remediation recommendations.

```bash
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Build the Docker image
      run: docker build . --file Dockerfile --tag my-vuln-image:latest
    - name: Monitor Docker Vulnerabilities
      uses: snyk/actions/docker@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      with:
        image: my-vuln-image:latest
        command: monitor
```