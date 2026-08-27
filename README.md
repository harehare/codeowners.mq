<h1 align="center">codeowners.mq</h1>

A [GitHub `CODEOWNERS`](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners) parser and matcher implemented as an [mq](https://github.com/harehare/mq) module.

## Features

- Parses `CODEOWNERS` files (`*`, `**`, `[...]` gitignore-style path patterns)
- Looks up owners for a given path, applying GitHub's "last matching pattern wins" rule
- Renders a Markdown table summarizing all patterns and their owners

## Installation

Copy `codeowners.mq` to your mq module directory, or place it anywhere and reference it with `-L`.

```sh
cp codeowners.mq ~/.local/mq/config/
```

### HTTP Import (no local installation needed)

HTTP imports are disabled by default; pass `--allow-http-import` to import directly from GitHub without any local setup:

```sh
mq --allow-http-import -I raw 'import "github.com/harehare/codeowners.mq" | codeowners::codeowners_for(., "src/index.js")' CODEOWNERS
```

Pin to a specific release with `@vX.Y.Z`:

```sh
mq --allow-http-import -I raw 'import "github.com/harehare/codeowners.mq@v0.1.0" | codeowners::codeowners_for(., "src/index.js")' CODEOWNERS
```

## Usage

```sh
mq -L /path/to/modules -I raw \
  'import "codeowners" | codeowners::codeowners_for(., "src/index.js")' CODEOWNERS
```

If you copied it to the mq built-in module directory:

```sh
mq -I raw 'import "codeowners" | codeowners::codeowners_for(., "src/index.js")' CODEOWNERS
```

## API

| Function | Description |
|---|---|
| `codeowners_parse(input)` | Parses `CODEOWNERS` contents into an array of pattern entries |
| `codeowners_owners(entries, path)` | Returns the owners array for `path` (last matching pattern wins); `[]` if unowned |
| `codeowners_for(text, path)` | Parses `text` and looks up owners for `path` in one call |
| `codeowners_is_unowned(entries, path)` | `true` if no pattern matches `path` |
| `codeowners_to_markdown_table(entries)` | Renders a Markdown table of pattern / owners, in file order |

Each pattern entry is a dict: `{"pattern": <original glob>, "regex": <compiled regex>, "owners": [<string>, ...]}`.

`path` is matched as given — a leading `/` is stripped, but callers are expected to pass a path relative to the repository root using `/` separators.

## Example

Given `CODEOWNERS`:

```
# default owner
*       @global-owner

# JS/TS
*.js    @js-owner @org/js-team
*.tsx   @frontend-team

# docs
/docs/  docs@example.com
```

```sh
mq -I raw 'import "codeowners" | codeowners::codeowners_for(., "src/app.js")' CODEOWNERS
# => ["@js-owner", "@org/js-team"]

mq -I raw 'import "codeowners" | codeowners::codeowners_for(., "docs/guide.md")' CODEOWNERS
# => ["docs@example.com"]

mq -I raw 'import "codeowners" | codeowners::codeowners_for(., "README.md")' CODEOWNERS
# => ["@global-owner"]

mq -I raw 'import "codeowners" | codeowners::codeowners_to_markdown_table(codeowners::codeowners_parse(.))' CODEOWNERS
# | Pattern | Owners |
# | --- | --- |
# | * | @global-owner |
# | *.js | @js-owner, @org/js-team |
# | *.tsx | @frontend-team |
# | /docs/ | docs@example.com |
```

## Compatibility

Requires [mq](https://github.com/harehare/mq) v0.7 or later.

## License

MIT
