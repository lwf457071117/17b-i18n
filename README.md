# 17b-i18n

App 远程国际化仓库，包含两部分：

1. **UI 文案语言包**（`i18n/`）：界面上的按钮、提示等短文案。
2. **静态内容**（`content/`）：规则、FAQ、公司介绍、隐私政策、退款政策等长文档。

中文（`zh-CN`）和英文（`en-US`）两者都内置在 App 包内，其余语言放在这里，通过 jsDelivr CDN 在运行时加载。**新增语言无需修改 App 代码、无需重新打包上架。**

## 目录结构

```
17b-i18n/
├── manifest.json              # 语言清单：决定 App 显示哪些语言 + 各语言版本号
├── i18n/                      # UI 文案语言包（文件名带版本号）
│   ├── vi-VN.2.json           # 越南语 v2
│   ├── id-ID.1.json           # 印尼语 v1
│   └── <code>.<version>.json
└── content/                   # 静态内容（与 i18n 同名规则）
    ├── vi-VN.2.json           # 越南语 v2：rules/rulesPaid/faq/company/privacy/refundPolicy
    ├── id-ID.1.json           # 印尼语 v1
    └── <code>.<version>.json
```

> **版本号放在文件名里**（如 `vi-VN.2.json`），不是放在 `?v=` 查询参数。
> 原因：jsDelivr 会忽略 query 参数，且 `@main` 边缘缓存最长约 12 小时；
> 用「带版本的不可变文件名」可以在改版后**立即生效**，无需等待或手动刷新缓存。
> 客户端会读取 `manifest.json` 里每个语言的 `version`，拼出对应的带版本文件名去拉取。

`content/<code>.<version>.json` 是一个对象，包含 6 个段：

```json
{
  "rules": [],         // 免费版规则（数组）
  "rulesPaid": [],     // 付费版规则（数组）
  "faq": [],           // 常见问题（数组）
  "company": {},       // 公司介绍（对象）
  "privacy": {},       // 隐私政策（对象）
  "refundPolicy": []   // 退款政策（数组）
}
```

> 结构必须与英文模板（App 内 `src/assets/json/*.en.json`）一致，只翻译值。

## CDN 地址

App 端通过以下地址读取：

- 清单：`https://cdn.jsdelivr.net/gh/lwf457071117/17b-i18n@main/manifest.json`
- UI 文案：`https://cdn.jsdelivr.net/gh/lwf457071117/17b-i18n@main/i18n/<code>.<version>.json`
- 静态内容：`https://cdn.jsdelivr.net/gh/lwf457071117/17b-i18n@main/content/<code>.<version>.json`

> 同一语言的 UI 文案与静态内容共用 `manifest.json` 里该语言的 `version`。
> 改版时：写一份新文件名（版本 +1）+ 把 `manifest.json` 的 `version` 改成同一个值。
> `manifest.json` 是唯一可能命中 jsDelivr 缓存的入口，推送后需手动刷新它一次（见下方）。

## manifest.json 字段说明

```json
{
  "version": "2026-06-30",
  "languages": [
    {
      "code": "vi-VN",        // 语言代码（与文件名一致）
      "label": "Tiếng Việt",  // 语言选择列表里显示的名字
      "version": "1",         // 该语言包版本号，改文案就 +1（用于客户端刷新缓存）
      "uniLocale": "vi",      // 传给 uni.setLocale 的值；uni 不支持就填 en
      "bucket": "en",         // 静态资源分桶；没有专属资源就填 en
      "fallback": "en-US"     // 缺词时回退到的语言
    }
  ]
}
```

## 新增一种语言（例：泰语 th-TH，版本从 1 开始）

1. 翻译并新增两个文件（key/结构都以英文为模板，只翻译值），文件名带版本号：
   - `i18n/th-TH.1.json`：UI 文案（参考现有 `i18n/vi-VN.2.json`）。
   - `content/th-TH.1.json`：静态内容（6 个段，参考 `content/vi-VN.2.json`）。
2. 在 `manifest.json` 的 `languages` 数组里加一行（`version` 与文件名版本一致）：

   ```json
   { "code": "th-TH", "label": "ภาษาไทย", "version": "1", "uniLocale": "en", "bucket": "en", "fallback": "en-US" }
   ```

3. 推送并刷新清单缓存：

   ```bash
   git add .
   git commit -m "Add th-TH language pack"
   git push
   curl "https://purge.jsdelivr.net/gh/lwf457071117/17b-i18n@main/manifest.json"
   ```

完成。App 无需改动、无需打包，用户下次进入即可在语言列表看到并选用。

## 修改已有语言（改文案/改内容）

1. 把该语言的 `version` +1（如 vi-VN 从 2 → 3）。
2. 新增对应的新文件名：`i18n/vi-VN.3.json` 和/或 `content/vi-VN.3.json`（按需，只改了内容就只更新 content）。
   - 旧版本文件可保留也可删除，客户端只会按 manifest 里的 version 拉新文件名。
3. 把 `manifest.json` 里该语言的 `version` 改成 3。
4. 推送并刷新清单缓存：

   ```bash
   git add .
   git commit -m "Update vi-VN to v3"
   git push
   curl "https://purge.jsdelivr.net/gh/lwf457071117/17b-i18n@main/manifest.json"
   ```

带版本号的文件名是全新路径，jsDelivr 立即返回最新内容；只需刷新 `manifest.json` 这一个入口。

## 注意事项

- **为什么版本号放文件名而不是 `?v=`**：jsDelivr 会忽略 query 参数，`?v=2` 与不带参数视为同一文件，无法用来刷新；且 `@main` 边缘缓存最长约 12 小时。带版本的文件名是不可变路径，改版即时生效。
- **manifest.json 是唯一入口**：它会被缓存，所以每次推送后用 `purge.jsdelivr.net` 刷新一次。
- **中文 / 英文**：内置在 App 内，不在本仓库；修改它们仍需改 App 源码并打包。
- **小程序端**：需在微信后台把 `cdn.jsdelivr.net` 加入 request 合法域名（配置一次即可）。
- 翻译建议上线前找母语者校对。
