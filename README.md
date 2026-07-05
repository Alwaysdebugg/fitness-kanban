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
- **打卡记录不见了**：默认记录保存在浏览器本地（localStorage），换浏览器或清缓存会重置。想要**跨设备同步**请按下面「云同步」一节配置 Supabase。

---

## 云同步（可选）：Supabase + Google 登录

配置后，用 Google 账号登录即可在手机、电脑之间自动同步打卡记录。**未配置时页面照常以「本地模式」运行，不受影响。**

原理：打卡数据仍先写本地（离线可用），登录后按 `updated_at` 时间戳与云端做「最后写入优先」的双向同步。`anon key` 是公开可提交的，安全性由数据库 Row Level Security（RLS）保证——每个用户只能读写自己那一行。

### 1. 创建 Supabase 项目
1. 打开 https://supabase.com → 注册 / 登录 → **New project**（免费）
2. 项目建好后进 **Settings → API**，记下两项：
   - **Project URL**（形如 `https://xxxx.supabase.co`）
   - **anon public** key

### 2. 建表 + 开启 RLS
进 **SQL Editor**，粘贴并运行：

```sql
create table if not exists public.workout_state (
  user_id    uuid primary key references auth.users(id) on delete cascade,
  state      jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now()
);
alter table public.workout_state enable row level security;
create policy "own row select" on public.workout_state
  for select using (auth.uid() = user_id);
create policy "own row insert" on public.workout_state
  for insert with check (auth.uid() = user_id);
create policy "own row update" on public.workout_state
  for update using (auth.uid() = user_id) with check (auth.uid() = user_id);
```

### 3. 配置 Google 登录
1. 到 [Google Cloud Console](https://console.cloud.google.com/) → 新建项目 → **APIs & Services → Credentials → Create Credentials → OAuth client ID**（类型选 **Web application**）
2. 在 **Authorized redirect URIs** 填入 Supabase 的回调地址：
   `https://<你的项目>.supabase.co/auth/v1/callback`
3. 拿到 **Client ID / Client Secret**，填到 Supabase → **Authentication → Providers → Google**，启用并保存
4. Supabase → **Authentication → URL Configuration**：
   - **Site URL** 填：`https://<用户名>.github.io/<仓库名>/`
   - **Redirect URLs** 里也加上同一个网址

### 4. 把密钥填进网页
编辑 `index.html`，找到顶部的 `SUPABASE CONFIG`，替换两行：

```js
const SUPABASE_URL = "https://xxxx.supabase.co";
const SUPABASE_ANON_KEY = "eyJhbGci...";   // anon public key
```

保存后 `git push`，Actions 自动重新部署。刷新网页顶部会出现「用 Google 登录同步」按钮。

### 常见问题
- **点登录后跳回来没登上**：多半是第 3 步的 Site URL / Redirect URLs 没填对，或没带结尾斜杠。
- **登录后记录没同步**：确认第 2 步 SQL 跑成功、RLS 策略已建。
- **本地模式（灰点）**：说明 `index.html` 里还留着 `YOUR_...` 占位符，未配置云同步。
