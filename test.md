# DDPM 去噪扩散模型 完整处理流程
## 符号定义
$x_0$：清晰样本 | $x_t$：t步加噪样本 | $T$：总步数 | $\epsilon$：高斯噪声
$\bar{\alpha}_t=\prod_{i=1}^t\alpha_i$ | $\epsilon_\theta$：UNet预测噪声 | $\mathcal{L}$：损失函数

---

## 1. 正向扩散过程（固定加噪，无训练）
输入：
清晰图像 $x_0$
↓
逐步加噪（递推公式）：
$x_t = \sqrt{\alpha_t} x_{t-1} + \sqrt{1-\alpha_t}\epsilon$
↓
加速计算（直达公式）：
$x_t = \sqrt{\bar{\alpha}_t} x_0 + \sqrt{1-\bar{\alpha}_t}\epsilon$
↓
终止状态：
纯高斯噪声 $x_T \sim \mathcal{N}(0,I)$

---

## 2. 模型训练过程（学习预测噪声）
随机采样时间步
$t \in [1,T]$
↓
生成加噪样本 $x_t$
↓
UNet模型预测噪声：
$\hat{\epsilon} = \epsilon_\theta(x_t, t)$
↓
计算损失：
$\mathcal{L} = \mathbb{E}\left\|\epsilon - \epsilon_\theta(x_t,t)\right\|_2^2$
↓
反向传播更新参数 $\theta$
↓
模型训练完成

---

## 3. 逆向推理过程（生成清晰图像）
初始化纯噪声$x_T$
↓
逆向迭代去噪（$t=T \to 1$）：
$x_{t-1} = \frac{1}{\sqrt{\alpha_t}}\left(x_t - \frac{1-\alpha_t}{\sqrt{1-\bar{\alpha}_t}}\epsilon_\theta\right) + \sigma_t z$
↓
迭代至 $t=0$
↓
输出清晰生成样本$\hat{x}_0$
↓
结束