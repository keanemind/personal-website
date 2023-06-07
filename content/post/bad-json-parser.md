---
title: Bad JSON Parser
date: 2023-06-07T06:09:08.808Z
draft: false
---
I realized earlier today that writing even a bad, noncompliant JSON parser is more difficult than I thought. I spent a couple hours and came up with this, which mostly works for valid input:

```javascript
function* tokenizer(input) {
  let isString = false;
  let stringOrKey = undefined; // this is always a string or object key; it can never be any other JSON type
  let value = "";
  for (const c of input) {
    const primitiveToken = {
      type: "PRIMITIVE",
      value:
        value === "null"
          ? null
          : value === "false"
          ? false
          : value === "true"
          ? true
          : parseFloat(value),
    };
    if (isString) {
      if (c === '"') {
        stringOrKey = value;
        isString = false;
        value = "";
      } else {
        value += c;
      }
    } else {
      if (c === '"') {
        if (stringOrKey !== undefined) {
          yield { type: "PRIMITIVE", value: stringOrKey };
          stringOrKey = undefined;
        }
        isString = true;
      } else if (c === "{") {
        if (stringOrKey !== undefined) {
          yield { type: "PRIMITIVE", value: stringOrKey };
          stringOrKey = undefined;
        }
        yield { type: "OBJECT_START" };
      } else if (c === "}") {
        if (stringOrKey !== undefined) {
          yield { type: "PRIMITIVE", value: stringOrKey };
          stringOrKey = undefined;
        }
        if (value.length) {
          yield primitiveToken;
          value = "";
        }
        yield { type: "OBJECT_END" };
      } else if (c === "[") {
        if (stringOrKey !== undefined) {
          yield { type: "PRIMITIVE", value: stringOrKey };
          stringOrKey = undefined;
        }
        yield { type: "ARRAY_START" };
      } else if (c === "]") {
        if (stringOrKey !== undefined) {
          yield { type: "PRIMITIVE", value: stringOrKey };
          stringOrKey = undefined;
        }
        if (value.length) {
          yield primitiveToken;
          value = "";
        }
        yield { type: "ARRAY_END" };
      } else if (c === ",") {
        if (stringOrKey !== undefined) {
          yield { type: "PRIMITIVE", value: stringOrKey };
          stringOrKey = undefined;
        }
        if (value.length) {
          yield primitiveToken;
          value = "";
        }
      } else if (c === ":") {
        if (stringOrKey !== undefined) {
          yield { type: "KEY", value: stringOrKey };
          stringOrKey = undefined;
        }
      } else if (c.match(/\s/)) {
        if (value.length) {
          yield primitiveToken;
          value = "";
        } else {
          // Do nothing, ignore this whitespace
        }
      } else {
        value += c;
      }
    }
  }
  if (stringOrKey !== undefined) {
    yield { type: "PRIMITIVE", value: stringOrKey };
    stringOrKey = undefined;
  }
  if (value.length) {
    yield {
      type: "PRIMITIVE",
      value:
        value === "null"
          ? null
          : value === "false"
          ? false
          : value === "true"
          ? true
          : parseFloat(value),
    };
    value = "";
  }
}

function parse(input) {
  const iter = tokenizer(input);
  const ARRAY_END = Symbol("ARRAY_END");
  function parseToken(iter) {
    const { done, value: token } = iter.next();
    if (done) {
      return undefined;
    }
    if (token.type === "PRIMITIVE") {
      return token.value;
    } else if (token.type === "OBJECT_START") {
      const ret = {};
      let next = iter.next();
      while (next.value.type === "KEY") {
        ret[next.value.value] = parseToken(iter);
        next = iter.next();
      }
      console.assert(next.value.type === "OBJECT_END");
      return ret;
    } else if (token.type === "ARRAY_START") {
      const ret = [];
      let token = parseToken(iter);
      while (token !== ARRAY_END) {
        ret.push(token);
        token = parseToken(iter);
      }
      return ret;
    }
    return ARRAY_END;
  }
  return parseToken(iter);
}
```

The most interesting aspect is that the tokenizer is iterative, whereas the parser is recursive. As far as I am aware, its main limitations are its poor error handling and lack of support for escape codes within strings.
