# FX.Rates.Library.v2: UiPath library

A reusable UiPath library with one activity, **GetExchangeRate**, that returns the exchange rate between two currencies on a date. It uses ECB reference rates from the public [Frankfurter API](https://frankfurter.dev). No API key is needed.

Use it in any process that works with foreign currencies: invoices, purchase orders, expense reports, reconciliation or reporting. Example consumer: [uipath-fx-expense-converter-v2](https://github.com/Viktoriia-Bazylevska/uipath-fx-expense-converter-v2).

## Install

1. Get the package from your Orchestrator tenant library feed, or publish it there from this project.
2. In your Studio project: **Manage Packages** → tenant feed → search **FX.Rates.Library.v2** → **Install**.
3. Search for **GetExchangeRate** in the Activities panel.

The library depends only on `UiPath.System.Activities` and `UiPath.WebAPI.Activities`, so it adds no other packages to your project.

## Arguments

| Argument | Direction | Type | Description |
| --- | --- | --- | --- |
| `in_BaseCurrency` | In | String | 3-letter ISO code to convert from, e.g. `USD`. Case-insensitive |
| `in_QuoteCurrency` | In | String | 3-letter ISO code to convert to, e.g. `EUR`. Case-insensitive |
| `in_RateDate` | In | DateTime | Date of the rate. Must not be in the future |
| `out_Success` | Out | Boolean | `True` if a rate was returned |
| `out_Rate` | Out | Decimal | Exchange rate, e.g. `0.89367` |
| `out_RateDate` | Out | DateTime | Date of the rate actually used, as returned by the API |
| `out_ErrorMessage` | Out | String | Reason for failure; empty on success |

## Behaviour

- **Same currency** (e.g. EUR → EUR): returns rate `1` without calling the API.
- **Validation:** codes must be 3 letters and the date must not be in the future; otherwise it fails with "Invalid input" and makes no API call.
- **Rate date:** the API can return a rate for a different date than the one requested. `out_RateDate` is always the date of the rate used, and a Warning is logged when it differs from `in_RateDate`.
- **Retries:** a Retry Scope with 3 attempts, 2 seconds apart; the HTTP request times out after 10 seconds. Timeouts, connection errors and HTTP 5xx are retried. 4xx errors (including 404 and 422) are not retried.
- **The activity never throws.** Always check `out_Success` and `out_ErrorMessage`:

| Message | Cause |
| --- | --- |
| `Invalid input: …` | Bad currency code or a future date |
| `Currency or date not supported` | The API has no rate for this currency or date |
| `Rate service unavailable, try again later` | API error, timeout, or no response after retries |
| `No rate returned for <quote>` | The response held no rate for the target currency |

- **Logging:** Info at start and on success, Warning when the rate date differs from the requested date or validation fails, Error on failure. Full URLs and responses are not logged.

## Examples

**Low-code:** add **GetExchangeRate**, set `in_BaseCurrency` = `"USD"`, `in_QuoteCurrency` = `"EUR"`, `in_RateDate` = `new DateTime(2026, 11, 3)` (C# project) or `New DateTime(2026, 11, 3)` (VB project), then use an **If** on `out_Success`.

**Coded workflow (C#):** Studio generates a service named after the package, so the call is `fXRatesLibraryv2.GetExchangeRate`. It returns the four Out arguments as a tuple:

```csharp
var (success, rate, rateDate, error) = fXRatesLibraryv2.GetExchangeRate(
    in_BaseCurrency: "USD",
    in_QuoteCurrency: "EUR",
    in_RateDate: new DateTime(2026, 11, 3),
    isolated: false);

if (!success)
{
    Log("Rate lookup failed: " + error, LogLevel.Warn);
}
```

Expected result for this example: `success = true`, `rate = 0.89367`, `rateDate = 2026-11-03`.

## Requirements

- UiPath Studio 26.10 or later, Windows project
- Internet access to `api.frankfurter.dev` from the robot

## Version history

| Version | Changes |
| --- | --- |
| 1.0.1 | First release: GetExchangeRate |
| 1.0.2 | 22 Assign activities grouped into 6 Multiple Assigns; no behaviour change |
| 1.0.3 | Removed unused dependencies (Excel, Mail, UIAutomation, Testing, CodedWorkflows); entry point set to `GetExchangeRate.xaml`; no behaviour change |

## Notes

- Rates are ECB reference rates, not bank or card rates.
- Built with UiPath Autopilot (coding agent) and reviewed by a person. Known limitation: one Workflow Analyzer warning (ST-MRD-009, deep nesting) is accepted.
