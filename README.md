# A Georust Book

This is a collection of lessons to help people get started using [Rust](https://rust-lang.org) for geospatial analysis.

## Pre-requisites for Building

**Install rust**: https://www.rust-lang.org/tools/install

At the time of writing, rust version 1.63 was required to pass all the tests.

**Install our book-building framework:** `mdbook`

    cargo install mdbook

Note: We have disabled playground integration in book.toml for now. This seems to require a newish version of mdbook. Things seem to work with mdbook v0.4.22.

Run the local development service:

    bin/serve

## Run Tests

The lessons include a lot of code snippets. Most of them are runnable. You can build them and verify the assertions within. Some of them are quite slow, so run them in release mode and be patient.

    cargo test --release

