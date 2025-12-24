#!/bin/bash
# =========================================================================
# SKRYPT NAPRAWCZY: ZASTĘPUJE CAŁY ~/.bashrc POPRAWNYM PLIKIEM
# =========================================================================

echo "=== ZASTĘPOWANIE ~/.bashrc POPRAWNYM PLIKIEM ==="

# 1. Tworzymy NOWY, POPRAWNY ~/.bashrc
cat > /tmp/new_bashrc << 'EOF'
# ~/.bashrc: executed by bash(1) for non-login shells.
# see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
# for examples

# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

# don't put duplicate lines or lines starting with space in the history.
# See bash(1) for more options
HISTCONTROL=ignoreboth

# append to the history file, don't overwrite it
shopt -s histappend

# for setting history length see HISTSIZE and HISTFILESIZE in bash(1)
HISTSIZE=1000
HISTFILESIZE=2000

# check the window size after each command and, if necessary,
# update the values of LINES and COLUMNS.
shopt -s checkwinsize

# If set, the pattern "**" used in a pathname expansion context will
# match all files and zero or more directories and subdirectories.
#shopt -s globstar

# make less more friendly for non-text input files, see lesspipe(1)
[ -x /usr/bin/lesspipe ] && eval "$(SHELL=/bin/sh lesspipe)"

# set variable identifying the chroot you work in (used in the prompt below)
if [ -z "${debian_chroot:-}" ] && [ -r /etc/debian_chroot ]; then
    debian_chroot=$(cat /etc/debian_chroot)
fi

# set a fancy prompt (non-color, unless we know we "want" color)
case "$TERM" in
    xterm-color|*-256color) color_prompt=yes;;
esac

# uncomment for a colored prompt, if the terminal has the capability; turned
# off by default to not distract the user: the focus in a terminal window
# should be on the output of commands, not on the prompt
#force_color_prompt=yes

if [ -n "$force_color_prompt" ]; then
    if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
        # We have color support; assume it's compliant with Ecma-48
        # (ISO/IEC-6429). (Lack of such support is extremely rare, and such
        # a case would tend to support setf rather than setaf.)
        color_prompt=yes
    else
        color_prompt=
    fi
fi

if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
unset color_prompt force_color_prompt

# If this is an xterm set the title to user@host:dir
case "$TERM" in
xterm*|rxvt*)
    PS1="\[\e]0;${debian_chroot:+($debian_chroot)}\u@\h: \w\a\]$PS1"
    ;;
*)
    ;;
esac

# enable color support of ls and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    alias ls='ls --color=auto'
    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
fi

# some more ls aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'

# Alias definitions.
if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi

# =========================================================================
# FUNKCJA getscript - System wymiany skryptów (v3.0 - DZIAŁAJĄCA)
# =========================================================================
getscript() {
    # 1. Pobierz adres źródłowy z pliku ustawień
    SETTINGS_URL="https://raw.githubusercontent.com/Lisek999/AI-firma-skrypty-operacyjne/main/SKRYPTY_OPERACYJNE.md"
    SOURCE_URL=$(curl -s "$SETTINGS_URL")

    if [ -z "$SOURCE_URL" ]; then
        echo "BŁĄD: Nie udało się pobrać adresu źródłowego z pliku ustawień."
        return 1
    fi

    # 2. Pobierz skrypt
    SCRIPT_CONTENT=$(curl -s "$SOURCE_URL")

    if [ -z "$SCRIPT_CONTENT" ]; then
        echo "BŁĄD: Nie udało się pobrać skryptu z: $SOURCE_URL"
        echo "Sprawdź, czy plik EDITOR.md nie jest pusty."
        return 1
    fi

    # 3. Przygotuj ścieżkę zapisu i nazwę pliku
    SCRIPT_DIR="$HOME/skrypty"
    mkdir -p "$SCRIPT_DIR"

    if [ -n "$1" ]; then
        FILENAME="$1.sh"
    else
        FILENAME="skrypt_$(date +%Y%m%d_%H%M%S).sh"
    fi

    FULL_PATH="$SCRIPT_DIR/$FILENAME"

    # 4. Zapisz skrypt do pliku
    echo "$SCRIPT_CONTENT" > "$FULL_PATH"
    chmod +x "$FULL_PATH"

    echo "========================================"
    echo "SKRYPT ZAPISANY JAKO: $FULL_PATH"
    echo "ŹRÓDŁO: $SOURCE_URL"
    echo "========================================"

    # 5. Wykonaj zapisany skrypt
    echo "URUCHAMIANIE..."
    echo "---"
    bash "$FULL_PATH"
}
EOF

# 2. Zastąp stary plik nowym
echo "Zastępuję ~/.bashrc nowym, poprawnym plikiem..."
cp /tmp/new_bashrc ~/.bashrc

# 3. Przeładuj
echo "Przeładowuję ~/.bashrc..."
source ~/.bashrc 2>/dev/null || echo "Uwaga: Przeładowanie zwróciło błąd, ale plik jest zaktualizowany."

# 4. Sprawdź
echo "Sprawdzam funkcję getscript..."
if type getscript &>/dev/null; then
    echo "✅ SUKCES: ~/.bashrc naprawiony, funkcja getscript działa!"
    echo "   Możesz teraz użyć: getscript test"
else
    echo "⚠️  Funkcja może nie być załadowana. Uruchom: source ~/.bashrc"
    echo "   Lub otwórz nowy terminal."
fi

echo "=== NAPRAWA ZAKOŃCZONA ==="
