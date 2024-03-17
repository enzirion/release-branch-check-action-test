# release-branch-check-action-test
git-flowなリリースをしていて、revertとcherry-pickを組み合わせたときに特定のファイル差分がmainブランチ上で消えてしまう問題を検出するgithub actionを検証します。
少しだけ状況違うけど同じような問題について、以下の記事で解説されています
https://www.isoroot.jp/blog/2502/

# test_mainブランチで消える手順の再現手順
## ブランチ準備
```
git checkout main
git checkout -b test_main
git checkout -b develop
```

## 差分作成
```
git checkout develop
touch dummy
git add .
git commit -m "差分"
DIFF_COMMIT=$(git show --format='%h' --no-patch)
```

## 差分を含んだrelease作成
```
git checkout -b release1
```

## 差分をrevertしてc-pick
```
git checkout develop
git revert ${DIFF_COMMIT}
REVERT_COMMIT=$(git show --format='%h' --no-patch)
git checkout release1
git cherry-pick ${REVERT_COMMIT}
```

## release1をtest_mainにマージ（1回目のリリース）
```
git checkout test_main
git merge --no-edit release1
```

## developでrevert-revertしておく
```
git checkout develop
git revert ${REVERT_COMMIT}
```

## 2回目のリリースブランチを作る
```
git checkout develop
git checkout -b release2
```
このあとrelease2をmainにマージすると、あるはずのdummyファイルがmain上に存在しないことになる


# リセット
```
git checkout master;git branch -D test_main;git branch -D develop;git branch -D release1;git branch -D release2;
```