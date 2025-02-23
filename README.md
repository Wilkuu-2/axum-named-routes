# Axum Named Routes (But poorly migrated to axum 0.7)

`axum-named-routes` is a library which allows users to easily define routes with names in axum, and get the route path from the route name at runtime.

[![Crates.io](https://img.shields.io/crates/v/axum-named-routes)](https://crates.io/crates/axum-named-routes)
[![Documentation](https://docs.rs/axum-named-routes/badge.svg)](https://docs.rs/axum-named-routes) 

## Safety

- Uses 100% safe rust with `#![forbid(unsafe_code)]`

## Usage Example

```rust
use std::{net::SocketAddr, path::PathBuf};
use axum::routing::get;
use axum_named_routes::{NamedRouter, Routes};

async fn index() -> &'static str {
    "Hello, World!"
}

// gets the routes from axum extensions
async fn nested_other(routes: Routes) {
    // this could panic if the name is not in the Routes map
    // but we know that it is because we got here
    let this_route = routes.has("ui.other");
    assert_eq!(this_route, &PathBuf::from("/ui/other"));
}

async fn other(routes: Routes) {
    // the get function does not panic rather it returns an Option
    let route = routes.get("ui.other");
    let this_route = routes.get("other");
    assert_ne!(route, this_route);
}

#[tokio::main]
async fn main() {
    let ui = NamedRouter::new()
        .route("index", "/", get(index))
        .route("other", "/other", get(nested_other));
    let app = NamedRouter::new()
        .nest("ui", "/ui/", ui)
        .route("other", "/other", get(other));

    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}
```

That example can be found in `examples/simple.rs`.

## Performance

The router uses a `HashMap` internally while creating the map, and wraps it in an `Arc` when it is finished to add it as an axum extension.
So overall the performance cost should be very low.

## License

This project is licensed under the [MIT license](https://choosealicense.com/licenses/mit/)
