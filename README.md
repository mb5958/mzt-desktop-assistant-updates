# MZT 桌面自动化助手 · 公开更新通道

这个仓库只做一件事：给 **MZT 桌面自动化助手** 的桌面程序提供**免配置自动更新**。
源码放在私有仓库 `mb5958/mzt-desktop-assistant`，这里只放版本清单和绿色版安装包，
因此分发给别的电脑时，对方不需要任何令牌或配置。

当前版本：**v1.4.0**

## 客户端怎么工作

1. **默认优先**读本仓库根目录的 `latest.json`（raw 地址，不受 GitHub API 每小时 60 次的匿名限流）
2. 清单读不到时才退回 `GET /repos/mb5958/mzt-desktop-assistant-updates/releases/latest`
3. 比对版本；安装包优先走 `url`，失败自动切 `fallbackUrl`（API 资产端点）
4. 流式下载 + 断点续传，用 `sha256` 校验完整性（GitHub 资产的 digest 就是它）
5. 由独立升级脚本等程序退出后覆盖程序文件，失败自动回滚，然后自动重启

任务配置在 `%APPDATA%\desktop-automation-assistant`，升级只覆盖程序文件，不会碰到它。

清单地址可在「更新设置 → 清单地址」里改，也可以指向内网服务器 / 对象存储，完全绕开 GitHub：

```
https://raw.githubusercontent.com/mb5958/mzt-desktop-assistant-updates/main/latest.json
```

## 发一个新版本（维护者）

1. 在源码仓库改 `desktop/package.json` 的 `version`、写 `CHANGELOG.md`，重新打包出
   `DesktopAssistant-Portable-vX.Y.Z-日期.zip`
2. 在本仓库新建 tag `vX.Y.Z` 的 Release，把 zip 作为资产上传
3. 更新本仓库根目录的 `latest.json`（字段见下）
4. 各电脑启动后自动检查，或点顶栏「⬆ 检查更新」立即升级

## latest.json 字段

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `version` | 是 | 版本号，如 `1.4.0`（不带 v 也行） |
| `tag` | 否 | Release tag，如 `v1.4.0` |
| `name` | 否 | 版本标题 |
| `notes` | 否 | 更新说明，显示在升级弹窗里 |
| `publishedAt` | 否 | ISO 8601 发布时间 |
| `releaseUrl` | 否 | 「查看发布页」按钮的链接 |
| `url` | 是 | 安装包下载地址（必须是 https），一般填 Release 资产的 browser_download_url |
| `fallbackUrl` | 否 | 备用下载地址，一般填 API 资产端点 `/releases/assets/{id}` |
| `size` | 否 | 安装包字节数，用于下载进度 |
| `sha256` | 否 | 安装包 sha256，强烈建议填，客户端会校验 |
| `asset` | 否 | `{"name","size","sha256"}`，可覆盖上面的资产信息 |

## 注意

- 本仓库**必须保持 public**，否则分发版匿名读不到更新
- zip 根目录直接是 `DesktopAssistant.exe`、`resources/` 等文件，不要多套一层文件夹
- 不要在这里放任何令牌；客户端读私有仓库用的令牌只存在用户本机
- 历史版本可定期清理，只保留 `latest.json` 指向的那个即可
