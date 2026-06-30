# 17b-i18n

App 远程国际化仓库，包含两部分：

1. **UI 文案语言包**（`i18n/`）：界面上的按钮、提示等短文案。
2. **静态内容**（`content/`）：规则、FAQ、公司介绍、隐私政策、退款政策等长文档。

中文（`zh-CN`）和英文（`en-US`）两者都内置在 App 包内，其余语言放在这里，通过 jsDelivr CDN 在运行时加载。**新增语言无需修改 App 代码、无需重新打包上架。**

## 目录结构

```
17b-i18n/
├── manifest.json          # 语言清单：决定 App 显示哪些语言
├── i18n/                  # UI 文案语言包
│   ├── vi-VN.json         # 越南语
│   ├── id-ID.json         # 印尼语
│   └── <code>.json
└── content/               # 静态内容（与 i18n 同名）
    ├── vi-VN.json         # 越南语：rules/rulesPaid/faq/company/privacy/refundPolicy
    ├── id-ID.json         # 印尼语
    └── <code>.json
```

`content/<code>.json` 是一个对象，包含 6 个段：

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

App 端通过以下地址读取（`@main` 为浮动版本，推送后自动生效）：

- 清单：`https://cdn.jsdelivr.net/gh/lwf457071117/17b-i18n@main/manifest.json`
- UI 文案：`https://cdn.jsdelivr.net/gh/lwf457071117/17b-i18n@main/i18n/<code>.json`
- 静态内容：`https://cdn.jsdelivr.net/gh/lwf457071117/17b-i18n@main/content/<code>.json`

> 静态内容与 UI 文案共用 `manifest.json` 里同一个 `version`，改内容时一并 +1 即可刷新缓存。

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

## 新增一种语言（例：泰语 th-TH）

1. 翻译并新增两个文件（key/结构都以英文为模板，只翻译值）：
   - `i18n/th-TH.json`：UI 文案（参考 `i18n/en-US.json` 或现有语言）。
   - `content/th-TH.json`：静态内容（6 个段，参考 `content/vi-VN.json`）。
2. 在 `manifest.json` 的 `languages` 数组里加一行：

   ```json
   { "code": "th-TH", "label": "ภาษาไทย", "version": "1", "uniLocale": "en", "bucket": "en", "fallback": "en-US" }
   ```

3. 推送：

   ```bash
   git add .
   git commit -m "Add th-TH language pack"
   git push
   ```

完成。App 无需改动、无需打包，用户下次进入即可在语言列表看到并选用。

## 注意事项

- **新增语言**：是全新文件路径，jsDelivr 无旧缓存，即时生效。
- **修改已有语言文案**：jsDelivr 边缘缓存最长约 12 小时。想立即生效，把该语言的 `version` +1，必要时到 https://www.jsdelivr.com/tools/purge 手动刷新。
- **中文 / 英文**：内置在 App 内，不在本仓库；修改它们仍需改 App 源码并打包。
- **小程序端**：需在微信后台把 `cdn.jsdelivr.net` 加入 request 合法域名（配置一次即可）。
- 翻译建议上线前找母语者校对。
