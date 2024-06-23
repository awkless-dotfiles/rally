<!--
SPDX-FileCopyrightText: 2024 Jason Pena <jasonpena@awkless.com>
SPDX-License-Identifier: MIT
-->

# Rally

Rally is a command-line tool designed for dotfile management. This application
acts as a fancy wrapper over [git][git-scm]. Allowing the user to treat their
home directory as their primary working tree. Thus, through Rally, the user can
use regular [git commands][git-cmds] to manage their dotfiles. Once the user
places their dotfiles under version control, they can begin creating
_collections_ through Rally's special configuration file.

The stand-out feature of Rally involves giving the user the ability to add or
remove their defined collections of dotfiles from their home directory via
special commands that Rally provides. This will be done through the [sparse
checkout][git-sparse-checkout] feature git offers. Thus, the user can manage
their dotfiles and easily deploy them whenever they want without the need to
copy, move, or symlink them.

## Installation

You will need the following pieces of software:

1. Git [>= 2.24.0].
1. Rust [>= 1.74.1].

Clone this repository and use Cargo like so:

```
# git clone https://github.com/awkless-dotfiles/rally.git
# cd rally
# cargo build --release
# cargo install
```

Make sure that your `$PATH` includes `$HOME/.cargo/bin` in order to execute the
Rally binary.

Enjoy!

## Contributing

This coding project is open to the following forms of contribution:

1. Improvements or additions to production code.
1. Improvements or additions to test code.
1. Improvements or additions to build system.
1. Improvements or additions to documentation.
1. Improvements or additions to CI/CD pipelines.

Refer to the [contribution guidelines][contributing] for more information.

## Acknowledgements

Nicola Paolucci's article [Dotfiles: Best way to store a bare git
repository][durdn-article] gave great inspiration to write the Rally
application by providing a basic template on how to use git to manage dotfiles.

The [March 31, 2021 GitHub blog entry][stolee-article] by Derrick Stolee also
gave inspiration for using git's sparse checkout feature to deploy and undeploy
dotfile collections.

## License

The Rally project is under the MIT license. See the provided [copy][license] for
more details.

[git-scm]: https://git-scm.com/
[git-cmds]: https://git-scm.com/docs
[git-sparse-checkout]: https://git-scm.com/docs/git-sparse-checkout
[contributing]: CONTRIBUTING.md
[durdn-article]: https://www.atlassian.com/git/tutorials/dotfiles
[stolee-article]: https://github.blog/2020-01-17-bring-your-monorepo-down-to-size-with-sparse-checkout/
[license]: LICENSE.txt
