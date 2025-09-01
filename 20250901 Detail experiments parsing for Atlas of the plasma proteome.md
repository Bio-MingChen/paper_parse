# Parsing experiments in Atlas of plasma proteome in health and disease in 53026 adults

[toc]

In this article, I have attempted to parse each step of the paper and clarify the analytical details and methods, in order to better understand them and apply them to other projects.

## Population characteristics and phenotypes

Diseases and traits in this research are listed in [Table S3](./supplemental%20information/20250901/table_S3.xlsx) [Table S4](./supplemental%20information/20250901/table_S4.xlsx) and [Table S5](./supplemental%20information/20250901/table_S5.xlsx)

The QC procedure of this part described as follow:

### Disease definition（疾病定义与QC步骤）

1. **数据来源**：UK Biobank 的 **电子病历 (EHR)**，疾病用 ICD-10 编码。
2. **Prevalent vs Incident 定义**：

   * *Prevalent disease*：发生在基线之前或基线当时。
   * *Incident disease*：基线之后首次发生，处理为 **time-to-event 数据**。
3. **QC 标准**：

   * 采用 **FinnGen disease endpoint code** 及其质量控制准则。
   * 应用性别/年龄的预定义条件，对某些疾病排除特定对照。
   * 原始 ICD-10 编码保留一位小数（因 UKB 数据仅有一位）。
   * 若疾病在基线前已有诊断，则不计入 incident；若在随访中才出现，则不计入 prevalent。
   * 每个疾病的 **对照组** = 未患该病的其余参与者。
   * 排除病例数 **少于 100** 的疾病终点。最终保留 **660 个 incident** 和 **406 个 prevalent** 疾病终点。

---

### Health-related traits（表型定义与QC步骤）

1. **数据来源**：UKB 的基线体检和问卷，包括身体测量、饮食、活动、生活方式、环境、血液和尿液指标等。
2. **处理方式**：

   * 使用 **PEACOK R 包**（基于 PHESANT）对变量进行分类。
   * 将变量划分为三类：**连续型、二分类、有序分类**。
3. **QC 步骤**：

   * 排除样本量 **<10,000** 的变量。
   * 二分类变量要求病例数和对照数均 **≥50**。
   * 最终纳入 **453 个连续型 trait**、**331 个有序分类 trait**、**202 个二分类 trait**。
   * 每个 trait 分配到 UKB 的二级或三级路径，用于分章分析。

---

### Summary

这部分的 QC 非常严格，保证了疾病和表型的统计学可靠性。具体步骤可以总结为：

1. **疾病 (disease definition)**

   * 使用 ICD-10 + FinnGen code
   * 区分 prevalent / incident
   * 排除 <100 病例的终点
   * 严格对照定义 + 基线前后排除规则

2. **健康相关表型 (traits)**

   * 基于 PEACOK 包分类
   * 设定最小样本量阈值（≥10,000）
   * 二分类变量需 ≥50 个病例/对照
   * 共纳入 986 个 traits，覆盖多维健康表型

---

### 疾病（diseases）

* 文章中 **prevalent/incident diseases** 是根据 **ICD-10** 编码整理的。
* 作者没有自己重新创造大类，而是按照 **ICD-10 chapter（章节）** 来分组，比如循环系统疾病、消化系统疾病、呼吸系统疾病等。
* 也就是说：这些疾病的 **chapter 分类本来就在 ICD-10 体系里**，UKB 提供的是 ICD-10 码，作者直接按照国际标准的 chapter 汇总（只是有些 ICD-10 二位小数在 UKB 里只保留了一位小数，作者提到过这个限制）。

---

### 健康相关表型（health-related traits）

* 这里作者不是用 ICD-10，而是用 **UKB 自己的 field code + PEACOK 包** 做分类。
* UKB 的 trait 数据包括身体测量、生活方式、饮食、NMR 代谢组学等，原始字段很多。
* 作者基于 UKB 的 **数据路径（UKB path）** 把 986 个 traits 分成了 **11 个章节**（chapters），比如 “NMR metabolomics”、“cognitive function”、“mental health traits”等。
* 这些章节并不是 ICD-10 的，而是 **UKB 已经有的 trait 路径体系**，作者在此基础上做了一些合并和整理。

