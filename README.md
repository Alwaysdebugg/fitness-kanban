# 每日健身看板 · GitHub Pages 部署

用 GitHub Actions 自动把 `index.html` 部署成一个网页，每次推送代码后自动更新。

## 文件结构

```
.
├── index.html                     # 健身看板网页（打开即用）
├── README.md                      # 本说明
└── .github/
    └── workflows/
        └── deploy.yml             # GitHub Actions 部署流程
```

---

## 方式 A：全程网页操作（最简单，无需命令行）

1. 打开 https://github.com/new 新建仓库
   - 仓库名任意，例如 `fitness-kanban`
   - 选择 **Public**（免费账号的私有仓库不支持 Pages）
   - 不用勾选任何初始化选项，点 **Create repository**

2. 上传文件
   - 在新仓库页点 **Add file → Upload files**，把 `index.html` 拖进去
   - 再点 **Add file → Create new file**，文件名处输入：
     `.github/workflows/deploy.yml`
     然后把本项目里 `deploy.yml` 的内容整段粘贴进去
   - 分别 **Commit** 保存

3. 开启 Pages
   - 进入仓库 **Settings → Pages**
   - **Build and deployment → Source** 选择 **GitHub Actions**

4. 触发部署
   - 到 **Actions** 标签页，若没自动运行，点开工作流选 **Run workflow** 手动触发
   - 等状态变绿（约 1 分钟）

5. 打开网站
   - 部署成功后，**Settings → Pages** 顶部会显示网址：
     `https://<你的用户名>.github.io/<仓库名>/`

---

## 方式 B：命令行 (git)

在本文件夹内执行（把 `<你的用户名>` 和 `<仓库名>` 换成自己的）：

```bash
git init
git add .
git commit -m "Add fitness kanban"
git branch -M main
git remote add origin https://github.com/<你的用户名>/<仓库名>.git
git push -u origin main
```

然后同样到 **Settings → Pages → Source** 选 **GitHub Actions**（只需设置一次）。

---

## 以后如何更新

改完 `index.html` 后，重新上传 / 或 `git push`，Actions 会自动重新部署，几十秒后网站即更新。

## 常见问题

- **网页打不开 / 404**：确认根目录有 `index.html`，且 Settings → Pages 的 Source 是 **GitHub Actions**。
- **Actions 报权限错误**：`deploy.yml` 已包含 `pages: write` 与 `id-token: write` 权限；若仍报错，检查 Settings → Actions → General → Workflow permissions 是否允许。
- **想用自己的域名**：Settings → Pages → Custom domain 里绑定即可。
- **打卡记录不见了**：记录保存在浏览器本地（localStorage），换浏览器或清缓存会重置，这与是否部署无关。
