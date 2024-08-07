# Developing Language Extensions

TBD: Document how to create an extension in Zed.

## Extension Capabilities

Extensions are a way to add extend functionality of Zed. Extensions may contain any combination of:

- [grammars](#grammars)
- [languages](#languages)
- [themes](./themes.md)

For example, you can have an extension that provides both a grammar and a language, or one that just provides a theme.

## Extension Structure

TBD: Document example directory structure of an extension:

```
my-extension/
  extension.toml
  languages/
    config.toml
    highlights.scm
```

## extension.toml

A Zed extension is a Git repository that contains an `extension.toml`:

```toml
id = "my-extension"
name = "My extension"
version = "0.0.1"
schema_version = 1
authors = ["Your Name <you@example.com>"]
description = "My cool extension"
repository = "https://github.com/your-name/my-zed-extension"
```

### Grammars

Zed implemented syntax highlighting using tree-sitter grammars. If your extension contains grammars, you can reference the provided grammars in your `extension.toml` like so:

```toml
[grammars.gleam]
repository = "https://github.com/gleam-lang/tree-sitter-gleam"
commit = "58b7cac8fc14c92b0677c542610d8738c373fa81"
```

The `repository` field must specify a repository where the Tree-sitter grammar should be loaded from, and the `commit` field must contain the SHA of the Git commit to use. An extension can provide multiple grammars by referencing multiple tree-sitter repositories.

Upon installation Zed will clone the specified repositories, build and compile the grammars for the WASM target

### Languages config.toml

For each tree-sitter grammar you provide create a `languages/lang_name` directory in your extension. Inside this directory create a `config.toml` file with the following structure:

```toml
name = "Dockerfile"
grammar = "dockerfile"
path_suffixes = ["Dockerfile", "Dockerfile.*"]
line_comments = ["# "]
```

- `name` is the human readable name that will show up in the Select Language dropdown.
- `grammar` is the grammar name as specified in the `extension.toml` and in your `grammar.js` tree-sitter grammar.
- `path_suffixes` (optional) is an array of file suffixes that should be associated with this language. This supports glob patterns like `config/**/*.toml` where `**` matchs 0 or more directories and `*` matches 0 or more characters.
- `line_comments` (optional) is an array of strings that are used to identify line comments in the language.

## config.toml

- `line_comments`: an array of strings that are used to identify line comments in the language.
- `block_comment`: a string that is used to identify block comments in the language.
- `autoclose_before`: a string that specifies the pattern to autoclose before.
- `brackets`: an object that specifies the brackets for the language.
 - `start`: the opening bracket character.
 - `end`: the closing bracket character.
 - `close`: the closing bracket character when it's not at the end of a line.
 - `newline`: whether to treat a newline as a closing bracket.
 - `not_in`: an array of strings that are not treated as brackets.
- `tab_size`: the number of spaces to use for indentation.
- `hard_tabs`: a boolean that specifies whether to use hard tabs or soft tabs.
- `word_characters`: an array of characters that are considered a word boundary.
- `prettier_parser_name`: the name of the Prettier parser to use for this language.
- `opt_into_language_servers`: array of language servers to opt into for the language.
- `first_line_pattern`: a regular expression that matches the pattern of the first line of a file.

TBD: Document additional config.toml keys:
- `code_fence_block_name`:
- `scope_opt_in_language_servers`:
- `increase_indent_pattern`:
- `decrease_indent_pattern`:
- `collapsed_placeholder`:

### Languages tree-sitter queries

Optionally create one or more `*.scm` files inside the `languages/lang_name` directory. These files contain tree-sitter queries that can be used to implement language specific features like syntax highlighting, folding, etc.

See: [Zed Tree Sitter Documentation](../tree-sitter.md)

### Language servers

Zed uses the [Language Server Protocol](https://microsoft.github.io/language-server-protocol/) to provide language support. This means, in theory, we can support any language that has an LSP server. If you wish to provide a language server with your extension, you will need to integrate against the [Zed extension API](https://crates.io/crates/zed_extension_api).

Create a Rust library at the root of your extension repository.

Your `Cargo.toml` should look like this:

```toml
[package]
name = "my-extension"
version = "0.0.1"
edition = "2021"

[lib]
crate-type = ["cdylib"]

[dependencies]
zed_extension_api = "0.0.6"
```

Make sure to use the latest version of the `zed_extension_api` available on crates.io.

In the `src/lib.rs` file in your Rust crate you will need to define a struct for your extension and implement the `Extension` trait, as well as use the `register_extension!` macro to register your extension:

```rs
use zed_extension_api as zed;

struct MyExtension {
    // ... state
}

impl zed::Extension for MyExtension {
    // ...
}

zed::register_extension!(MyExtension);
```

Finally, add an entry to your `extension.toml` with the name of your language server and the language it applies to:

```toml
[language_servers.some-language]
name = "My Extension LSP"
language = "Some Language"
```

TBD: Document additional extension.toml keys: `language_servers.{language_ids, code_action_kinds}`

## Slash commands

TBD: Document extension.toml `slash_commands` and `indexed_docs_providers`.

- Example: https://github.com/zed-industries/zed/blob/13af7c7ebd850ce2e4de500657a35d68fea4b950/extensions/gleam/extension.toml#L17-L28

## Testing your extension

TBD: Document `cmd-shift-x` "Install Dev" Extension. Maybe mention `tail -f ~/Library/Logs/Zed/zed.log` for debugging.

## Submitting your extension

- Fork the [zed-extensions](https://github.com/zed-industries/extensions) repository.
- Add your extension as git submodule in the `extensions` directory.
- Add your extension entry in the `extensions.toml` file.

```toml
[my-extension]
submodule = "extensions/my-extension"
version = "0.0.1"
```

- Run `pnpm sort-extensions` to ensure `extensions.toml` and `.gitmodules` are sorted otherwise the CI will fail.

Now create a pull request with your changes. Once your PR is merged, the extension will be packaged and published to the Zed extension registry.

## Updating your extension

- Update the version in your `extension.toml` file.
- Update the extension submodule to that commit of the new version.
- Make sure the `version` matches the one set in `extension.json` at the particular commit.
- Create a pull request with your changes.

> If you'd like to automate this process, there is a community [GitHub Action](https://github.com/huacnlee/zed-extension-action) you can use.
