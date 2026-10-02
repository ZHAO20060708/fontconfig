# Eric 的 Fontconfig 配置

面向 Linux 中文桌面的个人字体配置。默认英文用 Inter，中文黑体用更纱，宋体用 Noto Serif；生僻字按黑体、宋体分别回退到遍黑体和字雲。

配置文件：[fonts.conf](fonts.conf) · [直接下载](https://raw.githubusercontent.com/ZHAO20060708/fontconfig/main/fonts.conf)

## 默认字体与规则

| 场景 | 字体优先顺序 |
| --- | --- |
| 无衬线 / `system-ui` | Inter → Sarasa Gothic SC → Noto Sans CJK → Plangothic → Jigmo → Noto Color Emoji |
| 衬线 | Noto Serif → Noto Serif CJK → Jigmo → Plangothic → Noto Color Emoji |
| 等宽 | Sarasa Term SC → Sarasa Term TC / J → Plangothic → Jigmo → Noto Color Emoji |
| `LXGW WenKai GB` 的粗体请求 | 改用 `LXGW ZhenKai GB`，字重重置为 Regular |

- 简体中文字体优先。这套顺序也会影响其他 CJK 语言请求。
- Arial、Liberation Sans、微软雅黑、Segoe UI、PingFang SC 映射到 `sans-serif`；宋体映射到 `serif`；Liberation Mono 映射到 `monospace`。
- 通用字体回退使用 `binding="strong"`，避免更纱被 Fontconfig 的字体分类匹配挤到后面。具体字体名排在通用字体名前时，仍保留优先级。
- 启用抗锯齿，关闭 hinting，子像素顺序设为 RGB。显示器若使用其他子像素排列，请自行修改 `rgba`。
- 全局排除 Nimbus Sans，保留原配置针对 GitHub 的处理。

## Arch Linux：一条命令安装字体

前提：已经安装 `yay`、`pkexec`（来自 `polkit`），并在有 Polkit 认证代理的桌面会话中运行。使用普通用户执行；`--sudo pkexec` 指定图形认证，`--sudoloop=false` 关闭后台认证循环。命令会保留正常的安装和 AUR 构建确认。

普通 Arch 不需要额外添加 archlinuxcn。下面用 `yay` 安装仓库和 AUR 字体，再从臻楷官方发布页下载 GB 字体，最后刷新缓存：

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

如果已配置 **archlinuxcn**，可以全部交给 `yay`：

```bash
yay --sudo pkexec --sudoflags '' --sudoloop=false -S --needed \
  fontconfig inter-font ttf-sarasa-gothic \
  noto-fonts noto-fonts-cjk noto-fonts-emoji \
  ttf-plangothic ttf-jigmo ttf-lxgw-wenkai-gb ttf-lxgw-zhenkai && fc-cache -f
```

以上命令安装字体；下面的步骤安装本仓库配置。

### 包名与来源

包名和来源核对日期：2026-10-02。

| 包名 | 来源 | 用途 |
| --- | --- | --- |
| [`inter-font`](https://archlinux.org/packages/extra/any/inter-font/) | Arch Extra | 英文界面字体 |
| [`ttf-sarasa-gothic`](https://archlinux.org/packages/extra/any/ttf-sarasa-gothic/) | Arch Extra | 更纱 Gothic、Term 及多地区字体 |
| [`noto-fonts`](https://archlinux.org/packages/extra/any/noto-fonts/) | Arch Extra | Noto Serif 与其他文字回退 |
| [`noto-fonts-cjk`](https://archlinux.org/packages/extra/any/noto-fonts-cjk/) | Arch Extra | Noto Sans / Serif CJK 多地区字体 |
| [`noto-fonts-emoji`](https://archlinux.org/packages/extra/any/noto-fonts-emoji/) | Arch Extra | Noto Color Emoji |
| [`ttf-jigmo`](https://archlinux.org/packages/extra/any/ttf-jigmo/) | Arch Extra | Jigmo、Jigmo2、Jigmo3 |
| [`ttf-plangothic`](https://aur.archlinux.org/packages/ttf-plangothic) | AUR；archlinuxcn 也提供 | 遍黑体 P1、P2 |
| [`ttf-lxgw-wenkai-gb`](https://aur.archlinux.org/packages/ttf-lxgw-wenkai-gb) | AUR | 霞鹜文楷 GB |
| [`ttf-lxgw-zhenkai`](https://github.com/archlinuxcn/repo/tree/master/archlinuxcn/ttf-lxgw-zhenkai) | archlinuxcn；本次核对时 AUR 无此包 | 霞鹜臻楷 GB |

文楷和臻楷只用于文楷粗体替换规则，不是默认桌面字体；不使用文楷时可以跳过这两项。

## 手动下载字体

下载已经构建好的 `.ttf`、`.otf`、`.ttc`，不要把 GitHub 的源码压缩包当成字体安装包。

| 项目 | 项目网址 | 下载入口 | 需要的字体 / 下载说明 |
| --- | --- | --- | --- |
| Inter | [rsms/inter](https://github.com/rsms/inter) | [Releases](https://github.com/rsms/inter/releases) | 下载 Inter 字体压缩包，安装桌面字体；配置匹配 `Inter` |
| 更纱黑体 | [be5invis/Sarasa-Gothic](https://github.com/be5invis/Sarasa-Gothic) | [Releases](https://github.com/be5invis/Sarasa-Gothic/releases) | 推荐完整 TTC 包；至少包括 Gothic SC、Term SC；完整保留回退还需 Term TC、Term J |
| Noto Serif | [notofonts/latin-greek-cyrillic](https://github.com/notofonts/latin-greek-cyrillic) | [Releases](https://github.com/notofonts/latin-greek-cyrillic/releases) | 选择 `NotoSerif-*` 发布中的字体包；该仓库最新发布也可能是 Noto Sans |
| Noto CJK | [notofonts/noto-cjk](https://github.com/notofonts/noto-cjk) | [Sans 下载说明](https://github.com/notofonts/noto-cjk/blob/main/Sans/README.md) / [Serif 下载说明](https://github.com/notofonts/noto-cjk/blob/main/Serif/README.md) / [Releases](https://github.com/notofonts/noto-cjk/releases) | Sans 和 Serif 都要装；推荐多地区 TTC，覆盖 SC、TC、JP、KR。使用该项目的 CJK 命名字体 |
| 遍黑体 Plangothic | [Plangothic_Project](https://github.com/Fitzgerald-Porthmouth-Koenigsegg/Plangothic_Project) | [Releases](https://github.com/Fitzgerald-Porthmouth-Koenigsegg/Plangothic_Project/releases) | 安装 `PlangothicP1-Regular.ttf` 和 `PlangothicP2-Regular.ttf`，或包含二者的 Static 包 |
| 字雲 Jigmo | [官方主页与下载](https://kamichikoichi.github.io/jigmo/) | [官方主页内的 ZIP 下载](https://kamichikoichi.github.io/jigmo/) | 安装压缩包中的 Jigmo、Jigmo2、Jigmo3 三个字体 |
| Noto Color Emoji | [googlefonts/noto-emoji](https://github.com/googlefonts/noto-emoji) | [NotoColorEmoji.ttf（v2.051）](https://github.com/googlefonts/noto-emoji/blob/v2.051/fonts/NotoColorEmoji.ttf) / [直接下载](https://raw.githubusercontent.com/googlefonts/noto-emoji/v2.051/fonts/NotoColorEmoji.ttf) | 提供明确包含 `NotoColorEmoji.ttf` 的版本链接；配置不使用黑白 `Noto Emoji` 或 3D Emoji |
| 霞鹜文楷 GB | [lxgw/LxgwWenKaiGB](https://github.com/lxgw/LxgwWenKaiGB) | [Releases](https://github.com/lxgw/LxgwWenKaiGB/releases) | 安装 `LXGWWenKaiGB-*.ttf`；普通文楷、TC、Mono 版本不能代替这条 GB 字体规则 |
| 霞鹜臻楷 | [lxgw/LxgwZhenKai](https://github.com/lxgw/LxgwZhenKai) | [Releases](https://github.com/lxgw/LxgwZhenKai/releases) / [GB Regular 直接下载](https://github.com/lxgw/LxgwZhenKai/releases/latest/download/LXGWZhenKaiGB-Regular.ttf) | 安装 `LXGWZhenKaiGB-Regular.ttf` |

创建用户字体目录，把解压后的字体文件放进去，再刷新缓存：

```bash
mkdir -p "${XDG_DATA_HOME:-$HOME/.local/share}/fonts"
# 将下载的 .ttf / .otf / .ttc 文件复制到上面的目录，也可以按项目分子目录。
fc-cache -f
```

## 安装 fonts.conf

先安装字体，再执行下面的命令。已有配置会备份为带时间戳的文件。新文件先下载到临时位置，成功后才替换当前配置。

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

也可以下载仓库中的 `fonts.conf`，手动备份后放到 `~/.config/fontconfig/fonts.conf`；自定义 XDG 路径时使用 `$XDG_CONFIG_HOME/fontconfig/fonts.conf`。

重新打开浏览器、编辑器等应用，使它们读取新配置。KDE 中显式设置的界面字体仍由 KDE 字体设置控制；Fontconfig 负责匹配与回退。

## 检查实际匹配

命令统一使用 `:family=...`，避免直接输入带连字符的字体名称时产生解析歧义。

```bash
fc-conflist
fc-match ':family=sans-serif:lang=zh-cn:charset=0041'     # Inter，字符 A
fc-match ':family=sans-serif:lang=zh-cn:charset=4e2d'     # Sarasa Gothic SC，字符 中
fc-match ':family=system-ui:lang=zh-cn:charset=4e2d'      # Sarasa Gothic SC
fc-match ':family=serif:lang=zh-cn:charset=4e2d'          # Noto Serif CJK SC
fc-match ':family=monospace:lang=zh-cn:charset=4e2d'      # Sarasa Term SC
fc-match ':family=sans-serif:charset=20000'              # Plangothic P1
fc-match ':family=serif:charset=20000'                   # Jigmo 系列
fc-match ':family=sans-serif:charset=31350'              # Plangothic P2
fc-match ':family=serif:charset=31350'                   # Jigmo 系列
fc-match ':family=sans-serif:charset=1f600'              # Noto Color Emoji，字符 😀
fc-match ':family=LXGW WenKai GB:weight=bold'            # LXGW ZhenKai GB，Regular
```

在 Fontconfig 2.18.3 上验证过默认字体、指定字体、不同语言标签、生僻字和粗体规则。以上是 Fontconfig 的匹配结果；应用自行加载的网页字体及自身渲染策略也会影响最终显示。

参考：[Fontconfig 官方文档](https://fontconfig.pages.freedesktop.org/fontconfig/fontconfig-user.html) · [yay 官方手册](https://github.com/Jguer/yay/blob/next/doc/yay.8)。字体项目、包名及下载入口见上表。
