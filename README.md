# RSForge — Reed-Solomon 纠错锻造炉

<p align="center">
  <a href="https://github.com/CJX0712/rs-forge/actions/workflows/ci.yml"><img src="https://github.com/CJX0712/rs-forge/actions/workflows/ci.yml/badge.svg" alt="ci"></a>
  <a href="https://github.com/CJX0712/rs-forge/releases"><img src="https://img.shields.io/github/v/release/CJX0712/rs-forge?sort=semver" alt="release"></a>
  <a href="https://github.com/CJX0712/rs-forge/blob/main/LICENSE"><img src="https://img.shields.io/github/license/CJX0712/rs-forge" alt="license"></a>
  <img src="https://img.shields.io/badge/author-%E6%99%A8%E6%98%9F-1f6feb" alt="author">
</p>

单文件离线 Reed-Solomon 前向纠错（FEC）工作台。零依赖、离线可用、浏览器直接打开。

## 能干什么

- **编码**：任意文本（UTF-8 字节）→ RS 系统码（消息原样在前 + nsym 个校验字节在后）
- **注错**：一键注入 t = ⌊nsym/2⌋ 个随机字节错误（红色标记）
- **纠错**：Berlekamp-Massey 定位 + Chien 搜索 + GF(256) 高斯消元求幅值，精确还原并标出每个错误位置（绿色标记）
- **调参**：nsym = 2–16（纠错能力 t = 1–8），码字上限 255 字节

与 QR 码、CD、DVB、Voyager 深空通信同源的数学：GF(256)，本原多项式 0x11D。

## 数学内核

| 模块 | 实现 |
|---|---|
| 有限域 | GF(256) exp/log 表（α = 2，p(x) = x⁸+x⁴+x³+x²+1） |
| 生成多项式 | g(x) = Π(x − αⁱ)，i = 0..nsym−1（已缓存） |
| 编码 | 综合除法求余数（系统码：消息区还原，余数作校验） |
| 综合症 | Sⱼ = C(αʲ)，j = 0..nsym−1 |
| 错误定位 | Berlekamp-Massey 求 Λ(x)，Chien 搜索找根 |
| 错误幅值 | 已知位置下解 Sⱼ = Σ eᵢ·Xᵢʲ（GF 高斯-约当消元） |
| 自校验 | 纠回后重算综合症，非零即拒收（绝不静默出错） |

## 无头验证（Node，9/9 全绿）

1. GF 域公理：exp/log 往返、乘法逆元、255 个非零元两两不同、分配律+结合律 2000 随机例
2. 生成多项式：首一、次数正确、α⁰..αⁿˢʸᵐ⁻¹ 全零点（6 种 nsym）
3. 系统码：消息区逐字节不变 + 综合症全零
4. **闭环纠错：400 例随机注入（共 1005 个错误）→ 字节精确还原 + 位置逐一命中**
5. 单错穷举：每个位置 × 3 种值（80 例）全还原
6. 全值域字节（含 0x00/0xFF）长 200 nsym=16 注 8 错精确还原
7. 确定性
8. 超能力注错（t+2）：不崩溃、结果自洽（>t 错误理论允许检不出）
9. 编码 → 注 2 错 → 解码 → 原文往返

```bash
node _smoke.js   # 9/9
node _probe.js   # ASCII 探针：幅值精确解出注入的 XOR delta
```

## 用法

浏览器打开 `index.html` 即可。编码 → 注错 → 纠错，网格实时变色；内置自检面板随时重跑核心断言。

## License

MIT
