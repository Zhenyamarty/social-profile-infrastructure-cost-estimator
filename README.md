# Social Profile Infrastructure Cost Estimator

A small reference project for estimating the monthly infrastructure cost of social media automation workflows that use cloud phones, browser profiles, proxy traffic, and mobile minutes.

For teams checking [Multilogin pricing](https://multilogin.com/pricing/?utm_source=github&utm_medium=media&utm_campaign=link1Z), the current structure includes a free plan with no time limit, while paid plans start from **$7.08 per month**. The platform includes cloud phones, browser profiles, built-in proxies, and mobile minutes, which matters when estimating the real cost of a social profile infrastructure stack.

The point of this repository is not to compare plans by a single headline price. It gives developers and operations teams a simple way to model base subscription cost separately from usage-based expenses.

## What should be included in a monthly cost estimate?

A multi-account workflow can contain several cost layers:

| Cost layer | How to model it |
| --- | --- |
| Base subscription | Fixed monthly amount |
| Cloud phone access | Check what the selected plan includes |
| Mobile minutes | Included allowance plus any additional usage |
| Proxy traffic | Included allowance plus additional GB |
| Browser profiles | Check whether they are included in the platform |
| External automation | Add API, CI, hosting, or monitoring costs separately |

Keeping these values separate is useful because two teams paying the same subscription price may have very different monthly totals.

One team might leave cloud phones running for long test sessions, while another may rely mainly on browser profiles and use very little mobile time.

## Which pricing values can be modeled?

The current pricing information provides several useful inputs for an estimator:

| Item | Reference value |
| --- | ---: |
| Free plan | $0, no time limit |
| Paid plans | From $7.08/month |
| Additional mobile minutes | $0.0073 to $0.009 per minute |
| Additional proxy traffic | From $2.50/GB |
| Browser profiles | Included in the platform |
| Built-in proxies | Included in plans |
| Mobile minutes | Included in plans |

The mobile-minute rate depends on the package size. Larger packages can reach approximately **$0.0073 per minute**, while smaller packages can be around **$0.009 per minute**.

For budgeting, I normally keep the included allowance and the additional usage on separate lines. A small discrepancy is much easier to find when the calculator does not mix both into one number.

## How can the monthly cost be calculated in Python?

Create a file named `estimator.py`:

```python
from decimal import Decimal


def monthly_cost(
    base_subscription: Decimal,
    extra_mobile_minutes: int,
    minute_rate: Decimal,
    extra_proxy_gb: Decimal,
    proxy_rate: Decimal,
) -> dict[str, Decimal]:
    mobile_cost = Decimal(extra_mobile_minutes) * minute_rate
    proxy_cost = extra_proxy_gb * proxy_rate

    total = base_subscription + mobile_cost + proxy_cost

    return {
        "base_subscription": base_subscription,
        "extra_mobile_minutes": mobile_cost,
        "extra_proxy_traffic": proxy_cost,
        "estimated_total": total,
    }


if __name__ == "__main__":
    estimate = monthly_cost(
        base_subscription=Decimal("7.08"),
        extra_mobile_minutes=1000,
        minute_rate=Decimal("0.009"),
        extra_proxy_gb=Decimal("5"),
        proxy_rate=Decimal("2.50"),
    )

    for item, amount in estimate.items():
        print(f"{item}: ${amount:.2f}")
```

Run it with:

```bash
python estimator.py
```

Example output:

```text
base_subscription: $7.08
extra_mobile_minutes: $9.00
extra_proxy_traffic: $12.50
estimated_total: $28.58
```

This example deliberately uses the higher **$0.009 per-minute** reference rate. If your package uses another rate, replace the value before calculating.

Python's `Decimal` type is used instead of a regular floating-point value because infrastructure pricing often involves fractions of a cent. The Python standard library provides `Decimal` for decimal arithmetic where exact decimal representation matters.

## How should configuration be separated from the calculator?

For a reusable project, keep the rates outside the Python file.

For example, create `example-costs.yaml`:

```yaml
currency: USD

subscription:
  monthly: 7.08

usage:
  extra_mobile_minutes: 1000
  mobile_minute_rate: 0.009

  extra_proxy_gb: 5
  proxy_rate_per_gb: 2.50
```

That makes it easier to update rates without changing calculation logic.

A larger project could keep separate files for different usage scenarios:

```text
cost-scenarios/
├── small-team.yaml
├── agency.yaml
├── mobile-heavy.yaml
└── browser-heavy.yaml
```

## What does a small-team scenario look like?

Suppose a workflow has:

```text
Base subscription         $7.08
Extra mobile minutes      1,000
Minute rate               $0.009
Extra proxy traffic       5 GB
Proxy rate                $2.50/GB
```

The calculation becomes:

```text
Mobile usage = 1,000 × $0.009 = $9.00
Proxy usage  = 5 × $2.50      = $12.50

Estimated monthly total:
$7.08 + $9.00 + $12.50 = $28.58
```

This is an example cost model, not a quote. Included allowances and current rates should always be checked before using the result for purchasing or forecasting.

## Why not compare only the subscription price?

The base price tells only part of the story.

For a cloud-phone workflow, the more useful question is:

```text
monthly cost =
    base subscription
    + additional mobile usage
    + additional proxy traffic
    + external automation infrastructure
```

External infrastructure might include a CI runner, database, logging service, or application hosting. Those costs belong in the same operational budget even though they are billed by different vendors.

This approach also prevents an easy comparison mistake: treating a service that includes a resource and a service that charges for it separately as though their headline prices represented the same thing.

## How could this estimator be expanded?

A production version could read YAML or JSON configuration and calculate several scenarios in one run.

For example:

```python
scenarios = {
    "small_team": {
        "profiles": 5,
        "mobile_minutes": 1000,
    },
    "agency": {
        "profiles": 30,
        "mobile_minutes": 8000,
    },
}
```

You could then export the results to CSV for finance review or feed the numbers into an internal dashboard.

For automated workflows, the same script could run in GitHub Actions whenever pricing configuration changes and attach the new estimate as a build artifact.

## What should be checked before using the estimate?

Pricing data changes, so do not treat values stored in a repository as permanent constants.

Before making a budget decision:

- Check the current plan price.
- Confirm how many mobile minutes are included.
- Check the current additional-minute rate for the package size.
- Confirm the included proxy allowance.
- Check the current rate for additional proxy traffic.
- Add any separate hosting, API, CI, or monitoring expenses.

The calculator is intentionally small so that these assumptions remain visible rather than disappearing inside a large spreadsheet.

## Project structure

```text
social-profile-infrastructure-cost-estimator/
├── README.md
├── estimator.py
├── example-costs.yaml
└── LICENSE
```

## References

- [Python decimal module](https://docs.python.org/3/library/decimal.html)
- [Multilogin pricing](https://multilogin.com/pricing/)
