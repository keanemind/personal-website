---
title: Generating a Schema for a 5 GB JSON File in 45s
date: 2023-06-06T06:24:04.182Z
draft: false
---
Earlier this year, as part of a database migration from Firebase RTDB to Postgres, I was faced with figuring out the schema of a 5 GB JSON file. The file was large due to the JSON's width, rather than depth; it contained some objects which had more than 100,000 keys, and several large arrays. Those large objects had dynamically-generated keys (such as uuidv4s) which would be ids mapping to other entities. Since the RTDB instance had been in use for over two years as business requirements and the schema evolved, many of the fields within certain entities were long forgotten.

Here’s an example:

```
{
    "users": {
        "[some uuid]": {
            "createdAt": "really old",
            "fieldWeDontUseAnymore": "some value",
            ...
        },
        "[some uuid]": {
            "createdAt": "less old",
            "fieldWeActuallyUse": "some value",
            ...
        },
	...
    }
}
```

We wanted to know what possible values and types could be at a given path within the JSON. However, all of the tools I found for generating JSON schemas from JSON files assumed smaller file sizes, and thus exhaustively included all keys in all objects, and all elements of all arrays, in the resulting schema. I was not interested in receiving a schema where every uuid in the users object was listed out. 

Additionally, most of these tools and libraries worked by first parsing the entire JSON file in memory, followed by generating the schema afterward. Due to the extra overhead of data structures in Python and JavaScript compared to the raw JSON string, attempting to parse the file in those runtimes ballooned memory usage beyond what was available on my laptop, leading to stalling or crashing.

I thus decided to write my own schema generator that would iterate over the input JSON file in a streaming manner, generating a schema on the fly. Upon determining that an object or array meets a size threshold as the program landed upon the nth object key or array element, it would collapse that object or array's schema, combining the schemas of the children recursively.

Say, for example, that our input JSON file is the earlier example with the users object containing uuid keys. My program would first see the open brace, and determine that, so far, the schema is:

```typescript
{}
```

It would then encounter the key `"users"`, and would thus update the schema to be:

```typescript
{
    users: unknown;
}
```

It would then see another open brace. After processing several user objects, the schema would match exactly a subset of the full users object, and look something like this (written as a TypeScript type):

```typescript
{
    users: {
        uuid0: {
            createdAt: "really old";
            fieldWeDontUseAnymore: "some value";
            nestedField: {
                key: "value";
            }
        },
        uuid1: {
            createdAt: "less old";
            fieldWeActuallyUse: "some value";
            nestedField: {
                key: "different value";
                anotherKey: "another value";
            }
        }
    }
}
```

See how the TypeScript type looks exactly the same as the JSON? That’s because it’s as if you used `const` for that object in TypeScript; the schema/type matches the object exactly—for now.

Since the users object is large, we haven't yet seen the closing brace for the users object, and so we carry on adding more and more to the schema. Upon reaching the 51st new key under the users object, the program would realize that the users object likely contains dynamically-generated keys, rather than a fixed set of human-readable keys, such as those within an individual user object inside the giant users object. The program would subsequently collapse the subschemas within the large users object schema by merging all of the schemas of each individual users object that has been seen. As it merges the schemas of the object under the `uuid0` key and the object under the `uuid1` key, it would also have to recurse and merge the schemas under the `nestedField` keys, since they map to non-primitive values. The result would look like this:

```typescript
{
    users: {
        [key: string]: {
            createdAt: "really old" | "less old";
            fieldWeDontUseAnymore?: "some value";
            fieldWeActuallyUse?: "some value";
            nestedField: {
                key: "value" | "different value";
                anotherKey?: "another value";
            }
        }
    }
}
```

Now, as it continues processing the rest of the user objects, it would continuously merge each user object's schema into the wildcard schema. As part of recursively merging subschemas, if too many possibilities of literal values for a certain field are reached, that field's type would go from a set of literal values to a generic type, such as string. So eventually, after processing every user object inside `"users"`, the schema might look something like this :

```typescript
{
    users: {
        [key: string]: {
            createdAt: string;
            fieldWeDontUseAnymore?: string;
            fieldWeActuallyUse?: string;
            nestedField: {
                key: string;
                anotherKey?: string | number;
                yetAnotherKey?: "enum choice 1" | "enum choice 2";
            }
        }
    }
}
```

Collapsing the schema keeps it readable and also ensures that memory usage only scales linearly with the depth of the JSON structure, rather than its width. This is because the widest object or array schema held in memory would have at most 50 keys or elements.

I wrote my first implementation using the [bfj npm package](https://www.npmjs.com/package/bfj). Unfortunately, although the bfj README mentions that it uses Bluebird promises to avoid a memory leak with the stdlib Promises, I still saw a memory leak when using the library. On my laptop my script was consuming upwards of 20 GB of RAM after running for 20 minutes. A look at a profiler showed that most of the memory in use was from closures coming from the Bluebird promises. 

I decided to take control over the memory usage of the program by translating my algorithm into Rust. `cargo run` took 30 minutes before spitting out a schema. I was already celebrating that it worked at all before I realized that I had built with the dev profile. Using the release flag, it ran in 45 seconds and never used more than 50 MB of memory. For the first time, my rudimentary knowledge of Rust had paid off!

> Side note: I tried and failed to get ChatGPT 3.5 to write the program for me. It kept trying to parse the entire file at once. I concede that my inexperience with prompting may have been at fault.

The output revealed that our user objects had more than 90 possible fields, of which only 40 were known before running my schema generator. And every single one of those 90 fields was optional, meaning that none of them were present in every user object. Granted, there were many thousands of user objects created over many months, but I would have expected at least one common field like "createdAt" or something. 

Our live code definitely expected some of those user object fields to always exist, so if certain users with ancient user objects tried to use the site, they would have encountered runtime errors from null pointers and other problems. No wonder people care about data integrity!