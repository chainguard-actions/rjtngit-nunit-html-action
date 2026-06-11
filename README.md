# NUnit HTML Report

This Github Action generates a human-readable HTML report from NUnit XML test results.

![](example.png)

## Usage

```yaml
- name: Generate HTML test report
  uses: rempelj/nunit-html-action@v1.0.1
  if: always()
  with:
    inputXmlPath: artifacts/results.xml
    outputHtmlPath: artifacts/results.html
```

## License

The scripts and documentation in this project are released under the [MIT License](https://github.com/rempelj/nunit-html-action/blob/main/LICENSE).

## Privacy

This Action contacts Chainguard's licensing server to verify authorization. Connection metadata (IP address, GitHub repository identifier, timestamp, and any metadata encoded in the auth token) is transmitted to Chainguard, Inc. even if authorization is denied in accordance with our [Privacy Notice](https://www.chainguard.dev/legal/privacy-notice)
