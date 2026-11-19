# Xboard Admin Dist

本仓库存放 Xboard 管理后台的构建产物及后台主题框架。Xboard 主仓库通过 Git 子模块将其挂载到 `public/assets/admin`。当前分支包含 MX、Mundo Connect 1、Mundo RDP 的管理界面。

## 主题目录

| 路径 | 用途 |
| --- | --- |
| `manifest.json` | Vite 入口、样式和分块清单 |
| `assets/`、`locales/` | 管理后台构建产物和翻译 |
| `themes/base.css` | 公共字体、焦点、卡片和动效 |
| `themes/minimal/theme.css` | 简约主题的浅色、深色变量 |
| `index.html` | 构建产物独立入口 |
| `preview.html` | 无后端依赖的主题预览 |

简约主题使用低饱和青绿、浅边框、细阴影和短过渡。系统启用减少动态效果时，公共入场动画和交互过渡会关闭。浅色和深色由现有后台的 `.dark` class 驱动。

## 新建主题

1. 复制 `themes/minimal/theme.css` 到 `themes/<id>/theme.css`，主题 ID 使用小写字母、数字及连字符，且以字母开头。
2. 将选择器中的 `data-admin-theme="minimal"` 改成自己的主题 ID。按需调整 Shadcn 变量及 `--admin-*`。颜色变量使用 HSL 通道，例如 `163 69% 26%`，组件会用 `hsl(var(--primary))` 包装。
3. 在本地起静态服务，例如 `python3 -m http.server 8080`。打开 `/preview.html`，在预览 HTML 中替换主题 ID 及 CSS 路径即可试用新主题。
4. 在实际 Xboard 后台验证登录页、仪表板、列表、弹窗、键盘焦点、手机布局、深色及减少动态效果。额外布局样式限定在自己的 `html[data-admin-theme="<id>"]` 下。避免依赖构建文件的哈希名。
5. 提交主题文件后，在 Xboard 主仓库更新 `public/assets/admin` 子模块指针，重新构建 Docker 镜像。部署后，在后台打开 **后台主题**，选择主题并点击 **应用到所有管理员**。如需设置首次部署的默认主题，可设定 `XBOARD_ADMIN_THEME=<id>`，或设为 `default` 使用原样式。服务器已有共享选择时，以共享选择为准。修改部署配置后重建容器，并刷新已有的 Laravel 配置缓存。

## 从后台上传自定义配色

只调整颜色时，可以直接上传 JSON 色板。上传文件最多 4 KB，`light` 和 `dark` 各需填写至少一个颜色变量。例如保存为 `my-theme.json`：

```json
{
  "id": "my-theme",
  "light": { "primary": "163 69% 26%", "primary-foreground": "0 0% 100%" },
  "dark": { "primary": "153 56% 69%", "primary-foreground": "163 44% 11%" }
}
```

在 **后台主题** 点击 **上传主题** 并选择 JSON 色板。服务器确认主题 ID 唯一、字段与颜色合法后才保存。选中保存的主题，再点击 **应用到所有管理员**。未填写的颜色继承简约主题。完整上传说明见 [Xboard README](https://github.com/Furina-Lee/Xboard/blob/master/README.md#%E7%AE%A1%E7%90%86%E5%90%8E%E5%8F%B0%E4%B8%BB%E9%A2%98)。

## 入口与发布

Xboard 的 `resources/views/admin.blade.php` 读取子模块的 Vite 清单并加载原始 CSS 与 JS，随后从 `/assets/admin/themes/` 加载主题。主题文件不加入 Vite 构建清单，方便独立迭代。独立入口 `index.html` 同样在原 CSS 后加载主题。替换上游构建产物时保留 `themes/`、预览和文档，并重新检查入口的引用。

发布以主仓库记录的子模块提交为准。只更新此仓库的分支时，主仓库指针和镜像不会自动切到新提交。
