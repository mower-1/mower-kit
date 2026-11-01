# 更新检测 · 服务端配置说明

客户端已内置检测逻辑（启动时检查，12 小时内不重复请求，仅在发现新版本时弹窗）。
本目录就是需要放到 GitHub Pages 上的内容。

---

## 一、部署步骤

1. 新建一个公开仓库，例如 `printershare-zh`；
2. 把本目录的 `update.json` 提交到仓库根目录（或 `docs/` 目录）；
3. 仓库 **Settings → Pages**，Source 选择分支 `main`、目录 `/ (root)`，保存；
4. 等待 1～2 分钟，访问确认可打开：
   `https://<你的用户名>.github.io/printershare-zh/update.json`

如果 Pages 打不开，可以用 jsDelivr 作为替代地址（同样读这个仓库文件）：

```
https://cdn.jsdelivr.net/gh/<你的用户名>/printershare-zh@main/update.json
```

---

## 二、修改客户端地址

客户端地址写在 `apk_src/smali/com/dynamixsoftware/printershare/zx.smali`，共两处，改成你的真实地址后重新打包：

- `URL_JSON` 常量（仅作说明用）
- `b()` 方法内的 `const-string v1, "https://..."` ← **这处才是实际请求地址**

未替换前默认值是 `https://YOURNAME.github.io/printershare-zh/update.json`，
域名不存在时请求会失败并静默退出，不影响其它功能。

---

## 三、发版流程

1. 打包新 APK，上传到 GitHub Releases 或其它下载地址，拿到直链；
2. 修改 `update.json`：

| 字段 | 说明 |
|---|---|
| `build` | **必须严格递增的整数**，客户端靠它判断新旧 |
| `version` | 展示给用户的版本名，可写 `12.25.1-zh17` |
| `url` | APK 下载直链 |
| `notes` | 更新说明，支持 `\n` 换行 |

3. 提交，等 Pages 刷新（约 1 分钟）；
4. 用旧版打开应用验证弹窗。

---

## 四、版本号规则

客户端当前 `BUILD = 16`（`zx.smali` 中的 `const/16 v1, 0x10`）。

- 远端 `build` **大于** 16 → 提示更新；
- 等于或小于 16 → 视为已是最新，不提示；
- 用户点过"跳过此版本"后，同一 `build` 不再提示（记录在 SharedPreferences 的 `skip` 键）。

因此**每次发版都要把 `build` 加 1**，同时同步修改客户端里的 `BUILD` 常量，
否则新版本发出去后，用户装了新版仍会被提示"有新版本"。

---

## 五、限制

- 只提示，不强制更新；
- 不校验安装包签名或哈希，如需防替换，可在 json 里加 `sha256` 字段，客户端下载后比对（当前未实现）；
- GitHub Pages 与 jsDelivr 在国内的可达性不稳定，如需稳定可换自建域名或国内 CDN，只要返回同样的 JSON 即可。
