# curvekit

US Treasury yield curves and SOFR overnight rates for Rust, with an optional bundled-parquet loader.

```toml
[dependencies]
curvekit = "1.1.0"
```

```rust,no_run
use curvekit::Tenor;

#[tokio::main]
async fn main() -> curvekit::Result<()> {
    let r = curvekit::treasury_rate_at("2020-03-20", Tenor::Y10).await?;
    println!("10Y on 2020-03-20: {r:.6}");
    Ok(())
}
```

Full documentation: <https://github.com/userFRM/curvekit>

Licensed under MIT OR Apache-2.0.
