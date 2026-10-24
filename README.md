# 部署到 Render.com（5 分钟）

这个目录就是需要推送到 GitHub 的仓库内容：`attendance.war`、`Dockerfile`、`entrypoint.sh`。
War 已经在本地构建并被 Tomcat 9 实测通过（根路径可直接访问）。

> 重要：war 在 Dockerfile 里被复制为 `ROOT.war`，所以应用在域名**根路径**
> `https://<服务名>.onrender.com/` 就能打开。若按实验指导书写成
> `attendance.war`，应用会落在 `/attendance/`，根路径会 404。

## 1. 建 GitHub 仓库并推送

在 GitHub 上新建仓库 `attendance`（不要勾选 Add a README）。仓库地址将是
https://github.com/linhaoli0926/attendance 。然后在本目录执行：

```bat
cd C:\Users\linha\Documents\deepseek-harness\default-workspace\deploy
git init
git add attendance.war Dockerfile entrypoint.sh .gitattributes
git commit -m "Simple Attendance by Seat Id"
git branch -M main
git remote add origin https://github.com/linhaoli0926/attendance.git
git push -u origin main
```

推送时弹出的登录窗口用浏览器授权即可（Git Credential Manager）。

## 2. 创建 Render Web Service

1. 打开 https://render.com ，用 GitHub 账号登录（Sign up with GitHub）。
2. 右上角 **New + → Web Service**，选择刚推的 `attendance` 仓库。
3. 填写：
   - **Name**：`attend` + 你 DID 的后 5 位（例如 DID 后 5 位是 `12345`，就填 `attend12345`）。
     这个名字就是域名：`https://attend12345.onrender.com`
   - **Region**：Singapore（或 Oregon，离你近的）
   - **Runtime**：Docker（会自动识别 Dockerfile）
   - **Branch**：main
   - **Instance Type**：Free
4. 点 **Create Web Service**，等日志出现 `Server startup`、状态变成 **Live**。

## 3. 验收

浏览器打开 `https://attend<你的后5位>.onrender.com/`，应当直接看到
「Student Id / Seat Id」表单（不是 404、不是 Tomcat 欢迎页），然后：

- 提交 `001` / `A1` → 显示 `001:A1 is recorded:`
- 点 Attendee List → 看到 `001 : A1`
- Student Id 留空提交 → 显示 `Student Id can not be blank`

## 4. 常见问题

| 现象 | 原因 / 处理 |
|---|---|
| 根路径 404，但 `/attendance/` 能开 | Dockerfile 里 war 名不是 `ROOT.war` |
| 构建失败 `COPY failed: attendance.war` | war 没提交到仓库根目录 |
| 容器启动后立刻退出 | `entrypoint.sh` 被 Windows 转成了 CRLF；本目录的 `.gitattributes` 已强制 LF，若手改过请检查 |
| 打开很慢（30–60 秒） | 免费实例空闲后会休眠，首次访问要冷启动，属正常 |
| 之前的数据没了 | 记录在内存里，实例重启/休眠回收后清空，属正常 |
