# How to Automate Tencent CAPTCHA

Tencent CAPTCHA can block an automated scenario during registration, login, or form submission. This often happens in Selenium, Playwright, and other E2E tests.

If you control the application configuration, use Tencent CAPTCHA test mode. If the test works with a real CAPTCHA or you cannot change the configuration, get the result through Captcha Solver and pass it to the page callback.

This article explains how to automate Tencent CAPTCHA with the official [Captcha Solver Python SDK](https://github.com/captcha-solver-api/python-sdk). The SDK repository contains the installation instructions, API reference, and complete synchronous and asynchronous examples.

This repository was created as an example for the article: [How to Automate Tencent CAPTCHA](https://captcha-solver.com/en/blog/how-to-automate-tencent-captcha).

## Tencent CAPTCHA Data

You need the following data to create a task:

- `websiteURL` - the URL of the page with the CAPTCHA;
- `appId` - the Tencent CAPTCHA identifier;
- `clientKey` - the access key for the Captcha Solver API.

Tencent CAPTCHA uses `appId`, not `websiteKey`. Find the `appId` in the widget configuration on the page. For example:

```javascript
new TencentCaptcha("YOUR_APP_ID", onSolved);
```

Replace `YOUR_APP_ID` with the value from the target page configuration.

The Python SDK accepts the `clientKey` as the first argument of `CaptchaClient`. In the examples in the [SDK repository](https://github.com/captcha-solver-api/python-sdk), the key is stored in the `CAPTCHA_API_KEY` environment variable.

## Choosing the Task Type

Captcha Solver supports two Tencent task types:

- `TencentTaskProxyless` - solve without a client proxy;
- `TencentTask` - solve through a client proxy.

Use `TencentTaskProxyless` if the solve does not require a specific proxy. Use `TencentTask` if the solve request must be executed through your proxy.

## Installing the Python SDK

Install the SDK from its public GitHub repository:

```bash
pip install git+https://github.com/captcha-solver-api/python-sdk.git
```

For the current installation requirements and configuration options, see the [Python SDK README](https://github.com/captcha-solver-api/python-sdk#installation).

## Automating Tencent CAPTCHA

The SDK's `solve()` method creates a task, checks its status, and returns the result after the solve is complete. Use the ready-made [synchronous Tencent example](https://github.com/captcha-solver-api/python-sdk/blob/main/examples/sync/tencent.py) as the starting point for a regular test.

The proxyless task has this form:

```python
from captcha_sdk import CaptchaClient
from captcha_sdk.tasks import TencentTaskProxyless


client = CaptchaClient("YOUR_CLIENT_KEY")

solution = client.solve(
    TencentTaskProxyless(
        websiteURL="https://example.com/register",
        appId="YOUR_APP_ID",
    )
)
```

For a client proxy, use `TencentTask`. The complete example and all proxy fields are documented in the [SDK Tencent example](https://github.com/captcha-solver-api/python-sdk/blob/main/examples/sync/tencent.py).

The supported proxy types are `http`, `socks4`, and `socks5`. If the proxy does not require authentication, omit `proxyLogin` and `proxyPassword`.

### Local Python Examples

The repository includes the complete SDK examples for both client modes:

- [Synchronous example](examples/sync/tencent.py)
- [Asynchronous example](examples/async/tencent.py)

Set `CAPTCHA_API_KEY` and replace the placeholder `websiteURL` and `appId` values before running either example:

```bash
python examples/sync/tencent.py
python examples/async/tencent.py
```

Both examples show proxyless and proxy-based tasks. They print the complete Tencent result object, which must be passed unchanged to the page callback.

## Passing the Result to the Callback

Tencent CAPTCHA returns an object containing the verification result:

```javascript
const solution = {
  appid: "...",
  ret: 0,
  ticket: "...",
  randstr: "..."
};
```

Pass this object to the callback specified when initializing the CAPTCHA:

```javascript
onSolved(solution);
```

`onSolved` is an example. Use the actual callback from the page implementation. Captcha Solver returns the result but does not submit the form automatically. The E2E test must pass the object from the Python process to the browser and call the page callback.

## Integrating with an E2E Test

The automated workflow is:

1. Open the page containing the CAPTCHA.
2. Get the `appId` from the page configuration.
3. Create `TencentTaskProxyless` or `TencentTask`.
4. Call `client.solve()`.
5. Pass the returned object to the page callback.
6. Submit the form.
7. Continue with the test assertions.

Get the solution immediately before submitting the form and pass it to the same page where the CAPTCHA was created.

For asynchronous test infrastructure, use `AsyncCaptchaClient`. The complete implementation is available in the [asynchronous Tencent example](https://github.com/captcha-solver-api/python-sdk/blob/main/examples/async/tencent.py).

## Custom Tencent CAPTCHA Script URL

If the page loads the Tencent CAPTCHA script from a custom URL, pass it through the optional `captchaScript` parameter. See the [SDK Tencent example](https://github.com/captcha-solver-api/python-sdk/blob/main/examples/sync/tencent.py) for the current parameter format.

Use `captchaScript` only for pages that actually load Tencent CAPTCHA from a custom URL.

## Checking the Integration

Before running the test, check the following:

- `websiteURL` points to the page containing Tencent CAPTCHA;
- `appId` matches the widget configuration;
- `CAPTCHA_API_KEY` contains a valid `clientKey`;
- `captchaScript` is used only for a custom script URL;
- `TencentTask` contains `proxyType`, `proxyAddress`, and `proxyPort`;
- the callback receives the entire result object;
- the test passes the result to the same page where the CAPTCHA was created.

## Resources

- [Captcha Solver Python SDK](https://github.com/captcha-solver-api/python-sdk)
- [Synchronous Tencent example](https://github.com/captcha-solver-api/python-sdk/blob/main/examples/sync/tencent.py)
- [Asynchronous Tencent example](https://github.com/captcha-solver-api/python-sdk/blob/main/examples/async/tencent.py)
- [Tencent CAPTCHA documentation](https://captcha-solver.com/en/docs/methods#tencent)
- [Captcha Solver API repository](https://github.com/dzmitry-duboyski/captcha-solver)

## Summary

For an authorized E2E test, use Tencent's test mode when you control the application configuration. Otherwise, use `TencentTaskProxyless` without a client proxy or `TencentTask` with your proxy.

Pass `websiteURL` and `appId` to the task, call `client.solve()`, and pass the returned object to the page callback before submitting the form. For the full implementation, refer to the [Captcha Solver Python SDK](https://github.com/captcha-solver-api/python-sdk) and its [Tencent examples](https://github.com/captcha-solver-api/python-sdk/tree/main/examples).