---

✅ 总结：

* **疾病的 chapter** → 来源于 **ICD-10 官方章节**（UKB 本来就提供 ICD-10 码）。
* **Traits 的 chapter** → 来源于 **UKB 的 trait 路径体系**，作者在 UKB 原有的分类基础上整理。

### 协变量选择
---

| Category / 类别                                | Covariate / 协变量                  | 中文说明            | 备注              |
| -------------------------------------------- | -------------------------------- | --------------- | --------------- |
| **Demographic variables / 人口学变量**            | Age                              | 年龄              | 连续变量            |
|                                              | Sex                              | 性别              | 男 / 女           |
|                                              | Ethnicity                        | 种族              | 白人、亚洲人、黑人、混合及其他 |
|                                              | Townsend deprivation index (TDI) | Townsend 区域剥夺指数 | 社会经济地位指标        |
| **Sample & measurement factors / 样本与检测相关因素** | Time fasted at blood collection  | 采血时空腹时间         | 小时数             |
|                                              | Season of sample collection      | 采样季节            | 夏/秋 vs 冬/春      |
|                                              | Sample age                       | 样本年龄            | 从采样到蛋白质检测的时间间隔  |
| **Health-related variables / 健康相关变量**        | Body mass index (BMI)            | 体质指数            | 体重(kg)/身高²(m²)  |
|                                              | Smoking status                   | 吸烟状态            | 从未、既往、当前        |


## Atlas of protein and diseases/traits associations

---

### 总览（输入与范围）

* **样本与随访**：UKB-PPP 中 **53,026** 名参与者；事件随访从基线到首次确诊/死亡/或住院记录最晚日期（**2023年11月**）的最早者；做 incident 分析时，**已有该病的个体在分析中被排除**。
* **蛋白测量**：Olink Explore，使用 **NPX** 归一化强度；>50% 缺失的蛋白剔除后纳入 **2,920** 个蛋白。
* **疾病终点**：依据 ICD-10 住院记录定义 **prevalent（基线前/当时）** 与 **incident（基线后）** 疾病；按 FinnGen 端点与 QC 规则处理；**病例数 <100 的终点剔除**（最终 406 prevalent + 660 incident）。
* **协变量**（所有模型均调整）：年龄、性别、种族、TDI、空腹时间、采样季节、样本年龄、BMI、吸烟状态；缺失以**中位数填补**（适用于若干项）。

---

### A. 蛋白–疾病关联（Atlas of protein–disease association）

#### A1. Prevalent diseases（横断面）

**目标**：评估基线蛋白水平与“基线时/之前已发生”的疾病是否相关。

**步骤**

1. **为每个疾病构建二元结局**：是否为该 prevalent 病例（对照=其余未患该病者）。
2. **单蛋白回归**：逐一对 **2,920 蛋白 × 406 疾病** 运行 **logistic 回归**（对每个蛋白–疾病对做一元暴露、多协变量校正）。模型实现：**statsmodels v0.13.1（Python 3.9.16）**。
3. **协变量调整**：纳入上节列出的协变量。
4. **显著性阈值（多重校正）**：**Bonferroni p < 0.05 / (2,920 × 406)**。
5. **结果记录**：输出 **OR、95%CI、p 值**；显著关联的方向与 p 值见 Fig. 1B。

> 复现提示：prevalent 模型不涉及随访时间，只需构建病例/对照并校正协变量与技术因素（如季节、空腹时间等）。

---

#### A2. Incident diseases（纵向）

**目标**：评估基线蛋白水平对“新发事件”的风险。

**步骤**

1. **时间-事件数据准备**：起始=基线；结束=首次确诊/死亡/或住院记录截止（2023-11）；**已有该病的个体不进入该病的 incident 分析**。
2. **单蛋白生存模型**：对 **2,920 蛋白 × 660 疾病** 逐一拟合 **Cox 比例风险模型**，协变量同上。实现：**lifelines CoxFitter v0.27.4（Python 3.9.16）**。
3. **显著性阈值（多重校正）**：**Bonferroni p < 0.05 / (2,920 × 660)**。
4. **结果记录**：输出 **HR、95%CI、p 值**；显著关联的方向与 p 值见 Fig. 1C。

