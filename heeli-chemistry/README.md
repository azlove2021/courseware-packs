# 初中化学互动课件 · 离线版

把 [app.heeli.cc](https://app.heeli.cc/courseware/junior-chemistry)（轻页Lab · 初中化学学科互动网页）复刻成**断网可用**的本地版本。

双击 `index.html` 即可使用：不需要联网、不需要登录、不需要装 Python 或任何运行时。

## 内容

| 课件 | 单元 | 体积 |
|---|---|---|
| 氧气制取 | 九年级上册 · 第二单元 | 1.5 MB |
| 水的电解 | 九年级上册 · 第四单元 | 1.4 MB |
| 自制简易净水器 | 九年级上册 · 第四单元 | 19 MB |
| 化学反应中原子的重新组合 | 九年级上册 · 第五单元 | 2.7 MB |
| 氯化钠溶液配制 | 九年级下册 · 第二单元 | 3.1 MB |
| α 粒子散射实验 | 实验与拓展 · 科学拓展 | 577 KB |

合计 6 个课件 / 110 个文件 / 29 MB。

**范围说明**：原站初中化学共 44 个课件，其中 6 个标记为「免费课件」，其余 38 个需付费解锁。未解锁的课件，服务端 `/api/courseware/<slug>/shell` 返回 403 且不下发课件正文，因此无法离线化——本包收录的就是这 6 个可完整运行的全部课件。

## 目录结构

```
index.html                 总览页（自己写的，零外链）
使用说明.txt               给同事看的三行说明
assets/covers/             6 张封面（课件离线渲染后截图）
site/<slug>/               课件本体
  index.html               入口
  styles.css               样式
  runtime.js               全部逻辑（含内联的图片/几何数据）
  portal-bridge.js         与父页面通信（离线时静默失效，不影响运行）
  media/                   贴图、视频等素材
_tools/                    复刻工具链（见下）
```

## 复刻方法

目标站点是 Next.js 应用，课件正文不在 `app.heeli.cc` 上，而在 `courseware.heeli.cc` 的静态目录里：

```
https://courseware.heeli.cc/heeli-portal/courseware-static/chemistry/<slug>/v-private-<hash>/index.html
```

这个地址由 `GET /api/courseware/<slug>/shell` 下发（需要登录），并且资源本身要求带
`__Secure-heeli_chemistry` cookie。该 cookie **按单个课件授权**：在 A 课件页面拿到的 cookie
只能取 A 课件的资源，拿去取 B 课件会 403。所以流程必须是「每个课件单独取 cookie → 立刻下载」。

工具链：

```bash
python _tools/mirror_courseware.py            # 按清单镜像资源
python _tools/check_refs.py                   # 核对引用闭合（解析课件内的资源映射表）
python _tools/fill_missing.py <slug>          # 补抓漏掉的资源
python _tools/make_offline.py                 # 改造为可 file:// 直开
python _tools/build_index.py                  # 重新生成总览页
```

`_tools/harvest_merged.json` 记录每个课件的课件地址与资源清单，由浏览器端采集（真实打开课件、
点完所有 Tab 与控件，收集 `page.on("request")` 的全部请求）后合并而成。

## 关键发现（踩过的坑）

1. **cookie 按课件授权 + 滑动续期**：每次请求后 cookie 长度都会变（330→334→338→363），
   抓下来的副本几十秒后就失效。必须「取完立即下载」，不能先批量抓 cookie 再统一下载。
2. **`integrity` + `crossorigin` 会让 file:// 直接白屏**：`<link rel="stylesheet" integrity="sha256-…"
   crossorigin="anonymous">` 在 file:// 下 CORS 校验必然失败，`styles.css` 和 `runtime.js`
   全部 `net::ERR_FAILED`，页面只剩背景图。`make_offline.py` 会去掉这两个属性。
3. **`img.crossOrigin="anonymous"` 在 file:// 下同样拦图**：去掉后 canvas 会被标记为「被污染」，
   但课件本身不读像素，实测零报错、读数正常递增。
4. **课件用「逻辑名 → 哈希文件名」映射表**：`{"./assets/favicon.svg":"./media/asset.e55….png"}`。
   直接正则扫 `assets/xxx.webp` 会得到一堆假缺失，必须解析这张表。
5. **风控很敏感**：短时间内连续请求 `/api/courseware/*/shell` 会返回
   `{"code":"account_disabled"}` 并踢下线（账号直接不可用，需重新注册）。
   必须按真人节奏走：一个课件一次页面访问，请求间隔 2 秒以上。

## 验证结果

6 个课件在 `file://` 下逐个用真实浏览器打开、点击主按钮并等待渲染：

| 检查项 | 结果 |
|---|---|
| 资源请求失败 | **0**（110 个文件全部命中） |
| JS 运行时错误 | **0** |
| 控制台报错 | **0** |
| 交互验证 | 水的电解：读数 0 → H₂ 1.6 mL / O₂ 0.8 mL，电源状态切换正常；净水器：过滤进度 49%；α 散射：动画播放中 |

## 已知限制

- 课件内「实验要点」「帮助」等弹层的深层内容只验证了首屏，未穷举每个弹层的所有子页。
- `portal-bridge.js` 原本负责与站点父页面通信（收藏、埋点、全屏请求），离线后这些调用静默失效，
  不影响课件本体运行。
- 原站课件若发版更新，本包不会自动跟进；重新复刻需要重新走一遍 `mirror_courseware.py`。
- 仅收录免费课件；付费课件无服务端正文，无法复刻。

## 版权

课件内容版权归原作者（轻页Lab / heeli.cc）所有。本离线包仅供教学演示与内网学习使用，
请勿用于商业分发或公开传播。
