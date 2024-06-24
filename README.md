<!--
SPDX-FileCopyrightText: 2024 Jason Pena <jasonpena@awkless.com>
SPDX-License-Identifier: MIT
-->

# Rally

![Quality Check Status][quality-check-status]
![Reuse v3 Compliance Status][reuse-status]
![Rustc version][rustc-version]

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

## Usage

If you already know how to use git, then you already know how to use 90% of the
Rally binary. The other 10% boils down to additional commands that Rally offers
to control collection definitions. Rally offers a special configuration file
that allows you to control the default settings for its special command set, and
is the main place where you will define your dotfile collections.

This configuration file can be defined in one of two places:
`$HOME/.config/rally/config.toml` or `$HOME/.rallyrc.toml`. The Rally binary
will use the first location it finds the configuration file in the order
previously listed.

By default, Rally uses `$HOME` as the working tree and
`$HOME/.local/share/rally` as the git directory. You can change these locations
by using environment variables `$RALLY_WORK_TREE` and `$RALLY_GIT_DIR`
respectively.

As an example, assume that you have the following dotfile configurations in your
home directory:

- Basic shell configuration stored in `.profile` and `.shrc`.
- Bash configuration stored in `.bash_profile` and `.bashrc`, which sources your
  basic shell configuration.
- Aerc configuration stored in `.config/aerc/*`.
- DWM configuration stored in remote repository `https://github.com/example/dwm.git`.

Now here is how you would tell Rally about your shell configuration collections
in the configuration file:

```
[collection.sh]
    description = "Basic shell configuration"
    dotfiles = [".profile", ".shrc"]

[collection.bash]
    description = "Bash shell configuration"
    dotfiles = [".bash_profile", ".bashrc"]
    depends_on = ["sh"]
```

Now you can call the __deploy__ command to either deploy the _sh_ or _bash_
collection. Notice how in the _bash_ collection definition it uses the
`depends_on` field to add the _sh_ collection as a dependency. Thus, the
following will not only deploy the _bash_ collection, but also deploy the _sh_
collection as a dependency:

```
# rally deploy bash
[INFO Rally] Deploy 'bash' collection
  Dotfiles in 'bash' deployed
    .bash_profile
    .bashrc
  Dependency 'sh' in 'bash' deployed
  Dotfiles in 'sh' deployed
    .profile
    .shrc
```

If you deploy the _sh_ collection, then Rally will only deploy the _sh_
collection's dotfiles and nothing else, because it does not have any
dependencies.

You can also use the __undeploy__ command to remove either _sh_ or _bash_
collections from your home directory. By default the undeploy command will not
remove dependencies of a collection definition unless you tell it to do so:

```
# rally undeploy bash --include-deps
[INFO Rally] Undeploy 'bash' collection
  Dotfiles in 'bash' undeployed
    .bash_profile
    .bashrc
  Dependency 'sh' in 'bash' undeployed
  Dotfiles in 'sh' undeployed
    .profile
    .shrc
```

If you try to undeploy the _sh_ collection directly, then Rally will warn you
about it being a dependency of the _bash_ collection:

```
# rally undeploy sh
[WARN Rally] Collection 'bash' relies on 'sh' as a dependency
Are you sure you want to undeploy 'sh' [y/n]?: y
[INFO Rally] Undeploy 'sh' collection
  Dotfiles in 'sh' undeployed
    .profile
    .shrc
```

Now lets look at the Aerc configuration. Git does not preserve file permissions,
but Aerc needs its configuration directory to have a file permission of 600.
Rally allows you to use hooks to address this issue. Here is an example
collection definition for Aerc in the configuration file:

```
[collection.aerc]
    description = "Configuration for AERC mail client"
    dotfiles = [".config/aerc/*"]

    [collection.aerc.deploy_hooks]
        dotfiles_hook = "chmod 600 #@"
```

Now when you deploy the _aerc_ collection, Rally will execute the contents of
the `dotfiles_hook` field:

