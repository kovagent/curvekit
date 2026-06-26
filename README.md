# curvekit

US Treasury yield curve and SOFR overnight rate for Rust. Served from bundled parquet with on-demand fetch and a local cache. No API keys. Offline after the first query.

## Install

```toml
[dependencies]
curvekit = "1.1.0"
```

Until it is published to crates.io, depend on the repository directly:

```toml
curvekit = { git = "https://github.com/userFRM/curvekit" }
```

## Quick start

```rust,no_run
use curvekit::Tenor;

#[tokio::main]
async fn main() -> curvekit::Result<()> {
    let curve = curvekit::treasury_curve_for("2020-03-20").await?;
    let r     = curvekit::treasury_rate_at("2020-03-20", Tenor::Y10).await?;
    let today = curvekit::treasury_today().await?;
    let sofr  = curvekit::sofr_today().await?;

    println!("10Y on 2020-03-20: {r:.6}");
    println!("Latest Treasury:   {}", today.date);
    println!("Latest SOFR:       {:.4}%", sofr.rate * 100.0);
    let _ = curve;
    Ok(())
}
```

## Client pattern

Create the client once and reuse it so the local cache is shared across calls.

```rust,no_run
use curvekit::{Curvekit, Date, Tenor};

#[tokio::main]
async fn main() -> curvekit::Result<()> {
    let client = Curvekit::new();

    // Any date form is accepted.
    let curve = client.treasury_curve("2020-03-20").await?;
    let _ = client.treasury_curve(20200320u32).await?;
    let _ = client.treasury_curve((2020i32, 3u32, 20u32)).await?;
    let _ = client.treasury_curve(Date::today_et()).await?;

    // Named tenors, plus ad-hoc tenors by linear interpolation between knots.
    let r_10y = curve.get(Tenor::Y10).unwrap_or(0.0);
    let r_45d = curve.get(Tenor::days(45));
    println!("10Y: {r_10y:.4}  45D: {r_45d:?}");

    // Interpolated rate in one call.
    let r = client.treasury_rate("2020-03-20", Tenor::Y10).await?;
    println!("10Y interpolated: {r:.6}");

    // Latest SOFR observation.
    let sofr = client.sofr_latest().await?;
    println!("SOFR {}: {:.4}%", sofr.date, sofr.rate * 100.0);

    // Blocking variants for sync code, no async runtime needed.
    let _ = client.treasury_curve_blocking(20200320u32)?;
    let _ = client.treasury_rate_blocking("2020-03-20", Tenor::Y10)?;
    Ok(())
}
```

## CLI

```bash
# Print the Treasury curve for a date.
curvekit-cli get treasury --date 2026-04-14

# Print the SOFR rate for a date.
curvekit-cli get sofr --date 2026-04-14

# Backfill full history.
curvekit-cli backfill

# Append the most recent observation.
curvekit-cli append-today
```

## Data

| Source | Coverage | Published |
|---|---|---|
| US Treasury par yield curve | 2002 to present | ~15:30 ET, business days |
| NY Fed SOFR | 2018-04-02 to present | ~08:00 ET, business days |
| NY Fed EFFR | 1954 to present | ~08:00 ET, business days |
| NY Fed OBFR | 2016-03-01 to present | ~08:00 ET, business days |

Rates come from the US Treasury and the Federal Reserve Bank of New York. Parquet files live under `data/` and are refreshed by an automated nightly job on business days. The first query downloads the year files it needs into the local cache; after that the data is served from disk and works offline.

## Cache

On first use the client downloads each year file into a per-user cache directory (XDG-compliant) and verifies it before writing. Later calls are served from disk. If the network is unavailable a previously cached file is still returned so existing workflows keep running.

| Variable | Effect |
|---|---|
| `CURVEKIT_BASE_URL` | Override the download origin |
| `CURVEKIT_CACHE_DIR` | Override the cache directory |
| `CURVEKIT_MIRROR_URL` | Override the mirror origin |

## API

Full API reference is on [docs.rs](https://docs.rs/curvekit).

## License

Dual-licensed under either of [MIT](LICENSE-MIT) or [Apache-2.0](LICENSE-APACHE) at your option.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).
