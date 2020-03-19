# bash-abbrev-alias
This bash functions provides functionality similar to Vim's abbreviation expansion.

See https://github.com/momo-lab/zsh-abbrev-alias for zsh version.

## Installation
Get bash-abbrev-alias.

```bash
$ cd /path/to/install
$ git clone https://github.com/momo-lab/bash-abbrev-alias.git
```

Append .bashrc file.

```.bashrc
source /path/to/install/bash-abbrev-alias/abbrev-alias
```

## For Example

```bash
$ abbrev-alias -g G="| grep"
$ ps aux G<push space key>
->
$ ps aux | grep 
```

```bash
$ git branch
* master
$ abbrev-alias -ge B='$(git symbolic-ref --short HEAD 2> /dev/null)'
$ git push origin B<push space key>
->
$ git push origin master 
```

## Help
Show `abbrev-alias --help`.

```bash
$ abbrev-alias --help
abbrev-alias 0.1.0
usage: abbrev-alias [OPTIONS] {name=value ...}
       abbrev-alias -u {name ...}

options:
  -c, --command   register as 'alias name=value'
  -g, --global    register as 'alias -g name=value' like
  -e, --eval      evaluates subshells on expansion
  -u, --unset     unregister alias
  -h, --help      show this help
  -v, --version   show version
```
