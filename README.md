# SnykCon2021
Using Snyk Effectively  on Github

## Github Action Scripts Used

This describes the use cases and the scripts used in the SnykCon talk. All of these workflow use [Snyk Actions] (https://github.com/snyk/actions) to execute the desired use cases.


# Open Source Delta Check
This Github Action lets you block pipelines only if new vulnerabilities are introduced. It uses the [Snyk Delta] (https://github.com/kriti-d/snyk-delta-check) tool to do the comparison with an already existing monitored projects to show results.

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