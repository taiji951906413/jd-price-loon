# 京东比价 · Loon 远程插件

京东 App 的历史比价插件。看商品时直接显示慢慢买的历史最低价，同时过滤京东广告。

**脚本全部自托管在本仓库 `script/` 目录**，不依赖 GitHub raw，也不依赖原作者 CDN 的存活。

---

## 安装

Loon → 配置 → 插件 → 右上角 `+` → 粘贴 → 保存：

```
https://gcore.jsdelivr.net/gh/taiji951906413/jd-price-loon@main/jingdong.plugin
```

`gcore` 拉不动就换镜像，内容一样：

```
https://fastly.jsdelivr.net/gh/taiji951906413/jd-price-loon@main/jingdong.plugin
https://testingcf.jsdelivr.net/gh/taiji951906413/jd-price-loon@main/jingdong.plugin
```

> 不要用 `raw.githubusercontent.com`，国内网络层直接不可达（TLS 握手就失败）。

## 前置条件

- iPhone / iPad，已装 Loon
- **已安装并信任 Loon 的 MITM 证书** —— 插件的 `[Script]` 依赖它，不装证书脚本不执行
- 慢慢买 App（首次取 CK 用，之后可以不常开）
- 仅支持**京东标准模式**，极速版不适用

---

## 首次使用：取慢慢买 CK

插件的比价功能靠慢慢买接口，需要先抓一次 CK。

1. 确认 Loon 的「重写」「MitM」都开着，证书已信任
2. 确认插件里 `[京东比价]获取慢慢买Cookie` 开关是**开**的（默认开）
3. **上滑杀掉慢慢买 App 进程**（后台挂着不会重新发请求，这步最容易漏）
4. 重新打开慢慢买 App → 点底部【我的】
5. 等 1～3 秒，弹出 `京东比价 - 获取ck成功🎉` 就成功了

脚本会把整段请求体存进 Loon 持久化存储的 `manmanbuy_val` 键，另外单独存 `c_mmbDevId` 到 `慢慢买CK` 键。

### 没弹窗？

看 Loon 日志（配置 → 日志），搜 `manmanbuy`：

- **有记录** → MITM 通了，问题在脚本下载，检查上面的插件地址能不能拉到
- **没记录** → MITM 没拦到，依次查：证书是否真的「已信任」（只装不信任等于没装）、
  App 是否真的发了请求（先断网进一次【我的】再联网）、是否走了 HTTP/3

App 走 QUIC 的话 Loon 拦不到，把插件 `[Rule]` 里注释的那条解开即可：

```
AND, ((DOMAIN, apapia-sqk-weblogic.manmanbuy.com), (PROTOCOL, UDP)), REJECT-DROP
```

---

## 四种显示模式

插件 `[Argument]` 里可以开关，任选其一：

| 开关 | 效果 |
|---|---|
| 详情通知版 | 点进商品详情页，弹窗显示历史最低价 |
| 详情嵌入版（表格图） | 详情页内嵌表格图（默认开） |
| 详情嵌入版（折线图） | 详情页内嵌折线图 |
| 标题嵌入版 | 点商品标题直接看嵌入式价格 |

---

## 关于慢慢买接口的两个实测结论

自己按脚本算法直接打过接口，记下来省得以后再踩：

1. **`c_mmbDevId` 不做设备绑定校验**。随便生成一个 UUID 当 devId，第一次请求就返回
   `code:2000` 并正常拿到 shareId/sign。也就是说 CK 里真正有用的只有这一个字段，
   它本质就是个本地生成的 UUID。
2. **但紧接着连发几次就变 `7700 当前访问人数过多，需登录验证后继续使用`**，两分钟没恢复。
   是频率/IP 级风控，不是设备合法性校验。**光有 CK 也不能无限刷**，查太勤一样被挡。

签名算法（供调试参考）：`token = MD5(URLEncode(KEY + 排序大写拼接 + KEY)).toUpperCase()`，
`KEY = 3E41D1331F5DDAFCD0A38FE2D52FF66F`。

另外脚本用了 `Promise.withResolvers()`，需要 **iOS 17.4+**。日志里出现
`Promise.withResolvers is not a function` 就是系统太老。

---

## 文件说明

| 文件 | 说明 |
|---|---|
| `jingdong.plugin` | 插件本体 |
| `script/jingdong_ad.js` | 京东去广告（源：RuCu6） |
| `script/replace-body.js` | 极速版广告文案替换（源：mieqq） |
| `script/jd_price_mw418.js` | 取 CK + 详情通知版比价（源：mw418） |
| `script/jd_price_wf021325.js` | 标题嵌入版 + 表格图比价（源：wf021325） |
| `script/JingDong.price.js` | 折线图版比价 |

## 致谢

原作者：RuCu6、Maasea、MuTu（githubdulong）、苍井灰灰（wf021325）、mw418、可莉、mieqq。
本仓库只做脚本自托管和地址修复，逻辑未改动。
