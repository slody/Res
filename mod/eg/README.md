# Egern 小红书模块说明

## 文件

- `Xiaohongshu.egern.yaml`：最小生产修复版。
  - 仅把转换后丢失 `tiny-gif` 响应体的 `/system_config/watermark` 条目改成 `url_rewrites` + `reject-img`。
  - 其余 `rewrites`、脚本、MITM、规则保持 Egern 转换稿原样。
  - 脚本仍使用上游 `fmz200/wool_scripts` 的 `xiaohongshu.js`，没有本地 JS 改动。

## 根因

Egern 能识别转换稿里的 `rewrites` + `actions` + `respond`，这不是失败原因。

真正的差异在评论区水印接口：

- Surge 原规则：`^https?://ci.(xiaohongshu|rnote).com/system_config/watermark data-type=tiny-gif status-code=200`
- Egern 转换后：只剩 `status: 200`，1×1 GIF 响应体丢失。

评论区保存会读取这个接口；返回空 200 时保存失败。

## 验证结果

- `v0`（原始转换稿 + 上游 JS）：失败。
- `v4`（上游 JS + 评论列表 `max_size`）：失败。
- `v3`（只修复 watermark，上游 JS、无 `max_size`）：成功。
- `v5`（JS + `max_size` + watermark）：成功。

结论：watermark 响应修复是必要且充分的；`max_size` 和 JS 修复不是这次失败的必要条件。

## 可选加固

如果后续遇到超大评论列表或评论视频缓存不命中的场景，可参考 `xhs-variants` 分支的 `v5-js-max-watermark`：

- 评论列表脚本加 `max_size: 20971520`
- 使用变体分支里的最小修复 JS

测试矩阵和逐项结果见 `xhs-variants` 分支：
<https://github.com/slody/Res/tree/xhs-variants/mod/eg/xiaohongshu-variants>
