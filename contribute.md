## 參與貢獻
貢獻者 [名單](https://github.com/yeasy/blockchain_guide/graphs/contributors)。

區塊鏈技術自身仍在快速發展中，生態環境也在蓬勃成長。

本書源碼開源託管在 Github 上，歡迎參與維護：[github.com/yeasy/blockchain_guide](https://github.com/yeasy/blockchain_guide)。

首先，在 GitHub 上 `fork` 到自己的倉庫，如 `docker_user/blockchain_guide`，然後 `clone` 到本地，並設置用戶信息。

```sh
$ git clone git@github.com:docker_user/blockchain_guide.git
$ cd blockchain_guide
$ git config user.name "yourname"
$ git config user.email "your email"
```

更新內容後提交，並推送到自己的倉庫。

```sh
$ #do some change on the content
$ git commit -am "Fix issue #1: change helo to hello"
$ git push
```

最後，在 GitHub 網站上提交 pull request 即可。

另外，建議定期使用專案倉庫內容更新自己倉庫內容。
```sh
$ git remote add upstream https://github.com/yeasy/blockchain_guide
$ git fetch upstream
$ git checkout master
$ git rebase upstream/master
$ git push -f origin master
```

