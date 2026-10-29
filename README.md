# 互动课件离线合集

两套可直接运行的互动课件，断网可用：不联网、不登录、不装任何软件。

**在线直接看：** https://azlove2021.github.io/courseware-packs/

## 两套合集

| 合集 | 课件数 | 体积 | 学科 |
|---|---|---|---|
| [初中化学互动课件](heeli-chemistry/index.html) | 6 | 27.7 MB | 化学 |
| [小白教育互动课件](xb-edu/index.html) | 53 | 52.6 MB | 物理 24 / 化学 12 / 地理 12 / 数学 5 |

## 怎么用

**在线看**：点上面的链接，直接进总目录。

**整包下载**：本页右上角 `Code → Download ZIP`，解压后双击根目录 `index.html`。

**单个课件**：小白教育的每个课件都是单文件 HTML，`xb-edu/html/` 里挑一个文件发给别人，双击就能看。

## 两套的形态不一样

| | 初中化学（轻页Lab） | 小白教育 |
|---|---|---|
| 结构 | 每课件是一个目录（HTML + CSS + JS + media） | 每课件是**单个 HTML**，全部内联 |
| 能否单独拷 | 要连目录一起拷 | 单个文件即可 |
| 打开方式 | 双击课程目录里的 `index.html` | 双击即可 |
| 3D | 3D 实验装置 + 视频素材 | 3D / 2D 视学科而定 |

## 关于范围

两套都是从站点上能公开拿到的部分整理而来，**不是完整的付费内容**：

- 初中化学：原站 44 个课件，只有 6 个免费，其余 38 个需付费解锁，服务端不下发正文
- 小白教育：原站 550 个课件，只有 53 个可免费下载，其余 497 个为付费资源

## 复刻方法（简述）

详细的踩坑记录在两个子目录的 README 里，这里只记要点。

**初中化学**（Next.js 站点）
- 课件正文在 `courseware.heeli.cc`，地址由 `/api/courseware/<slug>/shell` 下发
- 鉴权 cookie **按单个课件授权且滑动续期**，几十秒就失效 → 必须「取一个课件的 cookie，立刻下它的资源」
- 课件的 `styles.css` / `runtime.js` 带 `integrity` + `crossorigin`，`file://` 下会被 CORS 直接拦死，
  页面上只剩一张背景图 → 离线包必须去掉这两个属性
- 图片的 `crossOrigin="anonymous"` 同样要清掉（canvas 会被标脏，但课件不读像素）

**小白教育**（静态站 + JSON API）
- `GET /api/catalog` 匿名可取，550 个产品的元数据直接当索引
- `POST /api/downloads/prepare` 签发一次性下载地址，**5 分钟过期**
- POST 必须同时带 `Origin` 和 `X-Requested-With: studio`，少一个就是 403「请求来源不匹配」

## 目录结构

```
index.html              合集首页
heeli-chemistry/        初中化学 6 个课件
  index.html            该套的总目录
  site/<slug>/          各课件（HTML + 资源）
  assets/covers/        封面
xb-edu/                 小白教育 53 个课件
  index.html            该套的总目录（按学科分组）
  html/<id>.html        53 个单文件课件
  assets/covers/        封面
```

## 验证情况

- 初中化学：6 个课件在 `file://` 与 HTTP 下分别验证，资源失败 0 / JS 错误 0；水的电解读数 0 → H₂ 1.6 mL、O₂ 0.8 mL
- 小白教育：53 个课件逐个真实浏览器打开，导航失败 0 / 资源失败 0 / JS 错误 0，共 87 个 canvas

## 版权

课件内容版权归原作者所有（轻页Lab / heeli.cc、小白教育 / xbjiaoyu.com）。
本项目仅做离线化整理与打包，**未修改课件内容**，仅供教学演示与学习使用，请勿用于商业分发。
如原作者认为不妥，请联系删除。
