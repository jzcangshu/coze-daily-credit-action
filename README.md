# Coze daily credit action

每天用 Playwright（浏览器自动化）打开 `coze.cn`，通过 Cookie（登录凭证）保持登录；失败时推送 Bark（iOS 推送工具），成功时不推送。

## GitHub Secrets（GitHub 加密变量）

至少配置下面其中一个：

- `COZE_COOKIES_JSON`：推荐。只放 Coze / Volcengine 相关 Cookie 数组，脚本会转换成 Playwright storage state。
- `COZE_STORAGE_STATE_JSON`：也支持，但这个仓库不再建议用它，因为太大，GitHub Secret（GitHub 加密变量）容易超限。

失败推送：

- `BARK_PUSH_URL`：例如 `https://api.day.app/你的BarkKey`。不要把完整 Bark URL 写进代码或提交到 Git。

可选自动更新 Cookie：

- `GH_PAT`：Fine-grained personal access token（细粒度个人访问令牌），需要当前仓库的 Actions secrets 读写权限。配置后，每次成功运行会把刷新后的 Cookie 数组写回 `COZE_COOKIES_JSON`。

## GitHub Variables（GitHub 普通变量）

可选：

- `COZE_TARGET_URL`：默认 `https://www.coze.cn/home`。
- `COZE_CLICK_TEXTS`：逗号分隔的按钮文案，例如 `领取,签到,免费积分,立即领取`。
- `UPLOAD_UPDATED_STORAGE_STATE`：设为 `true` 时，成功运行后会把刷新后的登录状态作为 artifact（构建产物）上传。这里包含 Cookie，不建议长期打开。

## 获取 Cookie / Storage State

推荐本地用 Playwright 登录一次后导出：

```powershell
npm install
npx playwright install chromium
npx playwright codegen https://www.coze.cn/home --save-storage=storage_state.json
```

登录完成后关闭浏览器，把 `artifacts/storage_state.from-profile.json` 里的 Cookie 数组复制到 GitHub Secret `COZE_COOKIES_JSON`。更省事的做法是让工作流后续自动刷新。

如果你只有浏览器扩展导出的 Cookie，也可以把 Cookie JSON 数组复制到 `COZE_COOKIES_JSON`。

## 定时

`.github/workflows/coze-daily.yml` 里的 cron 是 `30 21 * * *`，对应北京时间每天 05:30 左右。真正访问 Coze 前会再随机等待 0 到 180 秒。GitHub Actions（GitHub 自动化流水线）本身也可能会有几分钟延迟。

## 本地测试

```powershell
$env:COZE_STORAGE_STATE_JSON = Get-Content .\storage_state.json -Raw
$env:BARK_PUSH_URL = "https://api.day.app/你的BarkKey"
$env:PLAYWRIGHT_CHANNEL = "chrome"
npm start
```

成功时只输出日志；失败时会推送 Bark，并在 `artifacts/` 里留下截图和 HTML，方便定位。`artifacts/` 可能包含登录页面信息，排查后请自行清理。

## 部署

```powershell
git init
git add .
git commit -m "Add Coze daily credit action"
git branch -M main
git remote add origin https://github.com/你的用户名/你的仓库.git
git push -u origin main
```

然后在 GitHub 仓库设置里添加上面的 Secrets（GitHub 加密变量）和 Variables（GitHub 普通变量）。
