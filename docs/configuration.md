# Configuration & Theming

## RustViewConfig

The `RustViewConfig` struct controls server behavior:

```rust
use rustview::prelude::*;

let config = RustViewConfig {
    bind: "0.0.0.0:9000".parse().unwrap(),     // Listen address
    title: "My Dashboard".into(),               // Browser tab title
    session_ttl_secs: 3600,                     // Session timeout (1 hour)
    max_upload_bytes: 10_000_000,               // 10 MB upload limit
    theme: Theme::default(),                    // Custom theme
    layout: Layout::default(),                  // Layout options
};

rustview::run_with_config(app, config);
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `bind` | `SocketAddr` | `127.0.0.1:8501` | Server bind address |
| `title` | `String` | `"RustView App"` | HTML page title |
| `session_ttl_secs` | `u64` | `86400` | Session TTL in seconds (24h) |
| `max_upload_bytes` | `usize` | `52_428_800` | Max upload size (50 MB) |
| `theme` | `Theme` | Dark theme | Color theme |
| `layout` | `Layout` | Full width, 2rem padding | Body layout options |
| `clarity_id` | `Option<String>` | `None` | Microsoft Clarity project ID (omit to disable) |
| `ga_id` | `Option<String>` | `None` | Google Analytics 4 Measurement ID (omit to disable) |

---

## Layout

The `Layout` struct controls the maximum width and padding of the application body.
Only set the fields you need — unspecified fields keep their defaults.
When no layout is configured at all, a sensible default of `800px` max-width with `2rem` padding is applied automatically:

```rust
use rustview::prelude::*;

// Builder pattern — set only what you need:
let layout = Layout::default().with_max_width(80);

// Struct update syntax also works:
let layout = Layout {
    max_width_percent: 80,
    ..Default::default()
};

// Or set everything explicitly:
let layout = Layout::default()
    .with_max_width(80)
    .with_padding("3rem 1rem");

let config = RustViewConfig {
    layout,
    ..Default::default()
};
```

| Field | CSS Variable | Default | Description |
|-------|-------------|---------|-------------|
| `max_width_percent` | `--rustview-max-width` | `0` (uses 800px) | Maximum body width as a percentage of the viewport (1–100). Set to 0 to use the built-in 800px default. |
| `padding` | `--rustview-padding` | `"2rem"` | CSS padding for the app body |

### Builder Methods

| Method | Description |
|--------|-------------|
| `Layout::default().with_max_width(80)` | Set width as viewport percentage (1–100) |
| `Layout::default().with_padding("3rem")` | Set only padding, keep default width |
| `.with_max_width(80).with_padding(s)` | Chain both |

---

## Theming

RustView ships with two built-in presets: **dark** (default) and **light**. You can use them
directly or override individual fields for a fully custom theme.

### Built-in Presets

```rust
use rustview::prelude::*;

// Dark theme — default (navy background, coral accent)
let config = RustViewConfig {
    theme: Theme::dark(),   // same as Theme::default()
    ..Default::default()
};

// Light theme — white background, same coral accent
let config = RustViewConfig {
    theme: Theme::light(),
    ..Default::default()
};
```

### Customizing a Preset

Start from a preset and override only the fields you need:

```rust
let theme = Theme {
    primary: "#6c5ce7".into(),   // change only the accent color
    ..Theme::dark()
};

let config = RustViewConfig {
    theme,
    ..Default::default()
};
```

### Fully Custom Theme

```rust
let theme = Theme {
    background: "#1a1a2e".into(),
    foreground: "#e0e0e0".into(),
    primary: "#e94560".into(),
    secondary_bg: "#16213e".into(),
    border: "#0f3460".into(),
    text_secondary: "#8899aa".into(),
    surface: "#0d0d1f".into(),
    code_fg: "#c0caf5".into(),
};

let config = RustViewConfig {
    theme,
    ..Default::default()
};
```

| Field | CSS Variable | Dark default | Light default | Description |
|-------|-------------|-------------|--------------|-------------|
| `background` | `--rustview-bg` | `#0e1117` | `#ffffff` | Page background |
| `foreground` | `--rustview-fg` | `#fafafa` | `#0e1117` | Primary text color |
| `primary` | `--rustview-primary` | `#ff4b4b` | `#ff4b4b` | Accent color (buttons, links, charts) |
| `secondary_bg` | `--rustview-secondary-bg` | `#262730` | `#f0f2f6` | Input/card background |
| `border` | `--rustview-border` | `#4a4a5a` | `#d0d3de` | Border color |
| `text_secondary` | `--rustview-text-secondary` | `#a3a8b8` | `#6c717e` | Labels and secondary text |
| `surface` | `--rustview-surface` | `#1a1b26` | `#e8eaf0` | Deep surface (code blocks, modal, table headers, sidebar) |
| `code_fg` | `--rustview-code-fg` | `#c0caf5` | `#1e2040` | Monospace code/JSON text color |

### Custom Theme Example — Ocean Theme

```rust
let theme = Theme {
    background: "#0a192f".into(),
    foreground: "#ccd6f6".into(),
    primary: "#64ffda".into(),
    secondary_bg: "#112240".into(),
    border: "#233554".into(),
    text_secondary: "#8892b0".into(),
    surface: "#060d1a".into(),
    code_fg: "#a8d8ea".into(),
};
```

---

## Analytics

RustView supports optional analytics integrations. When an ID field is `None` (the default), **no tracking code is emitted** in the generated HTML — not even an empty script tag.

### Microsoft Clarity

Provide your [Clarity](https://clarity.microsoft.com/) project ID to enable session recording and heatmaps:

```rust
let config = RustViewConfig {
    clarity_id: Some("your_clarity_project_id".into()),
    ..Default::default()
};
```

### Google Analytics 4

Provide your GA4 Measurement ID (starts with `G-`) to enable Google Analytics:

```rust
let config = RustViewConfig {
    ga_id: Some("G-XXXXXXXXXX".into()),
    ..Default::default()
};
```

### Both at the same time

Both services can be active simultaneously:

```rust
let config = RustViewConfig {
    clarity_id: Some("abc123xyz".into()),
    ga_id: Some("G-XXXXXXXXXX".into()),
    ..Default::default()
};
```

---

## Deployment

### Running on All Interfaces

```rust
let config = RustViewConfig {
    bind: "0.0.0.0:8080".parse().unwrap(),
    ..Default::default()
};
```

### Docker

```dockerfile
FROM rust:1.75 AS builder
WORKDIR /app
COPY . .
RUN cargo build --release

FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y libssl-dev ca-certificates && rm -rf /var/lib/apt/lists/*
COPY --from=builder /app/target/release/my-app /usr/local/bin/
EXPOSE 8501
CMD ["my-app"]
```

### Environment Variables

RustView does not read environment variables by default. Configure everything through `RustViewConfig`. You can read env vars in your own code:

```rust
let port: u16 = std::env::var("PORT")
    .unwrap_or_else(|_| "8501".to_string())
    .parse()
    .unwrap();

let config = RustViewConfig {
    bind: format!("0.0.0.0:{port}").parse().unwrap(),
    ..Default::default()
};
```
