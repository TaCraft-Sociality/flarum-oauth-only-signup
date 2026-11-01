# OAuth Only Signup

关闭「邮箱 + 密码」注册，只允许通过第三方账号（QQ / GitHub / 微软 / TcsSkin 等）创建新账号；**老用户仍可正常使用邮箱 + 密码登录**。

## 行为

| 场景 | 结果 |
| --- | --- |
| 登录框「注册」入口 | 隐藏（`allowSignUp` 前端属性被强制为 `false`） |
| 第三方登录后创建新账号 | 允许（携带 `RegistrationToken`，由 OAuth 回调生成） |
| `POST /register` 或 `POST /api/users` 提交邮箱 + 密码 | 拒绝，返回 `本站已关闭邮箱注册，请使用 QQ、GitHub、微软或 TcsSkin 登录。` |
| 老用户邮箱 + 密码登录 | 不受影响 |
| 管理员在后台创建用户 | 允许（`actor->isAdmin()`） |

## 实现

- `extend.php` 用 `Extend\ApiResource(ForumResource::class)` 把 `allowSignUp` 字段的 `get` 覆盖成 `false`。这里只改**前端拿到的属性**，`allow_sign_up` 设置本身不变，所以 `POST /users` 端点依然对游客开放 —— 否则第三方注册也会被一起挡掉。
- `Saving` 事件里校验新建账号的请求体：只有带合法 `RegistrationToken` 的请求（即 OAuth 回调发起的注册）才放行，其余一律抛 `ValidationException`。

## 注意

- 后台「权限 → 允许注册」开关仍会影响 `POST /users` 端点本身：**不要把它关掉**，关掉会连带禁止第三方注册。
- 本扩展不提供设置项，装上即生效；如需恢复邮箱注册，禁用本扩展即可。
- 未附带 JS 构建产物：界面变化完全通过上面的 `allowSignUp` 属性实现，无需编译。
