# User documentation

This project contains IT4Innovations user documentation source.

## Development

### Install

```console
$ sudo apt install libpython-dev
$ virtualenv venv
$ source venv/bin/activate
$ pip install -r requirements.txt
```

### Package upgrade with pip

```console
$ pip list -o
$ pip install --upgrade package
$ pip freeze | sed '/pkg-resources==/d' > requirements.txt
```

## Environments

* [https://docs.it4i.cz - master branch](https://docs.it4i.cz - master branch)
* [https://docs.it4i.cz/devel/$BRANCH_NAME](https://docs.it4i.cz/devel/$BRANCH_NAME) - maps the branches, available only with VPN access

## URLs

* [http://facelessuser.github.io/pymdown-extensions/](http://facelessuser.github.io/pymdown-extensions/)
* [http://squidfunk.github.io/mkdocs-material/](http://squidfunk.github.io/mkdocs-material/)

```
fair-share
InfiniBand
RedHat
CentOS
Mellanox
```

## Mathematical Formulae

### Formulas are made with:

* [https://facelessuser.github.io/pymdown-extensions/extensions/arithmatex/](https://facelessuser.github.io/pymdown-extensions/extensions/arithmatex/)
* [https://www.mathjax.org/](https://www.mathjax.org/)

You can add formula to page like this:

```
$$
MAX\_FAIRSHARE * ( 1 - \frac{usage_{Project}}{usage_{Total}} )
$$
```

To enable the MathJX on page you need to enable it by adding line ```---8<--- "mathjax.md"``` at the end of file.
