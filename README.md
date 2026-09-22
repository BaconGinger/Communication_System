# 通信系统综合实验

哈尔滨工程大学 · 通信系统综合实验课程设计。用 Xilinx Vivado + Verilog 实现，MATLAB 做理论推导与结果对比。

| 考核 | 选题 | 交付物 |
|---|---|---|
| 基础考核 | **LFSR 伪随机序列** | 基础实验报告 + 工程文件 |
| 综合考核 | **8ASK 调制解调系统** | 综合预习报告 + 综合实验报告 + 工程文件 |

---

## 目录结构

```
Communication_System/
├── basic_lfsr/            基础考核：LFSR 伪随机序列
│   ├── lfsr/              Vivado 工程（.xpr + .srcs 是全部交付物）
│   ├── matlab/            m 序列性质验证脚本
│   └── data/              仿真导出的序列
│
├── comprehensive_8ask/    综合考核：8ASK 调制解调系统
│   ├── as_8ask/           Vivado 工程
│   ├── matlab/            bertool 理论曲线 + 实测 BER 绘图
│   └── data/              testbench 导出的误码统计
│
└── docs/
    └── reports/           报告产出（基础报告 / 综合预习报告 / 综合实验报告）
```

每个实验目录下另有自己的 `README.md`，说明该目录内每个文件的作用。

## 入库范围

版本库里只放**源码、工程文件（`.xpr` + `.srcs`）、IP 配置（`.xci`）、系数文件（`.coe`）和报告**。

综合、实现、仿真过程中产生的一切缓存、网表、波形、日志均已由 `.gitignore` 排除，可随时从源码重建。顶层采用白名单策略：根目录下新出现的内容默认不入库，需显式放行。

## 运行

### Vivado 行为仿真

用图形界面打开对应的 `.xpr`，或走 Tcl 批处理（更快、可重复）：

```bash
vivado -mode batch -source run_sim.tcl
```

### MATLAB

```bash
matlab -batch "mseq_verify"    # 无界面跑脚本并退出，适合批处理出图
```

## 环境

| | 版本 |
|---|---|
| Vivado | 2018.3 |
| MATLAB | R2024b，需 Communications Toolbox（提供 `berawgn` / `bertool`） |

---

## 设计要点

**基础实验 LFSR** —— 32 位线性反馈移位寄存器，按本原多项式取抽头，输出最大长度序列（m 序列，周期 `2³² − 1`）。testbench 中验证 m 序列的性质，MATLAB 侧独立复算比对。

**综合实验 8ASK** —— 采用**基带等效仿真**，不做 DDS 载波。调制端每 3 bit 映射为 8 个双极性电平（±A, ±3A, ±5A, ±7A），解调端包络检波 + 7 门限判决。双极性映射等效于 8-PAM，理论误码率可直接用 `berawgn(EbNo,'pam',8)` 对上；若改用单极性 0…7A，则与理论值相差 3 dB。
