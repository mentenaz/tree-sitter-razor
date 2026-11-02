# tree-sitter-razor

[Tree-sitter](https://tree-sitter.github.io) grammar for [Razor](https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor) templates — the `.razor` and `.cshtml` file format used by ASP.NET Blazor and Razor Pages.

## Features

- Full Razor syntax: directives (`@page`, `@using`, `@inject`, `@model`, `@inherits`, `@implements`, `@attribute`, `@typeparam`, `@namespace`, `@code`, `@functions`), control structures (`@if`, `@else`, `@foreach`, `@for`, `@while`, `@do`, `@switch`, `@try`, `@lock`, `@using`), expressions (`@expr`, `@(expr)`), and comments (`@* ... *@`)
- HTML elements, attributes, void elements, and self-closing tags
- C# content captured as opaque tokens via an external scanner — no dependency on `tree-sitter-c-sharp`
- C# highlighting injected by editors via `queries/injections.scm`

## Usage

### With Zed

This grammar is used by the [zed-razor](https://github.com/dsbraz/zed-razor) extension. Install it from the Zed extension marketplace.

### Standalone

```bash
npm install
npm run generate   # regenerate src/ from grammar.js
npm test           # run the corpus tests
```

Requires [tree-sitter-cli](https://github.com/tree-sitter/tree-sitter/tree/master/cli).

## Grammar architecture

Razor mixes HTML and C# in a single file. The grammar handles this through an **external scanner** (`src/scanner.c`) that switches between HTML and C# modes depending on context.

C# content (inside `@code { }`, `@if (expr) { }`, `@(expr)`, etc.) is captured as opaque tokens and re-highlighted by the editor via language injection — keeping the grammar free of a C# dependency.

The `word` property is set to `$.identifier` and all Razor keywords are matched as explicit `seq('@', 'keyword')` sequences, which prevents identifiers like `@forecast` from being mis-tokenized as `@for`.

## Queries

| File | Purpose |
|---|---|
| `queries/highlights.scm` | Syntax highlighting for Razor-specific nodes |
| `queries/injections.scm` | Injects `csharp` into C# content nodes |

## About this fork

This is a fork of [dsbraz/tree-sitter-razor](https://github.com/dsbraz/tree-sitter-razor),
maintained by [Mentenaz](https://github.com/mentenaz) for the Razor extension used by
[Mentenaz Forge Super](https://forge.mentenaz-server.com). It differs from upstream in how
markup text is read (see `test/corpus/markup_braces_and_escapes.txt`):

- Braces are ordinary text inside an HTML element, so `<style>`, `<script>`, JSON and
  prose containing `{` and `}` parse without errors. A `}` still closes a Razor block when
  it sits directly in that block's body.
- A quoted attribute value may contain the other kind of quote, so `onclick="showTab('one')"` and
  `data-text='say "hi"'` parse (see `test/corpus/attribute_quotes.txt`).
- `@@` (an escaped at sign) is a `razor_escaped_at` node.
- `@:` (a single line of markup in a code block) is a `razor_text_line` node holding the
  rest of the line as `text_content`; an expression inside that line is not parsed separately.

Known limits: a literal `<` inside `<script>` or `<style>` text still confuses the parser (a
raw-text treatment of those elements would fix it), and `@:` is only recognised in a code
block's body.

## License

MIT