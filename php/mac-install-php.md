# Mac Install Php

```bash
xcode-select --install

brew install automake autoconf curl pcre bison re2c mhash libtool icu4c gettext jpeg openssl libxml2 mcrypt gmp libevent `libiconv` `bzip2` `zlib`
brew link icu4c
brew link --force openssl
brew link --force libxml2

curl -L -O https://github.com/phpbrew/phpbrew/raw/master/phpbrew
chmod +x phpbrew

# Move phpbrew to somewhere can be found by your $PATH
sudo mv phpbrew /usr/local/bin/phpbrew

phpbrew init

接着在 .bashrc 或 .zshrc 文件增加如下行：
[[ -e ~/.phpbrew/bashrc ]] && source ~/.phpbrew/bashrc

phpbrew lookup-prefix homebrew

phpbrew --debug install --stdout 7.1 +default \
    +iconv="$(brew --prefix libiconv)" \
    +bz2="$(brew --prefix bzip2)" \
    +zlib="$(brew --prefix zlib)"
```

