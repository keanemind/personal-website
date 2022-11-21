---
title: How to Use the EdgeDB Rust Client
date: 2022-11-21T03:30:08.504Z
draft: false
---
For my latest small project with Rust, which I'm working on mainly to get better acquainted with the language, I chose to use EdgeDB, a new database that I've had my eye on for several months. Unfortunately, I've found that the EdgeDB Rust client is not particularly well documented. It seems that much more effort was put into documenting the TypeScript client with guides for common use cases, and the Rust client could certainly use some love in that regard.

I will briefly describe how to write simple queries using the Rust client so you can avoid the trial and error that I went through.

Suppose your schema looks like this:
```
  type User {
    required property username -> str {
      constraint exclusive;
      constraint min_len_value(1);
    }

    property password -> str;
  }
```

Here's what an insert could look like:
```
let result = conn
        .query_required_single_json(
            "insert User {
        username := <str>$0,
        password := <str>$1
    };",
            &("someone@example.com", "hunter2"),
        )
        .await;
```

The EdgeQL cast `<str>` is necessary in front of each parameter in the query to avoid getting a runtime error. I didn't see this documented anywhere. The arguments in the query must have a cast that matches the Rust type being passed in. I don't believe the documentation contains the mapping between Rust types and EdgeQL types either; I mostly used guessing and checking to get my queries to work.

If you have an enum in your schema, you can insert it by passing a `&str` with the enum's exact name.

Suppose your schema looks like this:
```
scalar type MyEnum extending enum<ChoiceA, ChoiceB>;

type ContainsEnum {
  required property e -> MyEnum;
} 
```

Here's what an insert could look like:
```
let result = conn
    .query_required_single_json(
        "insert ContainsEnum {
            e := <MyEnum><str>$0
        };",
        &(
            "ChoiceB",
        ),
    )
    .await;
```

On 4th line, the `<MyEnum>` cast is not required in `<MyEnum><str>` due to [implicit casting](https://www.edgedb.com/docs/reference/edgeql/casts#implicit-casts). It's only there for clarity.

Being a complete beginner, I wasn't able to figure out a way to use the data returned by the non-json functions, like `query_required_single`. Instead, I used the json version of each query function and used `serde_json::from_str` to get the values I needed out of the result.