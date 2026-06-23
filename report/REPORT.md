# MODNet 卡通绿幕抠图方案对比实验

生成日期：2026-06-17

## 结论速览

本实验比较 4 种生图阶段方案，并固定使用 MODNet 做 alpha 估计。绿色方案使用线上同源 MODNet_v10 后处理；蓝幕方案使用同构 blue-key 后处理，避免把蓝幕硬塞进只认绿色的规则里。

- **不建议把白边作为默认方案。** 白边组 holes 从当前绿幕的 5.78% 降到 3.31%，green spill 也更低，但 `white_rim_pct` 从 3.41% 暴涨到 52.05%，`halo_dark` 也从 18.85 升到 55.56。视觉上就是稳定的贴纸白边。
- **深色外轮廓是最适合默认路径的候选。** 它的 `halo_dark` 最低（12.94），白边残留很低（0.89%），视觉上像角色线稿的一部分；缺点是 holes 没有显著优于当前方案。
- **蓝幕值得继续做专门后处理实验。** 蓝幕组 holes 最低（2.06%），halo 也低于当前绿幕，但 `blue_spill` 高（17.25），且遇到蓝色衣物时会逼迫生图改色。它更像“可选高质量模式”，不是无脑替换。

重要 caveat：这轮是 production-style prompt 实验，每个方案都重新生图，所以它评估的是“生图约束 + MODNet 抠图”的综合效果，不是同一张像素图上的纯 matting 对照。n=8、无重复，结论用于筛方向；下一轮若要做统计决策，建议每组 2 repeats 或增加同图合成控制组。

![overview](/Users/august/chromakey-matting-exp-2026-06-17/report/overview.jpg)

## 汇总指标（均值，越低通常越好；largest_cc_ratio 越高越好）

| 方案 | n | halo_dark | holes_pct | green_spill | blue_spill | white_rim_pct | dark_rim_pct | soft_band_pct | largest_cc_ratio |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Current Green | 8 | 18.85 | 5.776 | 0.91 | 1.16 | 3.41 | 57.60 | 2.15 | 0.9942 |
| White Outline Green | 8 | 55.56 | 3.308 | 0.37 | 0.06 | 52.05 | 4.18 | 2.01 | 0.9837 |
| Dark Contour Green | 8 | 12.94 | 5.668 | 1.42 | 0.49 | 0.89 | 69.61 | 2.41 | 0.9946 |
| Blue Screen | 8 | 14.27 | 2.062 | 0.02 | 17.25 | 0.50 | 53.20 | 2.05 | 0.9794 |

## 方案说明

- Current Green：当前纯 #00FF00 绿幕契约，无白边。
- White Outline Green：旧方案，在人物完整轮廓外加 1-2 px 纯白描边。
- Dark Contour Green：把边界增强改成风格内的深色外轮廓，避免白色贴纸边。
- Blue Screen：用纯 #0000FF 蓝幕，并用 blue-key 同构后处理测 spill/halo。

## 推荐

1. 默认生产 prompt 不要恢复白边。
2. 优先小改当前绿幕契约：加入“style-native dark ink outer contour, no white outline, no glow”，并继续使用 MODNet_v10。
3. 单独开一轮 blue/cyan key 后处理实验，重点解决 blue spill 和蓝色服饰冲突；不要直接用当前 green-only v10 处理蓝幕。
4. 白边只保留为 emergency/debug 对照项：它能证明边界对比有用，但最终视觉代价太高。

## 单图可视化

### wispy_black_hair_white_blouse

![wispy_black_hair_white_blouse](/Users/august/chromakey-matting-exp-2026-06-17/montages/wispy_black_hair_white_blouse.jpg)

### blonde_curls_dark_coat

![blonde_curls_dark_coat](/Users/august/chromakey-matting-exp-2026-06-17/montages/blonde_curls_dark_coat.jpg)

### silver_hair_translucent_sleeves

![silver_hair_translucent_sleeves](/Users/august/chromakey-matting-exp-2026-06-17/montages/silver_hair_translucent_sleeves.jpg)

### blue_jacket_prop_edges

![blue_jacket_prop_edges](/Users/august/chromakey-matting-exp-2026-06-17/montages/blue_jacket_prop_edges.jpg)

### red_gown_long_hair

![red_gown_long_hair](/Users/august/chromakey-matting-exp-2026-06-17/montages/red_gown_long_hair.jpg)

### spiky_hair_black_outfit

![spiky_hair_black_outfit](/Users/august/chromakey-matting-exp-2026-06-17/montages/spiky_hair_black_outfit.jpg)

### pastel_twin_tails_thin_ribbons

![pastel_twin_tails_thin_ribbons](/Users/august/chromakey-matting-exp-2026-06-17/montages/pastel_twin_tails_thin_ribbons.jpg)

### white_suit_dark_outline

![white_suit_dark_outline](/Users/august/chromakey-matting-exp-2026-06-17/montages/white_suit_dark_outline.jpg)
