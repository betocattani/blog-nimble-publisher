%{
  title: "Installing and config ASDF to Elixir and Erlang",
  author: "Luiz Cattani",
  tags: ~w(elixir phoenix process version_manager erlang),
  description: "Using the asdf tool to version manager of elixir and erlang"
}
---

# Install and config ASDF

Go to the asdf documentation and select the operating System that you uses.
https://asdf-vm.com/#/core-manage-asdf

Installing `asdf` using Homebrew
```bash
$ brew install asdf
```

Install Dependencies
```bash
$ brew install autoconf
$ brew install wxmac
```

Set ENV var to not install java
```bash
export KERL_CONFIGURE_OPTIONS="--disable-debug --without-javac"
```
Add a plugin to asdf
```bash
$ asdf add plugin erlang
```

List all erlang versions available
```bash
$ asdf list-all erlang
# ...
# 23.3.4
# 23.3.4.1
# 24.0-rc1
# 24.0-rc2
# 24.0-rc3
# 24.0
# 24.0.1
```

Install specifi version of erlang
```bash
$ asdf install erlang-24.0.1
```

Set globally the erlang version
```bash
$ asdf global erlang 24.0.1
```

List all elixir versions available
```bash
$ asdf list-all elixir
# ...
# 1.12.0-rc.1-otp-22
# 1.12.0-rc.1-otp-23
# 1.12.0-rc.1-otp-24
# master
# master-otp-21
# master-otp-22
# master-otp-23
# master-otp-24
```

Install a version of elixir to a specific OTP version
```bash
$ asdf install elixir 1.12.0-rc.1-otp-24
```

Set globally the elixir version
```bash
$ asdf global elixir 1.12.0-rc.1-otp-24
```

Verify the elixir version on the terminal
```bash
$ elixir -v
Erlang/OTP 24 [erts-12.0.1] [source] [64-bit] [smp:4:4] [ds:4:4:10] [async-threads:1] [jit] [dtrace]

Elixir 1.12.0 (compiled with Erlang/OTP 23)
```

After installation, restart the terminal to assume the configuration.

List all commands
```bash
$ asdf help
```