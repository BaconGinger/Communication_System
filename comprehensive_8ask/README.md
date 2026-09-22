# 综合考核 · 8ASK 调制解调系统

纯仿真路线（课件允许：*"若条件不允许，也可在 vivado 平台上利用 testbench 仿真测试验证"*），**基带等效**，不做 DDS 载波同步。

## 目录

```
comprehensive_8ask/
├── as_8ask/       Vivado 工程
├── matlab/        bertool 理论曲线 + 实测 BER 绘图脚本
└── data/          testbench 用 $fwrite 导出的误码统计 txt
```

## 系统框图

```
                ┌──────────── FPGA（Verilog）────────────┐
 3bit 数据 ──► 串并转换 ──► 电平映射 ──► 成形滤波 ──► ⊕ ──► 解调 ──► 判决 ──► 并串 ──► 输出比特
              (3→1符号)   (8ASK)       (FIR)        │     (包络)   (7门限)          │
                                                   │                              │
                                          AWGN 噪声源 (LFSR + FIR 整形)          │
                                                                                 ▼
                                                                    MATLAB 算 BER / 画曲线
```

## 分步执行

| # | 步骤 | 抄哪块 | 预算 |
|---|---|---|---|
| 1 | 建工程，搭 LFSR 数据源 + 串并转换 + 电平映射，testbench 看波形 | `gaussian_noise.v` 的 LFSR | 0.5 d |
| 2 | 接 AWGN 噪声源（LFSR + FIR 整形成高斯分布） | `gaussian_noise.v` + `gaussian_noise_filter.v` + `FIR_COE.m` 生成系数 | 0.5 d |
| 3 | 解调：包络检波 + 7 门限判决 + 并串转换。**先跑无噪声，确认 BER = 0** | 自写（最简单的一段） | 0.5 d |
| 4 | 扫 Eb/N0，统计错误比特数导出 txt | testbench 里加循环 + `$fwrite` | 0.5 d |
| 5 | MATLAB：`bertool` 出理论曲线，叠加实测点 | — | 0.5 d |
| 6 | 写报告 | 见下 | 0.5 d |

## 参数（先定死，别边做边改）

- 载波/采样：**基带等效仿真**，不需要 DDS 载波
- 符号率 : 采样率 = **1 : 8**
- 量化位宽：**12 bit 有符号**
- Eb/N0：**0, 2, 4, 6, 8, 10, 12, 15 dB**
- 每点比特数：**≥ 10⁵**

## 抄哪块（参考路径均在 `课件二/` 下）

| 要的 | 工程 | 文件 |
|---|---|---|
| LFSR | `通信系统综合实验5/gaussian_noise` | `gaussian_noise.v` |
| 高斯噪声整形 FIR + 系数 | 同上 | `gaussian_noise_filter.v` / `FIR_COE.m` / `fir_coe.coe` |
| 低通/成形滤波 | `通信系统综合实验7/DSB_modulation` | `fir_coe.coe` + fir_compiler IP |
| 位同步（扩展项） | `通信系统综合实验8/bit_alient_blog` | `phaseDetec.v` / `control.v` / `clk_gen.v` / `moniflop.v` |
| 卷积码（扩展项） | `通信系统综合实验6/pro_conv` | `conv.v` / `tb_conv.v` + Viterbi IP |

## 5 个必修项（少一个扣分）

1. testbench 仿真验证模块
2. MATLAB 调试与联调
3. 窄带高斯白噪声
4. MATLAB 理论误码率（**bertool**）
5. 实际误码率

## 报告结构

1. 实验目的
2. 实验原理 —— 8ASK 调制解调、AWGN 信道模型、`berawgn(EbNo,'pam',8)` 理论误码率推导
3. 系统设计 —— 框图、模块划分、参数表
4. FPGA 实现 —— 各 Verilog 模块说明 + IP 核配置截图
5. testbench 仿真验证 —— 波形截图 + 无噪声闭环 BER = 0 的证据
6. MATLAB 联调 —— 理论曲线 vs 实测点对比图
7. 误码率结果与分析 —— 差异来源（成形滤波的码间串扰、噪声量化、判决门限偏差）
8. 扩展项（做了才写）
9. **问题与解决**（这门课明确要求写，别省）
10. 结论

报告成品放 `docs/reports/`。
