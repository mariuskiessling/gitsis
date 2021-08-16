# gitsis

> Always use the correct SSH identity file depending on your current
> repository. 

## Why

Working with multiple SSH identities and Git can be hard. You might be working
as a contractor for multiple companies or simply work on personal projects on
your company device. In such cases, you probably deal with multiple SSH
identities. In a normal workflow, all these SSH keys are loaded into an SSH
agent instance and then tried one after another during the authentication phase
with a remote host. In case you want to control which key is preferred for
which host, the [↗ SSH config file](https://linux.die.net/man/5/ssh_config) is
a great place to get started.

This pattern however quickly breaks down in the context of Git where a remote's
host is often the same (e.g. `github.com`) regardless of the repository you are
working on. It is still possible to control the used key used by SSH but this
requires a workaround that either has to be set on the [↗
repository](https://www.fabian-keller.de/blog/configuring-a-different-ssh-key-per-git-repository/)
or [↗ remote](https://stackoverflow.com/a/11251797) level. I personally find
both options unsatisfying because each one of them requires additional steps in
my Git workflow when I set up a repository locally. This :star: should just
work :star2: out of the box every time.

Using the _wrong_ identity is especially a problem when only a subset of your
identities is [authorized for SSO
access](https://docs.github.com/en/github/authenticating-to-github/authenticating-with-saml-single-sign-on/authorizing-an-ssh-key-for-use-with-saml-single-sign-on)
access to a repository on GitHub. In such a case, a key which is not authorized
for a specific organisation will fail the whole interaction instead of
proceeding with the next registered public key.

## Usage

After the installation and configuration using the config file, you do not have
to do anything. Simply perform any Git operation that requires a remote and see
_gitsis_ work its magic.

In case you want to debug what identity file _gitsis_ is using, run

```
export GITSIS_DEBUG=1
```

before executing your Git command. This prompts _gitsis_ to write a debug log
to `/tmp/gitsis.log`.

## Installation

1. Install the required dependencies:
    * [Bash](https://www.gnu.org/software/bash/) (>= v4.2)
    * [jq](https://stedolan.github.io/jq/)

   > :information_source: In case you run MacOS and want to install _gitsis_
   > using [↗ `brew`](https://brew.sh/), jump straight to step 2. Brew will
   > also handle the installation of dependencies.

   On MacOS you can install both dependencies using `brew` by running the
   following command:
   ```
   brew install bash jq
   ```

   On Linux you can install them using e.g. `apt-get` by running the following
   command:
   ```
   apt-get install bash jq
   ```

   To install the dependencies on any other operating system/using any other
   package manager, please consult the respective documentation.

2. Download and install _gitsis_. In case you run MacOS and use `brew`, you can
   do this by executing the following command:

   ```
   brew install mariuskiessling/gitsis/gitsis
   ```

   Alternatively, you can directly download _gitsis_ into a directory that is
   inside your `PATH` and mark it as executable. A good default is
   `/usr/local/bin`. Depending on your system setup, you need root privileges
   to write to this directory. In such a case, prefix the following commands
   with `sudo` or consult the documentation of your operating system.

   ```
   curl -o /usr/local/bin/gitsis https://raw.githubusercontent.com/mariuskiessling/gitsis/main/gitsis
   chmod +x /usr/local/bin/gitsis
   ```

3. Set _gitsis_ as the [↗ SSH command for
   Git](https://git-scm.com/docs/git-config#Documentation/git-config.txt-coresshCommand).

   > :information_source: Replace `/usr/local/bin/gitsis` with your
   > installation path in case you chose a different one in the previous step.

   You can either do this using an environment variable by setting

   ```
   export GIT_SSH_COMMAND="/usr/local/bin/gitsis"
   ```

   inside your shell's init file (e.g. `~/.bashrc` for Bash or
   `~/.config/fish/config.fish` for fish). If you use an alternative shell,
   please consult its documentation to find out how to set the environment
   variable.

   Alternatively, you can set the command directly in Git's configuration by
   running the following command:

   ```
   git config --global core.sshCommand /usr/local/bin/gitsis
   ```

4. Finally, finish the installation by creating your configuration file.

## Configuration

_gitsis_ uses a JSON configuration file. The order in which the file is
discovered is:

1. `$XDG_CONFIG_HOME/gitsis/config.json`
2. `$HOME/.config/gitsis/config.json`

The following properties may be set inside the configuration file:

**.owners**: A map of repository owners and their respective private identity
files. The public keys are automatically derived by SSH.

You may use `~` to refer to your home directory when defining an identity file.
Other substitutions are not allowed.

In case no identity is defined inside _gitsis_' configuration file for a
specific repository owner that is currently being operated on, SSH will try the
[↗ default
keys](https://github.com/openssh/openssh-portable/blob/master/pathnames.h#L71)
in addition to those that may be registered in an SSH agent instance.

### Example

Let's suppose you work for Mega Corp and have two identity files; one for
your personal work and one for your work at Mega Corp. In such a case, your
config file can look like this:

```
{
  "owners": {
    "MY-GITHUB-USERNAME": "~/.ssh/id_personal",
    "mega-corp": "~/.ssh/id_mega_corp"
  }
}
```

## (Un)supported Use Cases

_gitsis_ operates on the assumption that your repositories are hosted under a
path that looks like this:

```
owner/my-repo.git
```

_owner_ can either be your personal username or an organisation name. This
assumption is true for most Git hosting solutions (e.g. GitHub or GitLab). In
case you are operating on a Git host that uses a different path schema, feel
free to open an issue.