```
# rally deploy aerc
[INFO Rally] Deploy 'aerc' collection
  Dotfiles in 'aerc' deployed
    .config/aerc/*
  Collection 'aerc' wants to execute the following hook on its dotfiles:
    chmod 600 .config/aerc/*
  Do you want to run the hook [y/n]? y
  Running hook
```

You may have noticed the strange notation used in the hook definition of the
_aerc_ collection. This notation has a format of `#[n | @]` where _n_ is a
number, and _@_ means "use everything". Thus, `#1` means to use index one of the
`dotfiles` field, and `#@` to use the entire contents of the `dotfiles` field.
This notation also extends to the `submodules_hook` field.

Finally, lets see how you can setup a collection definition for the DWM
configuration. Since this technically is not a dotfile, but a full on codebase
that needs to be compiled and install, you will need to use the `submodules`
field to add it to a collection. However, you need to first make DWM an actual
submodule:

```
# rally submodule add https://github.com/example/dwm.git $HOME/.local/share/dwm
# rally commit -m "Add DWM as a submodule"
```

Now you can setup DWM as a collection in the configuration file like so:

```
[collection.dwm]
    description = "Patched version of DWM"
    submodules = [".local/share/dwm"]

    [collection.dwm.deploy_hooks]
        submodules_hook = """
            cd #0 || exit
            make clean
            make
            sudo make install
            cd "$HOME" || exit
        """
    [collection.dwm.undeploy_hooks]
        submodules_hook = """
            cd #@ || exit
            sudo make uninstall
            cd "$HOME" || exit
        """
```

Notice that you now have hooks for both the deploy and undeploy commands that
will execute on the `submodules` field. Here is what deploy command will do
when you run it on the _dwm_ collection:

```
# rally deploy dwm
[INFO rally] Deploy 'dwm' collection
  Submodules in 'dwm' deployed
    .local/share/dwm
  Collection 'dwm' wants to execute the following hook on its submodules:
    cd .local/share/dwm || exit
    make clean
    make
    sudo make install
    cd "$HOME" || exit
  Do you want to run the hook [y/n]?: y
  Running hook
```

Here is what would happen if you were to undeploy the _dwm_ collection:

```
# rally undeploy dwm
[INFO rally] Undeploy 'dwm' collection
  Submodules in 'dwm' undeployed
    .local/share/dwm
  Collection 'dwm' wants to execute the following hook on its submodules:
    cd .local/share/dwm || exit
    sudo make uninstall
    cd "$HOME" || exit
  Do you want to run the hook [y/n]?: y
  Running hook
```

Finally, you can list all your current collections you have defined in the
configuration file through the __list-collections__ command:

```
# rally list-collections
[INFO rally] Currently defined collections
  aerc  Configuration for AERC mail client  (undeployed)
  bash  Bash shell configuration            (deployed)
  dwm   Patched version of DWM              (undeployed)
  sh    Basic shell configuration           (deployed)
```

You can control the overall verbosity of the Rally binary via the `-v` flag.
You can get more usage information from the Rally binary via the `--help`
command. Each of the commands just described to you also come with their own
`--help` for more information about themselves individually.

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

[quality-check-status]: https://img.shields.io/github/actions/workflow/status/awkless-dotfiles/rally/quality_check.yaml?style=flat-square&label=quality%20checks
[reuse-status]: https://img.shields.io/github/actions/workflow/status/awkless-dotfiles/rally/reuse.yaml?style=flat-square&label=reuse%20v3%20compliance
[rustc-version]: https://img.shields.io/badge/rustc-1.74.1%2B-blue
[git-scm]: https://git-scm.com/
[git-cmds]: https://git-scm.com/docs
[git-sparse-checkout]: https://git-scm.com/docs/git-sparse-checkout
[contributing]: CONTRIBUTING.md
[durdn-article]: https://www.atlassian.com/git/tutorials/dotfiles
[stolee-article]: https://github.blog/2020-01-17-bring-your-monorepo-down-to-size-with-sparse-checkout/
[license]: LICENSE.txt
