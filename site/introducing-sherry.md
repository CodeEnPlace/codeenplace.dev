---
title: Introducing Sherry
subtitle: A javascript shell scripting multitool
published: 2026-03-10
---

The shell is an awesome abstraction, it allows you to chain together wildly disparate programs into a single script. A shell scripting language is, in some ways, the ancient precursor to LLM agents; that asks:

> What if a program could interact with a computer in the same way a user does, just faster and automated?

## The Problem

The problem is that `bash` and `zsh`, the de facto standard shells have appalling syntax and are generally a pain to use. I mean... just look at this mess:

```bash
set -euo pipefail

for f in /var/log/*.log; do
  base="${f##*/}"
  clean="${base//-/_}"

  if [[ "$clean" =~ ^(sys|kern)_.+\.log$ ]]; then
    case "${clean%%_*}" in
      sys)  priority="high" ;;
      kern) priority="critical" ;;
      *)    priority="low" ;;
    esac

    if [[ -s "$f" ]]; then
      echo "[$priority] ${clean%.log} (${f})"
    else
      echo "[skip] $clean is empty"
    fi
  fi
done
```

`fi`? `esac`? `"${f##*/}"`? Yes, you _can_ learn how to be productive like this, but none of the rest of your team will understand what the hell your script is meant to do, and none of them will be able to confidently update it.

Shell languages are **GREAT** when you want to invoke a series of programs one after the other, but if you try to do anything more complicated than that it quickly becomes untenable. They are, ironically, **not** good languages for _scripting_.

## JavaScript

JavaScript is a real language^[hur durr, "`left-pad`", "wrote it in 10 days", "`[] == ''`". Yes, JavaScript has plenty of problems; it's not 2019 any more, get a grip or close the tab.] It's much clearer to read, it has vital features like JSON parsing built-in, and a package system for anything that isn't covered in the language. But one of the things Js kinda sucks as is subprocess spawning. You can do it, but it never feels quite right.

That's, hopefully, where `sherry` comes in:

```javascript
import $ from "@codeenplace/sherry";

const currentGitHash = await $("git", "rev-parse", "HEAD");

for await (const commitInfo of $("git", "log")) {
  console.log({ commitInfo });
}

const $_PROD = $({ env: { NODE_ENV: "production" } });
await $_PROD("npm", "run", "build");

await $("ls", "|", "rev");

await $("npm", "run", "build", {
  "--setting-one": true,
  "--setting-two": false,
  "--setting-three": "fooBarBaz",
});
```

### Straightforward to Use

Sherry aims to meet you where you need it.

Want to run a command and get the output? Easy:

```javascript
const currentGitHash = await $("git", "rev-parse", "HEAD");
```

Want instead to stream the output and iterate over it? But of course:

```javascript
for await (const commitInfo of $("git", "log")) {
  console.log({ commitInfo });
}
```

Need to set an EnvVar, or capture stderr instead? Easy:

```javascript
await $({
  env: { FOO: "bar" },
  outputs: "stderr",
})("some", "command", "here");
```

It also supports handy syntaxes inspired by [classnames], and exotic arguments like Promises and iterators:

```javascript
await $(
  "echo",
  { "--foo": "bar" }, // key, val style,
  { "not-included": false }, // optional style
  Promise.resolve("baz"),
  function* () {
    yield "qux";
  },
);
```

### Familiar

`sherry` uses a shell under the hood, so you can still access env vars, pipe between processes and into files:

```javascript
await $("echo", "$SHELL");
await $("echo", "foo", "|", "cat");
await $("echo", "foo", ">", "/tmp/file");
```

### Powerful

We're in JavaScript world now, it's so much easier to parse and manipulate data, and know what you're doing:

```javascript
const data = JSON.parse(
  await $(
    "some-command",

    { "--format": "json" },
  ),
);

if (data.ok) {
  await $("do-publish");
} else {
  await $("sudo", "shutdown", "-h", "now");
}
```

## Have a Go

I'm using it, I think it's good, it's on [npm] & [github]. I think if you [try] it out you'll be surprised; Not at how good sherry is, but at how bad bash scripting has always been.

[github]: https://github.com/CodeEnPlace/sherry
[npm]: https://www.npmjs.com/package/@codeenplace/sherry
[classnames]: https://www.npmjs.com/package/classnames
[try]: https://daringfireball.net/2025/04/try_switching_to_kagi
