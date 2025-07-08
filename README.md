# Personal website

This repository contains my personal website, which is hosted on [healeyj.com](https://healeyj.com/). 

The template was created by Olivier Bourgeois, which is available on [GitHub](https://github.com/bourgeoisor/bourgeoisor.github.io).

## How to run

These instructions assume you have Python 3.12+ installed. Fetch the dependencies:

```sh
pip3 install -r requirements.txt
```

Generate HTML files from Markdown:

```sh
./make.sh
```

The files will be generated into an `out/` directory. You could serve them with a simple HTTP server:

```sh
python3 -m http.server -d out/ 8080
```

And browse to `http://localhost:8080/`.

## How to deploy to Github pages

Use GitHub Actions?
