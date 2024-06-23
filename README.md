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

[git-scm]: https://git-scm.com/
[git-cmds]: https://git-scm.com/docs
[git-sparse-checkout]: https://git-scm.com/docs/git-sparse-checkout
