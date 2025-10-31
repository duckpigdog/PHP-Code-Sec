# PHP-Code-Sec 靶场（离线 | 支持 PHPStudy 一键部署）

一个覆盖常见 PHP 代码审计知识点的本地靶场。参考 DVWA / Pikachu 的组织方式，含安装向导与通关教程。

![](1.png)
![](2.png)
![](3.png)

## 功能概览

- 安装向导：生成数据库与配置（MySQL / PDO）
- 认证模块：登录 / 注册 / 注销（示范弱口令与 MD5 存储）
- 漏洞模块（部分）：
  - SQL 注入（数字型 / 字符型）
  - XSS（反射型 / 存储型 / DOM 型）
  - CSRF（邮箱修改）
  - 文件上传绕过（双扩展等）
  - 文件包含（LFI/RFI）
  - 路径遍历
  - 命令注入（Windows ping）
  - SSRF（内网探测示例）
  - 反序列化 / PHAR 元数据利用
  - 开放重定向 / 邮件头注入 / Host Header 注入
  - 弱加密（MD5 / rand）
  - 逻辑漏洞（价格参数篡改示例）
  - 会话固定（不 regenerate）
  - RCE（eval 代码执行）
  - JWT 弱校验（none 算法 / 未验签）
  - IDOR 越权访问
  - CORS 误配置（反射 Origin + 允许凭据）
  - HPP 参数污染（多值参数）
  - 任意文件写入（路径遍历写入）
  - phpinfo 敏感信息泄露
  - 点击劫持（未设防）

  扩展模块（新增）：
  - 变量覆盖（extract）
  - XPath 注入
  - 正则 ReDoS（灾难性回溯）
  - PHP 流包装器信息泄露（php://filter）
  - LDAP 注入（过滤器拼接）
  - Web 缓存投毒与欺骗（缓存键/Vary 错误）
  - 二次解析（双重解码）

## 环境要求

- Windows + PHPStudy（推荐）或本机 PHP 7.4+/8.x + MySQL 5.7+/8.0+
- 浏览器即可，无需外网

## 一键部署（PHPStudy）

1. 将本项目文件夹拷贝到 PHPStudy 的 `WWW` 目录，例如：`C:\phpStudy\WWW\PHP-Code-Sec`
2. 启动 PHPStudy 并确保 `Apache/Nginx + PHP + MySQL` 正常运行
3. 访问 `http://localhost/PHP-Code-Sec/setup/install.php`
4. 在安装页面填入 MySQL 信息（主机、端口、用户名、密码），数据库名可用默认 `phpsec_lab`
5. 点击安装，成功后自动生成 `config/config.inc.php` 并初始化表与样例数据
6. 返回首页开始练习：`http://localhost/PHP-Code-Sec/`

> 提示：PHPStudy 常见默认账户可能为 `root` / 空密码 或 `root` / `root`/`123456`，请以实际环境为准。

## 手动运行（本机 PHP）

- 在项目根目录运行：

```
php -S 127.0.0.1:3000 -t .
```

- 然后访问 `http://127.0.0.1:3000/setup/install.php` 完成安装；之后访问首页 `http://127.0.0.1:3000/`

## 目录结构

```
PHP-Code-Sec/
├─ index.php                 # 首页与模块导航
├─ login.php / register.php / logout.php / profile.php
├─ setup/install.php         # 安装向导
├─ core/                     # 初始化与数据库封装
├─ config/                   # 安装生成配置
├─ modules/                  # 漏洞模块
├─ pages/                    # 文件包含用到的演示页
├─ data/                     # 路径遍历演示文件
├─ uploads/                  # 文件上传目录
├─ assets/css/style.css      # 炫酷暗色样式
└─ writeup.md                # 通关教程（详细解法与payload）
└─ summary.md               # 模块源代码索引（快速定位路径）
```

## 重要说明

- 该靶场为教育演示用途，代码刻意存在大量安全问题，请勿在生产环境部署。
- 建议使用隔离环境（如虚拟机 / 容器 / 本地测试机）。
- 学习完成后请及时删除或隔离该项目，避免被误用。

## 常见问题

- 连接失败：检查 MySQL 端口、账号密码、PDO 扩展是否开启。
- 页面 500：查看 `setup/install.php` 是否已成功生成 `config/config.inc.php`。
- RFI/PHAR 演示：部分功能需要开启或利用特定 PHP 配置，详见 `writeup.md`。

## 许可证

本项目仅用于安全学习与审计实操示范。未经授权，禁止用于非法用途。
