<!--
SPDX-FileCopyrightText: 2024 Jason Pena <jasonpena@awkless.com>
SPDX-License-Identifier: MIT
-->

# Contributing

Thanks for taking the time to contribute! Remember, the information in this
documentation are just _guidelines_. Always use your best judgement!

## Expected Forms of Contribution

This project is open to the following forms of contribution:

1. Improvements or additions to production code.
1. Improvements or additions to test code.
1. Improvements or additions to build system.
1. Improvements or additions to documentation.
1. Improvements or additions to CI/CD pipelines.

Use the provided templates for bug reports, feature requests, and pull requests.
Please only use the bug tracker for reporting bug reports, or feature request
submissions.

Pull request submissions must occur on a separate branch, and compared by the
current state of the `main` branch. Keep your commit history linear. Linear
commit history makes it easier to perform rebase merging. This project does not
like merge commits.

## Commit Style

This project likes to use a style I'd call "purposeful
[conventional commits][conventional-commits]". Instead of classifying every
commit using _conventional commit_ messaging, you do so only if the message
should show up in the changelog. This project likes to use [Cargo Smart
Release][smart-release] for changelog generation.

Commit message __must__ show up in the changelog in case of breaking changes.
Examples for that are:

- change!: rename `Foo` to `Bar`

  And this is why we do it in the body.

- remove!: `Repo::obsolete()`

  Nobody used this method.

Features or other changes that are visible and people should know about look
like this:

- feat: add `Repo::foo()` to do great things (#234)

  And here is how it's used and some more details.

- fix: don't panic when calling `foo()` in bare repository (#456)

Everything else, particularly refactors or chorse, do not use _conventional
commits_ as these do not affect production code.

Commits should use the _subject_ to inform about the _what_, and the _body_
provides details about the _why_. The _how_ should be left to the code itself.
Examples could be:

- Simplify implementation of deploy command
- Add `some_dependency` to Cargo.toml
- Thanks clippy

Please refrain from using `chore:` or `refactor:` prefixes as for the most part,
users of Rally do not care about those. When a `refactor` changes the codebase
in some way, prefer to use `feat`, `change`, `rename`, or `remove` instead,
and most of the time the ones that are not `feat` are breaking changes so
would be seen with their _exclamation mark_ suffix, like `change!`.

Finally, maximum line width for commits is 80 characters. This includes the
_subject_, _body_, and _trailers_ of any commit. The _subject_ of a commit
should be capitalized, and written in the imperative mood. If you can, do not
use ending punctuation like periods, question marks, or exclamation marks.

[conventional-commits]: https://www.conventionalcommits.org/en/v1.0.0/
[smart-release]: https://github.com/Byron/cargo-smart-release
