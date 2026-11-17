# 关于这个复刻（Fork）

本仓库是 [saya-ch/dsh-mobile](https://github.com/saya-ch/dsh-mobile) 的复刻，遵循上游的 **Apache-2.0** 许可证。
所有代码与文档版权归上游作者所有，这里只保留一份可以自己改、自己验证的副本。

> DSH Mobile 是让手机安全连接电脑 DSH 的社区插件（Android App + PWA）。
> 上手请看上游的 [README](README.md)，本文件只记录「这份复刻做过什么、怎么验证」。

---

## 本地验证记录

验证时间：2026-11-17　环境：linux arm64 / Node 24.22.1 / npm 11.19.1

| 步骤 | 命令 | 结果 |
|---|---|---|
| 依赖 | `npm ci` | 190 个包，通过 |
| 类型检查 | `npm run lint` | 通过（`tsc --noEmit` 无输出） |
| 测试 | `npx vitest run` | **3507 通过 / 29 跳过 / 0 失败**（206 个文件） |

首次跑测试时是 37 项失败，追下来全部是**环境与平台**原因，不是代码缺陷。两个坑记在下面。

---

## 两个环境坑

### 1. `NODE_OPTIONS` 会让 35 个测试误报

如果父进程带着 `--max-old-space-size` 启动，扩展 worker 会拒绝启动：

```
MobileExtensionError: worker heap limits require launching DSH without --max-old-space-size
```

连带的 35 个 `extension-worker*` 测试全部失败，看起来像功能坏了，其实是环境变量。

**解法**：跑测试时清掉它。

```sh
env -u NODE_OPTIONS npx vitest run
```

### 2. iroh 只发布了 x64 产物

`src/iroh-manifest.ts` 里 `IROH_NATIVE_TARGETS` 只有 `win32-x64` 和 `linux-x64`。
在 arm64（比如手机、Apple Silicon、树莓派）上，iroh 组件安装会在下载之前就以
`iroh_platform_unsupported` 失败——这是正确行为，但上游那条测试写死了「下载后大小不匹配」，
于是在非 x64 平台必然失败。

本复刻把这条测试改成**平台感知**：x64 上仍然验证下载失败路径，其它平台验证「明确拒绝」。
改动只在 `tests/plugin.test.ts`，不涉及任何运行时代码。

---

## 这份复刻改了什么

只有一处：`tests/plugin.test.ts` 的 iroh 组件安装用例（平台感知）。
其余文件与上游 `main` 完全一致。

这样改的好处是：不与上游冲突（测试文件改动小），同步时不会打架。

---

## 同步上游更新

仓库首页有 **Sync fork** 按钮，或命令行：

```sh
git remote add upstream https://github.com/saya-ch/dsh-mobile.git   # 只需一次
git fetch upstream
git merge upstream/main
```

如果上游也把这条测试改成了平台感知，合并时可能有一处小冲突，保留任意一侧即可（语义相同）。

---

## 怎么用

**只是想把手机连上电脑**：不用管这个复刻，按上游 README 装官方版最省事。

```sh
npm install dsh-mobile
```

Android App 在上游 Releases 里下载 APK。

**想改代码**：就在这个复刻里改。改完记得跑一遍：

```sh
env -u NODE_OPTIONS npx vitest run
npm run lint
```

---

## 许可与署名

- 上游仓库：<https://github.com/saya-ch/dsh-mobile>
- 许可证：[Apache-2.0](LICENSE)
- 第三方声明：[THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)、[FUNNEL_THIRD_PARTY_LICENSES.txt](FUNNEL_THIRD_PARTY_LICENSES.txt)

按 Apache-2.0 要求，分发时请保留 `LICENSE`、第三方声明与版权信息，不要把这个复刻说成原创项目。
