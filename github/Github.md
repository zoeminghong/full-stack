# Github

分享有意思的项目和使用 GitHub 的小技巧。

## 小技巧

### 快速选择 gitignore

GitHub 官方开源了一个项目，这个项目为开发者编写好特定的 .gitignore 规则，做成了模板。开发者只需选择好自己的项目类别，将文件内容复制粘贴放到自己项目里面就可以用了。

https://github.com/github/gitignore

这是由 Uber 一名工程师 joeblau 所开发的 .gitignore 文件快速生成工具，开发者只需要在网站上搜索当前正在使用的操作系统、IDE、编程语言，它便会自动生成一个特定的 .gitignore 配置文件。

https://www.gitignore.io/

如果你不想用网站进行搜索，还可以安装下他的命令行工具。

安装完成后，就可以使用 gi 命令来快速生成 .gitignore 配置文件啦。

### 巧用 GitHub 项目模板

这个功能可以将以往创建过的仓库标记成模板（template），这样在你下一次创建仓库的时候，就可以使用这个模板功能，快速生成具有和原仓库一样的目录与文件内容。

每个模板仓库在 URL 末端带上 /generate 后，还可以将模板仓库通过链接分享给其他人，其它人在打开链接之后，便可以快速通过这个模板来创建新仓库。

GitHub 官方还称，未来会在 repo、issue 和 pull requests 中扩展更多模板类型，以避免开发者做一些重复性的工作，将更多精力专注于项目研发上，可以说非常值得期待了。

## 项目

### 快速生成 README 文档

https://github.com/kefranabg/readme-md-generator

支持根据 `package.json` 帮助生成 README 文件，但不支持其他语言。

