# budo-plugins

小布兜（Budo）远程扩展目录仓库：`catalog.json` 供应用「插件市场」拉取；zip 包经域名白名单 + SHA-256 + ad-hoc 签名校验后安装。格式规范见 budo 主仓 `docs/design/des/013-extensions.md`。

## 仓库结构

```
catalog.json                        # 目录（version=1，extensions[] 条目）
releases/<id>-<version>/<id>.zip    # 扩展包（平铺：manifest.json + <id>.dylib 在 zip 根）
```

## 发布一个扩展（清单）

1. `swiftc -emit-library -parse-as-library` 编译扩展 dylib（多 Swift 文件 + bridge.c 混编时必须显式 `-emit-library -parse-as-library`，否则链接报 `_main` 未定义），`codesign --force --sign -` ad-hoc 签名。
2. zip **平铺打包**（manifest.json 与 dylib 在根，不能带顶层目录——宿主按解压根找 manifest）。
3. 计算 zip 的 SHA-256（小写十六进制）写入 catalog；`url` 指向本仓库 raw/releases 路径（宿主域名白名单仅允许 `github.com` / `raw.githubusercontent.com`）。
4. push 后等待约 5 分钟（GitHub raw CDN `cache-control: max-age=300`），应用内点「刷新」即可见。

## 注意

- 扩展 id 强制 `x-` 前缀；manifest 与 dylib 内嵌 JSON 必须同源。
- 宿主图标取 manifest 的 `icon` 字段（SF Symbol 名），请为每个扩展声明合适的图标。
- 版本升级：catalog 条目 version 提高即可，宿主仅接受更高版本覆盖安装。
