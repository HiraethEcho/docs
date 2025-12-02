---
title: shell概览
summary: 主要靠ai生成的shell基本知识手册
date: 2025-05-05
tags:
  - shell
categories: handbook
status: draft
ai: true
---

# shell

## intro

A shell is a command-line interpreter that provides a user interface for interacting with the operating system. It allows users to execute commands, run scripts, and manage system processes. Popular shells include bash, zsh, and fish.

### history

The shell originated in the early days of UNIX, providing users a way to interact with the operating system. The Bourne Shell (sh) was one of the earliest, followed by more advanced shells like Bash, Zsh, and Fish. Each new shell introduced features that improved usability, scripting, and customization for users and administrators.

### posix

POSIX stands for "Portable Operating System Interface." It is a set of standardized operating system interfaces and shell command standards, maintained by the IEEE. POSIX ensures compatibility between Unix-like operating systems by defining common APIs, command-line utilities, and scripting interfaces.

In the context of shells, POSIX compliance ensures that shell scripts and commands written for one POSIX-compliant shell will work on another, regardless of the underlying operating system. For example, the `sh` shell is often used as a reference implementation for POSIX-compliant scripting. However, advanced shells like Bash and Zsh often extend POSIX standards with additional features and functionalities.

Key aspects of POSIX include:

1. **Shell Command Language**: Specifies syntax and utilities for scripting.
2. **System Interfaces**: Defines APIs for file system operations, process control, and more.
3. **Utility Programs**: Standardizes common tools like `grep`, `awk`, and `sed`.

POSIX plays a critical role in ensuring interoperability and portability across different Unix-based systems, making it a cornerstone of modern shell environments.

## Start a shell

There are three main cases that start a shell: login, interact, and in the shell scripts.

Linux starts a shell in the following situations:

1. **Login Shell**: Automatically invoked when a user logs in via TTY, SSH, or terminal emulator.
2. **Interactive Shell**: When a user opens a new terminal window or manually starts a shell.
3. **Shell in Scripts**: When a script containing a shebang (e.g., `#!/bin/bash`) is executed, the specified shell is started.

there are also other cases, like ssh, getty, rescure mode etc

### login shell

A shell starts automatically when you log in through a TTY, SSH, or terminal emulator. You can also start a shell manually by running its executable (e.g., bash, zsh, fish) from an existing shell or script. Shells start when prompted by the operating system or when explicitly invoked.

### interact shell

