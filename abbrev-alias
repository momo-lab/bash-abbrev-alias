typeset -g _ABBREV_ALIAS_VERSION='0.1.0'
typeset -Ag _abbrev_aliases

__abbrev_alias::magic_abbrev_expand() {
  local left=${READLINE_LINE:0:${READLINE_POINT}}
  local right=${READLINE_LINE:${READLINE_POINT}}
  if [[ $left =~ ([-_a-zA-Z0-9#%+.~]+)$ ]]; then
    local command=${BASH_REMATCH[1]}
    left=${left:0:$((${#left} - ${#command}))}
    local abbr=${_abbrev_aliases[$command]}
    [[ -z "$abbr" ]] && return
    [[ $abbr =~ ^([cg])/([01])/(.*)$ ]] || return
    local kind=${BASH_REMATCH[1]}
    local is_eval=${BASH_REMATCH[2]}
    local newbuffer=${BASH_REMATCH[3]}
    local re="(\\||;|&|\\$\\(|\`) ?$"
    [[ "$kind" == "c" && ! ("$left" == "" || "$left" =~ $re ) ]] && return
    if [[ "$is_eval" == "1" ]]; then
      newbuffer=$(eval "echo \"$newbuffer\"")
    fi
    READLINE_LINE="$left$newbuffer$right"
    READLINE_POINT=$(($READLINE_POINT + ${#newbuffer} - ${#command}))
  fi
}

__abbrev_alias::insert_space() {
  local left=${READLINE_LINE:0:${READLINE_POINT}}
  local right=${READLINE_LINE:${READLINE_POINT}}
  local space=" "
  READLINE_LINE="$left$space$right"
  READLINE_POINT=$(($READLINE_POINT + ${#space}))
}

__abbrev_alias::magic_abbrev_expand_and_insert_space() {
  __abbrev_alias::magic_abbrev_expand
  __abbrev_alias::insert_space
}

# bind space
bind -x '"\C-x ": __abbrev_alias::insert_space'
bind -x '" ": __abbrev_alias::magic_abbrev_expand_and_insert_space'

# bind enter
bind -x '"\C-\xFF": __abbrev_alias::magic_abbrev_expand'
bind '"\C-\xFE": accept-line'
bind '"\C-x\C-m": accept-line'
bind '"\C-x\C-j": accept-line'
bind '"\C-m": "\C-\xFF\C-\xFE"'
bind '"\C-j": "\C-\xFF\C-\xFE"'

__abbrev_alias::show() {
  local kind_filter=$1
  local key=$2
  local abbr=${_abbrev_aliases[$key]}
  [[ -z "$abbr" ]] && return 1
  [[ $abbr =~ ^([cg])/([01])/(.*)$ ]] || return
  local kind=${BASH_REMATCH[1]}
  local value=${BASH_REMATCH[3]}
  if [[ $kind_filter == $kind ]]; then
    echo "$key='$value'"
  fi
  return 0
}

__abbrev_alias::list() {
  local kind_filters=$@
  for kind_filter in ${kind_filters}; do
    for key in ${!_abbrev_aliases[@]}; do
      __abbrev_alias::show $kind_filter $key
    done
  done
}

__abbrev_alias::version() {
  echo "abbrev-alias $_ABBREV_ALIAS_VERSION"
}

__abbrev_alias::help() {
  __abbrev_alias::version
  echo "usage: abbrev-alias [OPTIONS] {name=value ...}"
  echo "       abbrev-alias -u {name ...}"
  echo
  echo "options:"
  echo "  -c, --command   register alias as 'alias name=value'"
  echo "  -g, --global    register alias as 'alias -g name=value' like"
  echo "  -e, --eval      evaluates subshells on expansion"
  echo "  -u, --unset     unregister alias"
  echo "  -h, --help      show this help"
  echo "  -v, --version   show version"
}

__abbrev_alias::register() {
  local kind=$1 is_eval=$2 key=${3%%=*} value=${3#*=}
  [[ "$kind" == "c" ]] && alias $key="$value"
  _abbrev_aliases[$key]="$kind/$is_eval/$value"
}

__abbrev_alias::unregister() {
  local key=$1
  if [[ -n "${_abbrev_aliases[$key]}" ]]; then
    unalias $key 2> /dev/null
    unset "_abbrev_aliases[$key]"
  else
    echo "no such alias: $key" >&2
    return 1
  fi
}

# Define command
abbrev-alias() {
  declare -i argc=0
  declare -a argv=()
  local is_eval=0 kind= mode=register
  while (( $# > 0 ))
  do
    case $1 in
      --help) __abbrev_alias::help && return 0 ;;
      --version) __abbrev_alias::version && return 0 ;;
      --unset) mode=unregister ;;
      --command) kind=c ;;
      --global) kind=g ;;
      --eval) is_eval=1 ;;
      -[!-]*)
        [[ "$1" =~ 'h' ]] && __abbrev_alias::help && return 0
        [[ "$1" =~ 'v' ]] && __abbrev_alias::version && return 0
        [[ "$1" =~ 'u' ]] && mode=unregister
        [[ "$1" =~ 'c' ]] && kind=c
        [[ "$1" =~ 'g' ]] && kind=g
        [[ "$1" =~ 'e' ]] && is_eval=1
        ;;
      *)
        ((++argc))
        argv=("${argv[@]}" "$1")
        ;;
    esac
    shift
  done

  # unregister
  if [[ "$mode" == "unregister" ]]; then
    local result=0
    for key in ${argv[@]}; do
      __abbrev_alias::unregister "$key"
      [[ $? -ne 0 ]] && result=1
    done
    return $result
  fi

  # list
  if [[ ${argc} -eq 0 ]]; then
    __abbrev_alias::list ${kind:-g c}
    return 0
  fi

  # register or list
  if [[ "$mode" == "register" ]]; then
    local result=0
    for keyvalue in "${argv[@]}"; do
      if [[ "$keyvalue" =~ = ]]; then
        __abbrev_alias::register ${kind:-c} $is_eval "$keyvalue"
      else
        __abbrev_alias::show ${kind:-c} "$keyvalue"
      fi
      [[ $? -ne 0 ]] && result=1
    done
    return $result
  fi

  # dead code
  return 1
}
