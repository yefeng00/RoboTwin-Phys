# 关节阻尼卡（拉抽屉系列）notes

场景：`put_object_cabinet` — 036_cabinet（model 46653，三抽 URDF，拉顶层抽屉 joint_1，
行程 qmax=0.1782m）+ 桌面小物体 112_tea-box/base4。机械臂 aloha-agilex，seed=8001，
head_camera D435 原生 320x240，12.5fps，H.264。

## 因子注入

```python
task.cabinet.set_properties(d, 0.0)   # 全部抽屉关节：驱动阻尼=d，刚度=0（纯阻尼器）
```

同一 seed、同一指令轨迹：contact_id=0 抓抽屉把手 → 关节空间直线插值拉 0.22m/3s
（50Hz 控制，250Hz 仿真）。只改 d。

## 档位实测（录制值）

| 档位 | 阻尼 d | 抽屉 q_end (m) | 开度 | 语义 |
|------|--------|----------------|------|------|
| L1 | 8  | 0.1782 | 100% | 全开 |
| L2 | 18 | 0.1238 | 69%  | ~3/4 |
| L3 | 19 | 0.0506 | 28%  | 小半 |
| L4 | 32 | 0.0163 | 9%   | 一条缝 |
| L5 | 96 | 0.0035 | 2%   | 拉不动 |

单调下降 ✓。每段 65 帧 = 5.2s（4-6s 规格内）。

## 失败卡 drawer_fail.mp4（d=96）

- 段 1：夹爪拉抽屉，q_after_tug=0.0035（2%，拉不动）；
- 段 2：抓 tea-box 举到抽屉前方上方（z=0.902）松手 → 落在抽屉外桌面（z=0.742，y=-0.046）。
- 176 帧 = 14.08s（8-15s 规格内）；转运段 2x 抽帧、物理段（拽/落）1x。

## 梯度机理与标定过程

- 低 d：夹爪咬住把手，抽屉全程跟随（track）→ 全开。
- d≈18-19：抓持力阈值悬崖 —— 阻尼反力 d·v 超过把手抓持摩擦，开始滑脱。
- 高 d：夹爪沿把手滑脱，抽屉仅被动摩擦拖动微爬，q_end ≈ 0.45/d（d=48/64/96 实测吻合，
  该区可复现）。
- **contact_id=0 是关键**：默认自动选点抓得浅，d=1 即滑脱且响应非单调（d=1→20%，
  d=4→13%，d=16→46% 乱序）；改用接触点 0 深抓后全区间有序。
- 复现性：track 区（d≤16）与爬行区（d≥32）稳定；悬崖区（17-22）运行间有 ±20% 开度
  方差（如 d=18 两次测得 69%/90%）。上表为最终成片实测值，五档单调成立。
- 标定扫描（contact_id=0）：d=8→100%，16→79%，17→88%，18→69/90%，19→47/28%，
  20→26%，22→35%，24→23%，32→11/9%，48→6%，64→4%，96→2%，256→0.1%，1024→0%。

## 复现命令（服务器 /data/yefeng/homepage_videos/damping_v2/）

```bash
CUDA_VISIBLE_DEVICES=7 python record_damping_v2.py record_a \
    --levels L1:8,L2:18,L3:19,L4:32,L5:96 --contact-id 0 --seed 8001
CUDA_VISIBLE_DEVICES=7 python record_damping_v2.py record_b \
    --fail-damping 96 --contact-id 0 --seed 8001
python record_damping_v2.py grid --qmax 0.1782   # 生成 compare 五格图
```

原始 pkl 帧缓存与 meta.json 在服务器 raw/ 子目录；scan_results.json 为标定扫描存档。
