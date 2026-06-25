# Phase Transformation Kinetics Modelling using ML

## 📌 Project Overview
This repository contains the source code, data pipelines, and machine learning models developed for the **MLL 202 Mini-Project at IIT Delhi**. 

The project computationally models and predicts the competition between **nucleation and growth rates** during solid-state phase transformations. By combining fundamental physical equations with machine learning surrogates, this framework explains why nucleation and growth rates peak at different temperatures and demonstrates how their competitive interaction governs the characteristic "C-curve" behavior in **Time-Temperature-Transformation (TTT)** diagrams.

---

## 🔬 Physics Layer & Theoretical Framework

The underlying data generation is anchored in first-principles thermodynamics and kinetics across **40 real metallic systems** spanning FCC, BCC, HCP, and diamond-cubic structures.

### 1. Classical Nucleation Theory (CNT)
The steady-state nucleation rate $I(T)$ is modeled as:
$$I(T) = Z \cdot \beta^* \cdot N_v \cdot \exp\left(-\frac{\Delta G^*}{k_B T}\right)$$

Where the classical critical nucleation barrier $\Delta G^*$ is expressed as:
$$\Delta G^* = \frac{16 \pi \gamma^3}{3 \Delta G_v^2}$$

* **Kinetic Behavior:** Near the melting temperature ($T_m$), the driving force approaches zero ($\Delta G^* \rightarrow \infty$), halting nucleation. At low temperatures, atomic diffusion ceases ($D \rightarrow 0$). Consequently, the nucleation rate peaks at an intermediate temperature range of approximately **$0.68 - 0.78 \ T_m$**.

### 2. Wilson-Frenkel Interface-Controlled Growth
The interface growth velocity $U(T)$ is governed by:
$$U(T) = \frac{D}{a} \left[1 - \exp\left(-\frac{\Delta G_v V_m}{RT}\right)\right]$$

* **Kinetic Behavior:** The growth rate peaks lower than the nucleation rate, at approximately **$0.55 - 0.70 \ T_m$**, because the atomic mobility factor ($D/a$) drops drastically at lower temperatures before the thermodynamic driving force can fully compensate.

### 3. Johnson-Mehl-Avrami-Kolmogorov (JMAK) Kinetics
For a three-dimensional continuous nucleation and growth process ($n = 4$), the volume fraction transformed $X(t, T)$ is given by:
$$X(t, T) = 1 - \exp\left(-k(T) \cdot t^n\right)$$

Where the overall kinetic rate constant $k(T)$ synthesizes both mechanisms:
$$k(T) = \frac{\pi}{3} I(T) \cdot U(T)^3$$

TTT curves representing specific transformation stages (e.g., 1%, 50%, 99%) are explicitly derived by mathematically inverting the JMAK equation.

---

## 📊 Dataset Construction & Feature Engineering

A structured dataset consisting of **10,000+ physics-computed data points** was compiled (40 metallic materials evaluated over 250 temperature steps each). 

To ensure cross-material generalizability, all input features were transformed into **dimensionless physical descriptors**:
* **Reduced Temperature:** $T/T_m$
* **Undercooling:** $(T_m - T)/T_m$
* **Stefan Number:** $\Delta H_f / R T_m$
* **Reduced Activation Energy:** $Q / R T_m$
* **Dimensionless Surface Energy:** $V_r$

### Data Splitting
* **Training Set:** 30 metallic systems
* **Hold-out Test Set:** 10 completely unseen metallic systems (used strictly for validating ML generalization capabilities).

---

## 🤖 Machine Learning Surrogates

To eliminate the high computational overhead of running multi-scale physical equations from scratch, two types of ML surrogates were implemented:
1. **Gradient Boosting Machine (GBM) Surrogates:** Trained on $\log_{10}$-transformed rate data to accurately capture exponential variations in $I(T)$, $U(T)$, and $k(T)$.
2. **Random Forest Regressors:** Trained to directly predict the critical TTT nose parameters: nose temperature ($T_{\text{nose}}$) and nose time ($t_{\text{nose}}$).

### 📈 Model Performance on Unseen Metals
The trained models demonstrate exceptional predictability when tested against the 10 completely independent hold-out metallic systems:

| ML Model | Target Property | Hold-out $R^2$ (Unseen Metals) |
| :--- | :--- | :---: |
| **GBM Surrogate** | Nucleation rate $I(T)$ | **0.971** |
| **GBM Surrogate** | Growth rate $U(T)$ | **0.968** |
| **GBM Surrogate** | JMAK rate constant $k(T)$ | **0.975** |
| **Random Forest** | TTT nose temperature $T_{\text{nose}}$ | **0.952** |
| **Random Forest** | TTT nose time $t_{\text{nose}}$ | **0.945** |

---

## 💡 Key Findings & Novel Contributions

1. **Quantified Peak Separation:** Across all 40 metals, the nucleation rate $I(T)$ consistently peaks at higher reduced temperatures than the growth rate $U(T)$. The mean peak separation is quantified as:
   $$\Delta T = T(I_{\text{peak}}) - T(U_{\text{peak}}) \approx 70 - 150^\circ\text{C}$$
   This happens because nucleation is strictly barrier-controlled, whereas growth is predominantly mobility-controlled.
2. **Origin of the TTT C-Curve:** The fastest transformation rate (the "nose" of the TTT curve) occurs exactly at the temperature where the product $I \cdot U^3$ is maximized.
3. **Reflected Refractory vs. Low-Melting Kinetics:** High- $T_m$, high- $Q$ refractory metals (e.g., W, Mo, Re) exhibit nose times spanning hours to days, whereas low- $T_m$ metals (e.g., Sn, Bi, Pb) complete phase transformations in milliseconds near their nose.
4. **Universal Dimensionless Scaling:** An $R^2 > 0.95$ validates that five dimensionless descriptors are sufficient to encode universal phase transformation kinetics across diverse crystalline structures (FCC, BCC, HCP, diamond-cubic).

---

## 👥 Contributors

| Name           | Roll Number |
|----------------|-------------|
| Rajgowtham M   | 2024MS10186 |
| Vignesh K      | 2024MS10669 |
| Suhadhri Das   | 2024MS10704 |
| Praveen Kumar  | 2024MS10784 |
| Srikanth       | 2024MS10567 |
| Hadia          | 2022MS12040 |

---

## 📚 References
1. Turnbull, D. & Fisher, J. C. (1949). *J. Chem. Phys.*, 17, 71.
2. Johnson, W. A. & Mehl, R. F. (1939). *Trans. AIME*, 135, 416.
3. Kelton, K. F. & Greer, A. L. (2010). *Nucleation in Condensed Matter*. Pergamon.
4. Brandes, E. A. & Brook, G. B. (1998). *Smithells Metals Reference Book*, 7th ed. Butterworth-Heinemann.
5. ASM International (1992). *ASM Handbook Vol. 3: Alloy Phase Diagrams*.
