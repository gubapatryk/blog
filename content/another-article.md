+++
title = "another one"
date = 2024-09-24

[taxonomies]
categories = ["demo"]
tags = ["code", "sample"]
+++

tego nie bylo
<!-- more -->

```rust

fn main() {
    println!("Hello, world!");
}
```

Moj kod is a code block with syntax highlighting. It's pretty cool, right?

```python

def main():
    print("Hello, world!")
```

This is another code block with syntax highlighting. It's pretty cool, right?

<!-- prettier-ignore-->
```js

function debounce(func, wait) {
  var timeout;

  return function () {
    var context = this;
    var args = arguments;
    clearTimeout(timeout);

    timeout = setTimeout(function () {
      timeout = null;
      func.apply(context, args);
    }, wait);
  };
}

```
