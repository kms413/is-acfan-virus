# acfun (`com.yyvgoj.vypfmlat`) 静态逆向安全分析

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)
[![Author](https://img.shields.io/badge/author-github%2Fkms413-blue.svg)](https://github.com/kms413)

对一个**自称 "acfun"、实际包名为 `com.yyvgoj.vypfmlat` 的 Android 应用**（版本 1.9.9）的静态逆向安全分析。

本仓库用于验证一条传言——"该 App 使用所谓 system service 进行攻击"——并对其权限、组件、原生层与 Flutter/Dart 业务层做了完整核查。

> 分析对象仅为本地的 apktool 反编译产物与 Dart AOT 反编译产物，**未在真机上运行动态验证**。

---

## 核心结论


| 问题                                             | 结论                                                                       |
| ------------------------------------------------ | -------------------------------------------------------------------------- |
| 利用 system service 攻击 / 控制设备              | ❌ 无证据。"system service" 字样来自 AndroidX WorkManager 与 OAID 厂商绑定 |
| 无障碍 / 设备管理 / 通知监听 / 悬浮窗 / 静默安装 | ❌ 无（无组件、无权限、无代码）                                            |
| 读取短信 / 通讯录 / 通话记录 / 定位              | ❌ 无（连权限都未申请）                                                    |
| 后台偷拍 / 偷录并上传                            | ❌ 无证据（相机/麦克风仅用户主动触发）                                     |
| 采集设备标识与设备指纹并上传                     | ✅**存在**（OAID/AAID/android_id/guid + 数十项指纹）                       |
| 行为埋点批量上报                                 | ✅**存在**（自研遥测系统，带断点续传/指数退避）                            |
| 上传用户选择的图片/视频                          | ✅ 具备该能力（`/file/upload/*`、`/sts/upload*`）                          |
| 动态下发可执行代码                               | ❌ 无（无 DexClassLoader / mmap-rwx / JIT）                                |
| 利用 CVE-2026-0089 提权安装应用                  | ❌ 无证据（Java/原生/Dart 三层均无 PackageInstaller 调用，包内无载荷）     |

**一句话**：这是一个用 Flutter 编写、冒充 AcFun 的擦边内容类 App。它**没有**用敏感权限做"攻击性"的事，但**确实**在采集设备标识、设备指纹与行为埋点并加密上传至自有后端。

完整结论、证据与风险推测见 **[安全分析报告.md](安全分析报告.md)**。

---

## 仓库结构

本仓库为 apktool 反编译产物，主要目录：


| 路径              | 说明                         |
| :---------------- | ---------------------------- |
| `安全分析报告.md` | **本次分析报告（主产出物）** |
| `LICENSE`         | CC BY-NC-SA 4.0 许可协议全文 |

> 注意：Flutter 的业务逻辑**不在** `smali` 里，而在 `libapp.so`。仅分析 `smali` 会漏掉全部 Dart 行为。

---

## 分析方法

1. **Android 层**：`apktool` 反编译后全量阅读 `AndroidManifest.xml` 与 `smali`，核查权限、组件与导出面。
2. **Dart 层**：用 [blutter](https://github.com/worawit/blutter) 反编译 `lib/arm64-v8a/libapp.so`（自行编译 Dart VM 3.10.4 + capstone 5.0.6），得到 2772 个 Dart 汇编单元与字符串/对象池（`pp.txt`、`objs.txt`）。
3. **交叉验证**：对反编译产物做特征字符串与交叉引用检索，定位采集字段、上报端点与加密实现。

工具链：`apktool 2.7.0`、`blutter`（自编译）、`ripgrep`。

---

## 免责声明

- 本仓库仅用于**安全研究、合规审查与公众知情**，不得用于任何非法用途。
- 报告所标注的"无证据"指**在反编译产物中未发现相应实现**，不等于数学意义上的绝对排除；如需确定性结论，须进行动态验证（抓包 + Frida）。
- 文中涉及的第三方应用、商标与域名归各自权利人所有，本分析与其无任何隶属或背书关系。

---

## 许可协议

本作品采用 **[知识共享 署名-非商业性使用-相同方式共享 4.0 国际 (CC BY-NC-SA 4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/)** 许可协议。

**作者：github/kms413**

- **署名** — 必须给出适当署名（`github/kms413`）并标明是否作出修改
- **非商业性使用** — 不得将本作品用于商业目的
- **相同方式共享** — 演绎作品须以相同许可协议分发

详见 [LICENSE](LICENSE)。
