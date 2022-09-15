[![github pages](https://github.com/chienfuchen32/chienfuchen32.github.io/actions/workflows/gh-pages.yaml/badge.svg?branch=main)](https://github.com/chienfuchen32/chienfuchen32.github.io/actions/workflows/gh-pages.yaml)

## local writing
```bash
$ hugo server -D
# open new shell
$ hugo new posts/<POST_NAME>.md
```

## push newest changes and deploy page with GitHub workflow
```bash
$ git add .
$ git push origin main
```
* please check details in `.github/workflows/gh-pages.yaml`
