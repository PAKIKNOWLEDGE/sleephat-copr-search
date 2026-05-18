# copr-search

## 缘起

事情是这样的——想装个 [Sarasa Gothic / 更纱黑体](https://github.com/be5invis/Sarasa-Gothic) 字体，但是想载那个nerdfont修复版的，去 Copr 搜，没搜到。问LLM确实有这个源,它的真实地址:
主页：https://copr.fedorainfracloud.org/coprs/lchh/sarasa-gothic-nerd-fonts/
二、为什么搜不到（3 个原因）
```
    包名带前缀，不是纯 sarasa-gothic-nerd-fonts里面的包名都加了 jonz94- 前缀：
        jonz94-sarasa-gothic-sc-nerd-fonts
        jonz94-sarasa-mono-sc-nerd-fonts
        jonz94-sarasa-fixed-sc-nerd-fonts
        直接搜全名会匹配不到。
```

直接搜纯包名（比如 `sarasa-gothic-sc-nerd-fonts`）就匹配不上了。而 Google 的 `site:` 限定搜索能搜到，但大陆连不上 Google。Bing 和 DuckDuckGo 能打开，索引又不全。
Copr 网页搜索本身也有局限——只搜项目名/描述，不搜包名，搜不全。API v3 有全部项目数据但不提供搜索接口。网页前端还被 Anubis 反爬保护了。
所以写了这个——把 Copr API 的 38,000+ 个项目全部拉下来建本地索引，离线模糊搜索，不依赖任何搜索引擎。搜项目名、owner、描述都能匹配，不用管里面的包名带什么前缀。
实测：

```
$ copr-search sarasa
🔍 找到 2 个匹配「sarasa」的项目:

  📦 lchh/sarasa-gothic-nerd-fonts
     启用: sudo dnf copr enable lchh/sarasa-gothic-nerd-fonts

  📦 regunakyle/sarasa-gothic
     启用: sudo dnf copr enable regunakyle/sarasa-gothic
```

## 安装

```bash
git clone 本项目
cd copr-search
chmod +x copr-search
# 建议放到 PATH 里
ln -s "$PWD/copr-search" ~/.local/bin/
```

需要 Python 3，不需要额外依赖（只用标准库）。

## 用法

### 1️⃣ 拉取项目索引

```bash
copr-search update
```

首次拉取约 1–2 分钟（~38,000 个项目），缓存在 `~/.cache/copr-search/`。

后续再次运行会增量更新并清理已删除的项目。

### 2️⃣ 搜索

```bash
copr-search sarasa
copr-search nerd font
copr-search neovim
```

模糊匹配项目名、owner、描述。搜不到就试试更短的关键词。

## 数据源

- API：`https://copr.fedorainfracloud.org/api_3/project/list`
- 建索引字段：`full_name`、`ownername`、`name`、`description`
- 存储：SQLite（`~/.cache/copr-search/projects.db`）

## 对比其他方案

| 方案 | 国内可用 | 搜全量 | 搜包名 | 模糊匹配 |
|------|:-------:|:-----:|:-----:|:-------:|
| Copr 网页搜索 | ✅ | ❌ 不全 | ✅ | ❌ |
| Google site: | ❌ | ✅ | ✅ | ✅ |
| Bing site: | ✅ | ❌ | ✅ | ✅ |
| **copr-search** | ✅ | ✅ | ❌(注) | ✅ |

> 注：API 只返回项目级别的数据，不包含项目内的包名列表。搜项目名效果很好，但如果要找某个项目里的具体包名还是得上网页看。

## License

MIT
