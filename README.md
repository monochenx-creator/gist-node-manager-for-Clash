# Gist 节点管理工具

一个纯前端的 Clash/Mihomo 订阅配置管理页面。用户可以把配置托管在自己的 GitHub Gist 里，然后在网页中新增、删除、重命名 `vless://` 节点，并把结果写回 Gist。订阅链接保持为 Gist raw 地址。

## 功能

- 读取和更新用户自己的 GitHub Gist 配置文件
- 粘贴 `vless://` 链接新增节点
- 在页面里勾选删除节点
- 给节点重命名，并同步更新策略组引用
- 手动添加网址/域名规则，并选择要走的策略组
- 自动生成 Gist raw 订阅链接，方便复制
- 通过本机 Clash/Mihomo 控制接口测试节点延迟
- 纯静态页面，不需要服务器保存用户 token

## 安全说明

这个工具是纯前端页面。GitHub token 只保存在使用者自己的浏览器 `localStorage` 里，页面不会把 token 发送到你的服务器。

开源或部署时，不要上传自己的真实订阅配置。仓库里的 `.gitignore` 已经忽略了 `Markonly.yaml`，示例配置请使用 `sample-config.yaml`。

## 本地使用

直接打开 `index.html` 即可。

如果浏览器限制本地文件访问，可以在项目目录启动一个本地预览：

```bash
python3 -m http.server 8765 --bind 127.0.0.1
```

然后访问：

```text
http://127.0.0.1:8765/
```

## GitHub Token

1. 打开 GitHub token 页面：
   `https://github.com/settings/tokens`
2. 建议创建 Fine-grained token。
3. 在 Account permissions 里添加 `Gists` 权限。
4. 权限选择 `Read and write`。
5. 生成 token 后复制到页面里的 `GitHub token` 输入框。

## 使用方式

1. 在 GitHub 创建一个 Gist，文件名例如 `config.yaml`。
2. 可以先把 `sample-config.yaml` 内容复制进去作为初始配置。
3. 打开页面，填写 GitHub 用户名、Gist ID、文件名和 token。
4. 点“读取”。
5. 粘贴 `vless://` 链接后点“加入节点”。
6. 添加网址规则时，一行一个网址或域名，选择策略组后点“加入规则”。
7. 需要改名时，在节点卡片里修改名称并点“重命名”。
8. 需要删除时，勾选节点后点“删除勾选”。
9. 确认生成预览没问题后，点“更新 Gist”。
10. 复制页面里的订阅链接，填到 Clash/Mihomo 客户端。

## 网址规则

“添加网址规则”支持输入完整网址或域名：

```text
example.com
https://chat.example.com/path
```

默认会生成 `DOMAIN-SUFFIX,域名,策略组`。也可以切换为：

- `DOMAIN`：完整域名
- `DOMAIN-KEYWORD`：域名关键词

规则会自动插入到最后的 `MATCH` 规则前面，避免被兜底规则提前命中。

## 延迟检测

浏览器不能直接测试 vless 节点，所以页面会调用本机 Clash/Mihomo 的控制接口。

默认控制地址：

```text
http://127.0.0.1:9090
```

如果检测连不上，可以在页面点“补上检测接口”，再更新 Gist 并重启 Clash/Mihomo。它会写入类似配置：

```yaml
external-controller: 127.0.0.1:9090
```

如果设置了 `secret`，也需要在页面里填同一个值。

线上部署后，浏览器可能因为 HTTPS 页面访问本机 HTTP 接口而拦截延迟检测。节点编辑、Gist 读写、订阅链接复制不受影响；延迟检测可以改为本地打开页面使用。

## 部署到 GitHub Pages

1. 新建一个公开仓库，例如 `gist-node-manager`。
2. 上传这些文件：
   - `index.html`
   - `README.md`
   - `sample-config.yaml`
   - `.gitignore`
   - `vendor/js-yaml.min.js`
   - `vendor/lucide.min.js`
3. 进入仓库的 Settings。
4. 找到 Pages。
5. Source 选择 `Deploy from a branch`。
6. Branch 选择 `main`，目录选择 `/root`。
7. 保存后等待 GitHub 生成访问地址。

生成的地址通常类似：

```text
https://你的用户名.github.io/gist-node-manager/
```

## 部署到 Cloudflare Pages

1. 把项目推到 GitHub 仓库。
2. 打开 Cloudflare Pages。
3. 选择 `Connect to Git`。
4. 选择你的仓库。
5. Framework preset 选择 `None`。
6. Build command 留空。
7. Output directory 填 `/` 或留空。
8. 部署完成后访问 Cloudflare 提供的域名。

## 注意

- 不要把自己的真实节点配置提交到公开仓库。
- 不要把 GitHub token 写进代码或 README。
- 节点名不要和策略组名重复，例如 `自动选择`、`🔰 代理`，页面会自动拦截这种冲突。
- 页面重新生成 YAML 时会保留配置结构，但原 YAML 注释会被去掉。
