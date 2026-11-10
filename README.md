# MZT 桌面自动化助手 · 公开更新通道

这个仓库只做一件事：给 **MZT 桌面自动化助手** 的桌面程序提供**免配置自动更新**。
源码放在私有仓库 `mb5958/mzt-desktop-assistant`，这里只放版本清单和绿色版安装包，
因此分发给别的电脑时，对方不需要任何令牌或配置。

当前版本：**v1.4.5**

<!-- DOWNLOAD:START -->

## 国内下载（推荐，v1.4.5）

完整绿色版安装包：**https://www.jianguoyun.com/p/Dfuf1b4Q7uHaDhijp6sGIAA**

- 版本 1.4.5，109.2 MB，sha256 `7eb192529c79d38de6a727841e0b1f6249d9a98136af32ade0c60b9d9d7d506c`
- 国内直连，绕开 GitHub 大文件；链接绑定文件路径，每次发版覆盖同名文件，**链接固定不变、永远指向最新版**
- 新电脑首次安装用这个；已装过程序的电脑会自动下载约 2.5 MB 的补丁包升级，无需手工下载
- GitHub Release 备用页：https://github.com/mb5958/mzt-desktop-assistant-updates/releases/tag/v1.4.5

<!-- DOWNLOAD:END -->

## 客户端怎么工作

1. **默认优先**读本仓库根目录的 `latest.json`（raw 地址，不受 GitHub API 每小时 60 次的匿名限流）
2. 清单读不到时才退回 `GET /repos/mb5958/mzt-desktop-assistant-updates/releases/latest`
3. **日常升级只下 `patches/patch-vX.Y.Z.zip`（约 2.5 MB）**：里面只有 `resources/` 目录，
   走 raw 直连，秒级完成，也不用替换 188 MB 的 exe（不会遇到 exe 占用问题）
4. 补丁下载失败才按 `candidates` 顺序退回完整包 / API 资产端点；
   每个候选自带 `sha256`，流式下载 + 断点续传 + 校验
5. 由独立升级脚本等程序退出后覆盖程序文件，失败自动回滚，然后自动重启

任务配置在 `%APPDATA%\desktop-automation-assistant`，升级只覆盖程序文件，不会碰到它。

清单地址可在「更新设置 → 清单地址」里改，也可以指向内网服务器 / 对象存储，完全绕开 GitHub：

```
https://raw.githubusercontent.com/mb5958/mzt-desktop-assistant-updates/main/latest.json
```

## 发一个新版本（维护者）

1. 在源码仓库改 `desktop/package.json` 的 `version`、写 `CHANGELOG.md`，打包出完整 zip
2. 在本仓库新建 tag `vX.Y.Z` 的 Release，把完整 zip 作为资产上传（换新电脑时用）
3. 把新包的 `resources` 目录压成 `patches/patch-vX.Y.Z.zip` 放进本仓库：
   ```bash
   cd win-unpacked && 7za a -tzip -mx=9 patch-vX.Y.Z.zip resources
   ```
   **压缩包里必须是 `resources/` 这一层**，不要多套一层目录
4. 更新根目录 `latest.json`：把 `url` / `sha256` / `size` 指向补丁包，
   同时把完整包的地址、大小、sha256 放进 `candidates` 作为兜底
5. 各电脑启动后自动检查，或点顶栏「⬆ 检查更新」立即升级

> 只有当 Electron 运行时本身升级（exe / dll 变化）时，才必须走完整包；
> 平时只改界面、服务端逻辑、asar，补丁包就够了。

## latest.json 字段

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `version` | 是 | 版本号，如 `1.4.0`（不带 v 也行） |
| `tag` | 否 | Release tag，如 `v1.4.0` |
| `name` | 否 | 版本标题 |
| `notes` | 否 | 更新说明，显示在升级弹窗里 |
| `publishedAt` | 否 | ISO 8601 发布时间 |
| `releaseUrl` | 否 | 「查看发布页」按钮的链接 |
| `url` | 是 | **首选**安装包地址（https），一般填补丁包的 raw 地址 |
| `mode` | 否 | `patch`（只含 resources）或 `full`（完整包） |
| `size` / `sha256` | 否 | 首选包的大小与校验值，强烈建议填 |
| `asset` | 否 | `{name,size,sha256,mode,url}`，可覆盖上面的资产信息 |
| `candidates` | 否 | 备选下载列表，按顺序重试。每项 `{url,name,size,sha256,mode}`，各自带自己的校验值 |

## 注意

- 本仓库**必须保持 public**，否则分发版匿名读不到更新
- zip 根目录直接是 `DesktopAssistant.exe`、`resources/` 等文件，不要多套一层文件夹
- 不要在这里放任何令牌；客户端读私有仓库用的令牌只存在用户本机
- 历史版本可定期清理，只保留 `latest.json` 指向的那个即可
