# Rust on AWS Lambda

This repository contains multiple crates that make it possible to run programs written in Rust directly as functions in AWS Lambda, while keeping a low footprint with regards to memory consumption, bundle size, and start-up speed.

[![Build Status](https://travis-ci.org/srijs/rust-aws-lambda.svg?branch=master)](https://travis-ci.org/srijs/rust-aws-lambda)

## Usage

### Install

Because this project is still in an early (but functional!) stage, it has not yet been published to the `crates` registry. You will therefore need to depend directly on the Github repository. Add the following to the `[dependencies]` section in your `Cargo.toml` file.

```toml
aws_lambda_runtime = { git = "https://github.com/srijs/rust-aws-lambda" }
```

### Create

The `aws_lamba_runtime` crate provides a runtime environment which will listen for messages from the lambda environment, and call a function every time the lambda is invoked. This function can be async, as the runtime itself is based on top of `futures` and `tokio`.

```rust
extern crate aws_lambda_runtime as lambda;

fn main() {
    // start the runtime, and return a greeting every time we are invoked
    lambda::start(|()| Ok("Hello ƛ!"))
}
```

### Input

To provide input data to your function, you can change the type of the argument that the function accepts. For this to work, the argument type needs to implement the `serde::Deserialize` trait (most types in the standard library do).

```rust
extern crate aws_lambda_runtime as lambda;

use std::collections::HashMap;

fn main() {
    lambda::start(|input: HashMap<String, String>| {
        Ok(format!("the values are {}, {} and {}",
            input["key1"], input["key2"], input["key3"]))
    })
}
```

Additionally, the `aws_lambda_events` crate provides strongly-typed lambda event types for use with [AWS event sources](https://docs.aws.amazon.com/lambda/latest/dg/use-cases.html). To use these types, first add the following to the `[dependencies]` section in your `Cargo.toml` file.

```toml
aws_lambda_events = { git = "https://github.com/srijs/rust-aws-lambda" }
```

Then use the types defined in the crate. For example, this would print out all the `S3Event` record names, assuming your lambda function was subscribed to the [proper S3 events](https://docs.aws.amazon.com/lambda/latest/dg/with-s3-example.html):

```rust
extern crate aws_lambda_runtime as lambda;
extern crate aws_lambda_events;

use aws_lambda_events::event::s3::S3Event;

fn main() {
    lambda::start(|input: S3Event| {
        let mut names = Vec::new();
        for record in input.records {
            names.push(record.event_name);
        }
        Ok(format!("Event names:\n{:#?}", names))
    })
}
```

Note that the types in `aws_lambda_events` are automatically generated from the [official Go SDK](https://github.com/aws/aws-lambda-go/tree/master/events) and thus are generally up-to-date.

### Context

While your function is running you can call `Context::current()` to get additional information, such as the ARN of your lambda, the Amazon request id or the Cognito identity of the calling application.

```rust
extern crate aws_lambda_runtime as lambda;

fn main() {
    lambda::start(|()| {
        let ctx = lambda::Context::current();

        Ok(format!("Hello from {}!", ctx.invoked_function_arn()))
    })
}
```

### Deploy

TBD
