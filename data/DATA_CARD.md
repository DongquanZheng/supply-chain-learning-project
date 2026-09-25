# Development data

本目录保存当前研究使用的开发数据，不包含尚未揭示的未来测试标签。

- `panel_source.csv`：可读的国家—周面板及建模目标
- `panel.npz`：模型使用的紧凑数组，包含 `x`、`y`、`weeks`、`countries` 和 `weights`
- `weights_source.csv`：由 WITS 贸易数据构造的国家间权重
- `target_geometry.npz`：目标与时间几何辅助数组
- `manifest.json`：数据构建清单与指纹

主要公共来源为 PortWatch、GDELT、WITS 和日历信息。使用或再发布时应同时遵守各上游数据源的条款，并在研究成果中引用相应来源。