#### B. 蛋白–健康相关性状（traits）关联（并列分析模块）

**目标**：把 **986 个 health-related traits** 当作结局，评估基线蛋白水平与性状的关系。

**步骤**

1. **按结局类型选模型**：

   * 连续型 → **线性回归（lm）**；
   * 二分类 → **logistic 回归（glm）**；
   * 有序分类 → **比例优势模型（polr）**；
     实现：R **MASS v4.2.0**。
2. **协变量调整**：同上所列协变量。
3. **显著性阈值（多重校正）**：**p < 1.71×10⁻⁸**（≈ 0.05 /（2,920 × \~1,000 traits））。
4. **Trait 的 QC 要求（来源于方法“Health-related trait”）**：

   * 合并蛋白样本后，样本量 **<10,000** 的变量剔除；
   * 二分类 trait 要求病例与对照 **各≥50**；
   * 共保留 **453 连续、331 有序、202 二分类** trait。

---

#### C. 产出清单（你复现时建议保存的表）

* **Prevalent**：每个蛋白×疾病的 OR、95%CI、p、是否显著（Bonferroni）。
* **Incident**：每个蛋白×疾病的 HR、95%CI、p、是否显著（Bonferroni）。
* **Traits**：每个蛋白×trait 的 β/OR（或对数优势）、SE、p、是否显著（1.71×10⁻⁸）。

---

#### D. 复现实操要点（容易踩坑的地方）

1. **疾病对照定义**：每个疾病单独建模，**对照=队列中未患该病的其他参与者**；incident 分析需先排除基线已病者。
2. **多重比较**：一定用文中给出的 **Bonferroni 阈值**，且 prevalent/incident 的分母不同（406 vs 660）。
3. **协变量一致**：三个模块（prevalent、incident、traits）**协变量同一套**并按文中规则**中位数填补**缺失。
4. **端点筛选门槛**：疾病终点 **病例≥100**；traits 有各自的样本量/事件数门槛。
5. **软件版本**：Python 3.9.16 + statsmodels 0.13.1 / lifelines 0.27.4；R MASS 4.2.0。


---
## 重要名词解释

### ICD-10


| 名称         | 全称                                                        | 中文        | 说明                                                        |
| ---------- | --------------------------------------------------------- | --------- | --------------------------------------------------------- |
| **ICD-10** | *International Classification of Diseases, 10th Revision* | 国际疾病分类第十版 | 由世界卫生组织（WHO）制定的疾病和相关健康问题的国际标准分类体系。它为疾病和各种健康状况提供了统一的代码和定义。 |

---

1. **背景与制定**

   * 由 **世界卫生组织（WHO）** 负责制定和更新。
   * 1990 年正式通过，1994 年起在全球逐步推广使用。
   * 目前已被大多数国家的卫生系统、医保系统和科研数据库采用。

2. **结构特点**

   * 使用 **字母+数字编码**（例如：C91.0 = 急性淋巴细胞白血病，ALL）。
   * 分为 **21 个章节（chapters）**，涵盖传染病、肿瘤、循环系统疾病、呼吸系统疾病、精神障碍等。
   * 每个疾病章节下再细分为更具体的三级或四级编码。

3. **主要用途**

   * **临床诊断**：医生在病历和出院小结中常用 ICD-10 编码记录疾病。
   * **医保与统计**：医保结算、卫生统计、疾病负担研究均依赖 ICD-10 编码。
   * **科研分析**：像 UK Biobank 这种大型队列数据库，就用 ICD-10 来定义疾病终点。

4. **在本研究（Plasma Proteome Atlas）中的应用**

   * 作者利用 UKB 的住院记录（ICD-10 编码）来定义 **prevalent disease** 和 **incident disease**。
   * 再按 ICD-10 的章节将这些疾病归类，比如 **循环系统 (Chapter IX)**、**消化系统 (Chapter XI)** 等。

---
