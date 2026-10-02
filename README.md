# Eric 的 Fontconfig 配置

一套开箱即用的 Linux 个人字体配置方案。主要解决 Linux 桌面常见的西文与中文风格割裂、生僻字豆腐块、以及部分中文字体算法假粗体发虚发胖的问题。

### 核心特性
- **清晰现代的界面西文**：默认无衬线西文优先匹配 **Inter**。
- **干脆利落的中文黑体**：中文无衬线与等宽优先匹配 **更纱黑体 (Sarasa Gothic / Term)**。
- **生僻字全覆盖兜底**：黑体回退至 **遍黑体 (Plangothic)**，宋体回退至 **字雲 (Jigmo)**，彻底告别豆腐块。
- **拯救文楷粗体**：请求 `LXGW WenKai GB` 的粗体时，自动平替为 **`LXGW ZhenKai GB` (霞鹜臻楷 Regular)**，告别算法描边糊成一团的“假粗体”。
- **基础渲染调优**：开启抗锯齿，关闭 Hinting，设定 RGB 子像素渲染；全局屏蔽丑陋的 Nimbus Sans。

配置文件：[fonts.conf](fonts.conf) · [直接下载](https://raw.githubusercontent.com/ZHAO20060708/fontconfig/main/fonts.conf)

---

## 字体回退逻辑

| 场景 | 字体匹配顺序 |
| :--- | :--- |
| **无衬线 / `system-ui`** | Inter → 更纱黑体 SC → Noto Sans CJK → 遍黑体 → 字雲 → Noto Color Emoji |
| **衬线 (Serif)** | Noto Serif → Noto Serif CJK → 字雲 → 遍黑体 → Noto Color Emoji |
| **等宽 (Monospace)** | 更纱等宽 SC → 更纱等宽 TC / J → 遍黑体 → 字雲 → Noto Color Emoji |
| **霞鹜文楷粗体** | 命中请求后自动重定向至 **霞鹜臻楷 GB Regular** |

> **实现细节**：
> 1. 通用回退规则声明了 `binding="strong"`，防止更纱黑体被系统自带的全局优先级规则挤到后排。
> 2. 常见系统字体别名（Arial、Segoe UI、微软雅黑、PingFang SC 等）直接重定向至上述体系；宋体映射至 `serif`；Liberation Mono 映射至 `monospace`。
> 3. 渲染配置默认使用 RGB 排列；若你的显示器是 BGR 排列，修改 `fonts.conf` 中的 `rgba` 即可。

---

## 安装指引

### 方案 A：Arch Linux（推荐，一条命令安装）

普通用户身份直接执行（脚本会自动调用 `pkexec` 进行图形提权）。

#### 1. 安装字体

**默认仓库 + AUR（自动从 GitHub 获取臻楷）：**

```bash
yay --sudo pkexec --sudoflags '' --sudoloop=false -S --needed \
  fontconfig curl inter-font ttf-sarasa-gothic \
  noto-fonts noto-fonts-cjk noto-fonts-emoji \
  ttf-plangothic ttf-jigmo ttf-lxgw-wenkai-gb && (
  set -eu
  fontconfig_font_dir="${XDG_DATA_HOME:-$HOME/.local/share}/fonts/LXGW-ZhenKai"
  mkdir -p "$fontconfig_font_dir"
  fontconfig_font_tmp="$(mktemp "$fontconfig_font_dir/.zhenkai.XXXXXX")"
  trap 'rm -f "$fontconfig_font_tmp"' EXIT
  curl --fail --location --retry 3 \
    'https://github.com/lxgw/LxgwZhenKai/releases/latest/download/LXGWZhenKaiGB-Regular.ttf' \
    --output "$fontconfig_font_tmp"
  install -m 644 "$fontconfig_font_tmp" "$fontconfig_font_dir/LXGWZhenKaiGB-Regular.ttf"
  fc-cache -f
)
```

**如果你已启用 `archlinuxcn`（全部走包管理）：**

```bash
yay --sudo pkexec --sudoflags '' --sudoloop=false -S --needed \
  fontconfig inter-font ttf-sarasa-gothic \
  noto-fonts noto-fonts-cjk noto-fonts-emoji \
  ttf-plangothic ttf-jigmo ttf-lxgw-wenkai-gb ttf-lxgw-zhenkai && fc-cache -f
```

*(文楷与臻楷仅供粗体替换规则使用，不需要该规则可自行从包列表中去掉)*

#### 2. 部署配置

自动下载 `fonts.conf` 并部署至用户目录，若存在旧配置会自动备份（附带时间戳后缀）：

```bash
(
  set -eu
  fontconfig_config_dir="${XDG_CONFIG_HOME:-$HOME/.config}/fontconfig"
  mkdir -p "$fontconfig_config_dir"
  fontconfig_config_tmp="$(mktemp "$fontconfig_config_dir/.fonts.conf.XXXXXX")"
  trap 'rm -f "$fontconfig_config_tmp"' EXIT
  curl --fail --location --retry 3 \
    'https://raw.githubusercontent.com/ZHAO20060708/fontconfig/main/fonts.conf' \
    --output "$fontconfig_config_tmp"
  if [ -e "$fontconfig_config_dir/fonts.conf" ]; then
    cp -p "$fontconfig_config_dir/fonts.conf" \
      "$fontconfig_config_dir/fonts.conf.bak-$(date +%Y%m%d-%H%M%S-%N)"
  fi
  chmod 644 "$fontconfig_config_tmp"
  mv "$fontconfig_config_tmp" "$fontconfig_config_dir/fonts.conf"
  fc-cache -f
)
```

---

### 方案 B：手动下载与安装（通用 Linux 发行版）

下载 Releases 中的字体成品文件（`.ttf` / `.otf` / `.ttc`），放进用户字体目录即可：

| 字体 | 项目地址 | 安装建议 |
| :--- | :--- | :--- |
| **Inter** | [rsms/inter](https://github.com/rsms/inter/releases) | 下载解压安装桌面字体包 |
| **更纱黑体** | [be5invis/Sarasa-Gothic](https://github.com/be5invis/Sarasa-Gothic/releases) | 推荐完整 TTC 包；至少包含 Gothic SC 与 Term SC |
| **Noto Serif** | [notofonts/latin-greek-cyrillic](https://github.com/notofonts/latin-greek-cyrillic/releases) | 下载 `NotoSerif-*` 字体发布包 |
| **Noto CJK** | [notofonts/noto-cjk](https://github.com/notofonts/noto-cjk/releases) | Sans 和 Serif 均需安装；推荐包含多地区的 TTC 包 |
| **遍黑体** | [Plangothic_Project](https://github.com/Fitzgerald-Porthmouth-Koenigsegg/Plangothic_Project/releases) | 安装 `PlangothicP1-Regular` 与 `PlangothicP2-Regular` |
| **字雲 Jigmo** | [kamichikoichi/jigmo](https://kamichikoichi.github.io/jigmo/) | 下载 ZIP 并解压包含的 Jigmo 1/2/3 字体 |
| **Noto Color Emoji** | [googlefonts/noto-emoji](https://raw.githubusercontent.com/googlefonts/noto-emoji/v2.051/fonts/NotoColorEmoji.ttf) | 下载彩色 Emoji 文件 `NotoColorEmoji.ttf` |
| **霞鹜文楷 GB** *(可选)* | [lxgw/LxgwWenKaiGB](https://github.com/lxgw/LxgwWenKaiGB/releases) | 安装 `LXGWWenKaiGB-*.ttf` |
| **霞鹜臻楷** *(可选)* | [lxgw/LxgwZhenKai](https://github.com/lxgw/LxgwZhenKai/releases/latest/download/LXGWZhenKaiGB-Regular.ttf) | 获取 `LXGWZhenKaiGB-Regular.ttf` |

解压并放置字体文件到目录后刷新缓存：

```bash
mkdir -p "${XDG_DATA_HOME:-$HOME/.local/share}/fonts"
# 将下载的字体放入上述目录，然后刷新缓存：
fc-cache -f
```

最后将本仓库的 [fonts.conf](fonts.conf) 保存到 `~/.config/fontconfig/fonts.conf`。

> **注意**：
> - 重启浏览器和终端等应用即可生效。
> - KDE Plasma 等桌面若在“系统设置 -> 字体”中强制指定了特定界面字体，界面优先采用该设置；Fontconfig 依然接管网页、文档和字符回退流程。

---

## 规则验证

运行以下测试命令，确认系统命中与回退是否符合预期：

```bash
# 1. 西文与中文匹配
fc-match ':family=sans-serif:lang=zh-cn:charset=0041'     # 命中 Inter（字符 A）
fc-match ':family=sans-serif:lang=zh-cn:charset=4e2d'     # 命中 Sarasa Gothic SC（汉字“中”）
fc-match ':family=system-ui:lang=zh-cn:charset=4e2d'      # 命中 Sarasa Gothic SC
fc-match ':family=serif:lang=zh-cn:charset=4e2d'          # 命中 Noto Serif CJK SC
fc-match ':family=monospace:lang=zh-cn:charset=4e2d'      # 命中 Sarasa Term SC

# 2. 生僻字扩展区回退
fc-match ':family=sans-serif:charset=20000'              # 命中 Plangothic P1
fc-match ':family=serif:charset=20000'                   # 命中 Jigmo 系列
fc-match ':family=sans-serif:charset=31350'              # 命中 Plangothic P2
fc-match ':family=serif:charset=31350'                   # 命中 Jigmo 系列

# 3. Emoji 与文楷粗体重定向
fc-match ':family=sans-serif:charset=1f600'              # 命中 Noto Color Emoji (😀)
fc-match ':family=LXGW WenKai GB:weight=bold'            # 粗体命中 LXGW ZhenKai GB Regular
```

---

## 参考

- [Fontconfig 官方文档](https://fontconfig.pages.freedesktop.org/fontconfig/fontconfig-user.html)
- [yay 手册](https://github.com/Jguer/yay/blob/next/doc/yay.8)
