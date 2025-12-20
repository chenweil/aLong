# rename-nav-brand

## 背景
首页导航栏品牌文字由主题 `themes/gokarna/layouts/partials/header.html` 的 `{{ .Site.Title }}` 渲染。

## 修改
- 修改 `config/_default/hugo.toml`
- `title` 从 `aLong Blog` 改为 `aLong`

## 预期结果
- 顶部导航 `class="nav-brand"` 显示为 `aLong`（去掉 `Blog`）
