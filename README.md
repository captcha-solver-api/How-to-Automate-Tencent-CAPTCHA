# How to Automate Tencent CAPTCHA

Tencent CAPTCHA can block an automated scenario during registration, login, or form submission. This often happens in Selenium, Playwright, and other E2E tests.

If you control the application configuration, use Tencent CAPTCHA test mode. If the test works with a real CAPTCHA or you cannot change the configuration, get the result through Captcha Solver and pass it to the page callback.

This repository contains synchronous and asynchronous examples for the official Captcha Solver SDKs:

- [Python SDK](https://github.com/captcha-solver-api/python-sdk)
- [JavaScript SDK](https://github.com/captcha-solver-api/javascript-sdk)

It accompanies the article [How to Automate Tencent CAPTCHA](https://captcha-solver.com/en/blog/how-to-automate-tencent-captcha).

## Tencent CAPTCHA Data

You need the following data to create a task:

- `websiteURL` — the URL of the page with the CAPTCHA;
- `appId` — the Tencent CAPTCHA identifier;
- `clientKey` — the access key for the Captcha Solver API.

Tencent CAPTCHA uses `appId`, not `websiteKey`. Find it in the widget configuration on the page:

```javascript
new TencentCaptcha("YOUR_APP_ID", onSolved);
```

Replace `YOUR_APP_ID` with the value from the target page configuration. Store the Captcha Solver API key in the `CAPTCHA_API_KEY` environment variable.

## Choosing the Task Type

Captcha Solver supports two Tencent task types:

- `TencentTaskProxyless` — solve without a client proxy;
- `TencentTask` — solve through a client proxy.

Use `TencentTaskProxyless` when the solve does not require a specific proxy. Use `TencentTask` when the solve request must be executed through your proxy. Supported proxy types are `http`, `socks4`, and `socks5`.

## Repository Structure

```text
examples/
├── python/
│   ├── sync/tencent.py
│   └── async/tencent.py
└── javascript/
    ├── sync/tencent.js
    └── async/tencent.js
```

Each example includes both proxyless and proxy-based tasks.

## Python Examples

Install the Python SDK and optional environment-file support:

```bash
pip install captcha-solver-api python-dotenv
```

Run the synchronous or asynchronous example from the repository root:

```bash
python examples/python/sync/tencent.py
python examples/python/async/tencent.py
```

Source files:

- [Synchronous Python example](examples/python/sync/tencent.py)
- [Asynchronous Python example](examples/python/async/tencent.py)

## JavaScript Examples

Install the example dependencies:

```bash
cd examples/javascript
npm install
```

Run the synchronous-style Promise example or the async/await example:

```bash
npm run sync
npm run async
```

Source files:

- [Promise-based JavaScript example](examples/javascript/sync/tencent.js)
- [Async/await JavaScript example](examples/javascript/async/tencent.js)

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

Pass the complete object to the callback specified when initializing the CAPTCHA:

```javascript
onSolved(solution);
```

`onSolved` is an example. Use the actual callback from the page implementation. Captcha Solver returns the result but does not submit the form automatically.

## Integration Flow

1. Open the page containing Tencent CAPTCHA.
2. Get the `appId` from the page configuration.
3. Create `TencentTaskProxyless` or `TencentTask`.
4. Call the SDK's `solve()` method.
5. Pass the returned object to the page callback.
6. Submit the form and continue with the test assertions.

Get the solution immediately before submitting the form and pass it to the same page where the CAPTCHA was created.

## Custom Tencent CAPTCHA Script URL

If the page loads the Tencent CAPTCHA script from a custom URL, pass it through the optional `captchaScript` parameter. Use this parameter only when the page actually loads a non-default script URL.

## Resources

- [Captcha Solver Python SDK](https://github.com/captcha-solver-api/python-sdk)
- [Python Tencent examples](https://github.com/captcha-solver-api/python-sdk/tree/main/examples)
- [Captcha Solver JavaScript SDK](https://github.com/captcha-solver-api/javascript-sdk)
- [JavaScript Tencent examples](https://github.com/captcha-solver-api/javascript-sdk/tree/main/examples)
- [Tencent CAPTCHA documentation](https://captcha-solver.com/en/docs/methods#tencent)
- [How to Automate Tencent CAPTCHA](https://captcha-solver.com/en/blog/how-to-automate-tencent-captcha)

## Checklist

Before running an example, verify that:

- `websiteURL` points to the page containing Tencent CAPTCHA;
- `appId` matches the widget configuration;
- `CAPTCHA_API_KEY` contains a valid client key;
- `captchaScript` is set only for a custom script URL;
- `TencentTask` contains `proxyType`, `proxyAddress`, and `proxyPort`;
- the callback receives the complete result object.