After login, starting a new non-login shell (like opening a new terminal window) usually loads only the interactive shell configuration file (`~/.bashrc` for Bash, `~/.zshrc` for Zsh`). Environment variables and settings exported in the login shell are inherited by the new shell, unless overridden in the new shell's config file.

You can start most shells without loading configuration files by passing specific flags. For example, Bash: `bash --noprofile --norc`; Zsh: `zsh -f`. This launches the shell in a minimal state, ignoring personal and system-wide configuration files.

To start a shell as a different user, use the `su - username` or `sudo -i -u username` commands. The login shell for the specified user will be started, and it will load that user's configuration files (such as those in their home directory). Controlling which files are loaded depends on the shell and options passed; for example, you can use `bash --noprofile --norc` after switching users to avoid loading configs.

When starting a new shell with a different user, to avoid inheriting unwanted configuration, use options that bypass user config files (e.g. for Bash: `bash --noprofile --norc`; for Zsh: `zsh -f`). If you want to inherit configurations from the parent shell, you can manually export the desired environment variables before starting the new shell, or source config files as needed.

By default, when a new shell is started, the present working directory (PWD) is inherited from the parent process (the shell or terminal that launched it). If a shell is started during login (for example, via TTY or SSH), the default directory is usually the user's home directory (`$HOME`).

### shell in DE

A special case is starting a shell in a DE (Desktop Environment).

Launcher programs like dmenu or rofi typically use the shell specified in the `SHELL` environment variable of the running user. If `SHELL` is not set, they may default to `/bin/sh` or whatever is configured in `/etc/passwd` for the user.

The default working directory (PWD) for a shell launched by a program like dmenu or rofi is typically the user's home directory (`$HOME`). However, this can be changed by the launcher or by the environment from which the launcher itself was started.

### shell in script

Yes, when running a shell within a script (using shebang like `#!/bin/bash` or `#!/bin/zsh`), the script may behave differently depending on the shell used. Some shells have distinct syntax, built-ins, or environment handling. Scripts can also use subshells to isolate environments, and you can explicitly invoke subshells with commands like `(command)` or by calling another shell executable within the script.

When a shell is used in a script, it uses the environment variables defined in the parent shell or operating system environment where the script is executed. These may include system-wide variables (e.g., PATH, HOME) and user-defined variables from configuration files (e.g., ~/.bashrc, ~/.zshrc). The script can also define its own environment variables, which will only persist during the script's execution. Additionally, the shell may load configuration files specific to the shell being used, depending on whether the script is executed in a login or non-login context.

For example:

- A script that starts with `#!/bin/bash` as the shebang will inherit environment variables from the parent Bash shell.
- If the script explicitly sources configuration files (e.g., `source ~/.bashrc`), it may load additional variables or settings.
- Subshells started within the script can have isolated environments, which can either inherit variables or define new ones for their scope.

To inspect or modify the environment variables in a script, commands like `export`, `printenv`, or `env` can be used.

Keep in mind that some shells have distinct syntax and behaviors concerning environment variables, so the choice of shell can influence the script's behavior.

To avoid unintended configurations, you can start the shell with options to bypass configuration files or explicitly define the environment variables needed by the script.

Example:

```sh
#!/bin/bash --noprofile --norc
export MY_VARIABLE="value"
echo "Current PATH: $PATH"
```

## configuration

### set shell

To change the default shell for a user, use the command:

```sh
chsh -s /path/to/shell
```

For example, to set Zsh as the default shell:

```sh
chsh -s $(which zsh)
```

### configuration files

zsh

| Shell Type        | System Configurations          | User Configurations        |
| ----------------- | ------------------------------ | -------------------------- |
| Login Shell       | `/etc/zprofile`, `/etc/zlogin` | `~/.zprofile`, `~/.zlogin` |
| Interactive Shell | `/etc/zshrc`                   | `~/.zshrc`                 |
| Shell in a Script | `/etc/zshenv`                  | `~/.zshenv`                |

bash

| Shell Type        | System Configurations                   | User Configurations                              |
| ----------------- | --------------------------------------- | ------------------------------------------------ |
| Login Shell       | `/etc/profile`                          | `~/.bash_profile`, `~/.bash_login`, `~/.profile` |
| Interactive Shell | `/etc/bash.bashrc`                      | `~/.bashrc`                                      |
| Shell in a Script | None (scripts do not load config files) | None                                             |

### syntax

Zsh Configuration Files:

1. **Setting Environment Variables**:

   ```zsh
   export PATH="$PATH:/new/path"
   export EDITOR="vim"
   ```

2. **Defining Aliases**:

   ```zsh
   alias ll="ls -la"
   alias gs="git status"
   ```

3. **Creating Functions**:

   ```zsh
   function greet() {
       echo "Hello, $1!"
   }
   ```

4. **Sourcing Other Files**:
   ```zsh
   source ~/.zshrc
   ```

Bash Configuration Files:

1. **Setting Environment Variables**:

   ```bash
   export PATH="$PATH:/new/path"
   export EDITOR="vim"
   ```

2. **Defining Aliases**:

   ```bash
   alias ll="ls -la"
   alias gs="git status"
   ```

3. **Creating Functions**:

   ```bash
   greet() {
       echo "Hello, $1!"
   }
   ```

4. **Sourcing Other Files**:
   ```bash
   source ~/.bashrc
   ```

## shells

Commonly used shells include:

- Bash (Bourne Again SHell): Default on many Linux systems
- Zsh (Z Shell): Known for advanced features and customization
- Fish (Friendly Interactive Shell): User-friendly and interactive
- sh (Bourne Shell): The original UNIX shell
- tcsh/csh: C Shell and its enhanced version

| Shell    | Pros                                                     | Cons                                         |
| -------- | -------------------------------------------------------- | -------------------------------------------- |
| Bash     | Widely supported, strong scripting capabilities          | Less modern interactive features             |
| Zsh      | Powerful completion, interactive features, customization | Slightly heavier than Bash                   |
| Fish     | User-friendly syntax and autocompletion                  | Less POSIX-compatible for scripting          |
| sh       | Minimal and portable                                     | Limited interactivity and scripting features |
| tcsh/csh | C-like syntax, good for some users                       | Less common, some scripting limitations      |
