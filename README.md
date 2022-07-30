# Aaron's Dev Dairy

This is a repo to develop and deploy my personal blog <[Aaron's Dev Dairy](https://doubleslash.me)>

[![Node.js Version](https://img.shields.io/badge/node-%3E=12.0-success.svg?style=flat-square&logo=Node.js&longCache=true)](https://hexo.io)

## 🚀 Installation & Start


```sh
$ npm install hexo-cli -g
$ npm install
$ hexo s
```

## 🎨 New Post

```sh
$ hexo new post "<Post Name>"
```

## 🔧 Deploy to github-pages
To implement an auto deployment on `Github Pages`, add your repo info in `_config.yml`:
```yml
deploy:
  type: git
  repo: <git repo url>
  branch: <branch name>
```
Run command `npm run publish` to trigger a deployment to github pages.

## ☁️ Deploy to remote server
Build up and update to master branch, will automatically deploy on aws
https://doubleslash.me
```
npm run build
```
