# Predicting-Diabetes-Status-from-Lifestyle-and-Health-Indicators-with-Spark
 
### *A Comparative Study of Distributed Machine Learning Algorithms*

---

## 🧠 Abstract
This project implements and compares four machine-learning algorithms for diabetes prediction using **Apache Spark MLlib** on a distributed computing cluster.  
We processed **253,680 patient records** with **21 health indicators**, implementing:
- Random Forest  
- Logistic Regression  
- Gradient Boosting Trees  
- Multi-Layer Perceptron  

Key findings:
- All algorithms achieved similar accuracy (**86.5 – 86.8 %**)  
- 2-VM runs were slower than 1-VM runs due to communication overhead  
- Distributed performance benefits depend on dataset size, CPU cores, and network speed  

---

## 1️⃣ Introduction

### 1.1 Background
Diabetes affects millions worldwide; early detection is vital.  
Machine learning enables data-driven risk prediction using routine health indicators.  
This project leverages **Apache Spark’s distributed framework** to implement and compare models for diabetes prediction.

### 1.2 Objectives
- Implement four ML algorithms with Spark MLlib  
- Analyze 253 K records / 21 features  
- Compare accuracy, AUC, F1, and training time  
- Evaluate 1 VM vs 2 VM performance  
- Identify factors influencing distributed speedup  

### 1.3 Dataset
**Source:** CDC BRFSS 2015 Survey  
Contains BMI, age, blood pressure, cholesterol, smoking, physical activity, and lifestyle factors.  
Binary label → diabetes presence (yes/no).  

---

## 2️⃣ Infrastructure and Setup

| Component | Specification |
|------------|---------------|
| **Platform** | Apache Spark 3.5.0 + Hadoop 3 |
| **Cluster** | 2 VMs (1 Master + 1 Worker) |
| **Resources** | 1 CPU core + 2 GB RAM per VM |
| **OS** | Fedora Linux (vSphere) |
| **Language** | Python 3 with PySpark |

### 2.2 Data Pre-processing
- Loaded 253 680 records (CSV)  
- Feature vector assembly → `VectorAssembler`  
- Standardization → `StandardScaler`  
- Split: 70 % train / 15 % val / 15 % test  
- Stored in Parquet for distributed loading  

---

## 3️⃣ Methodology

### 3.1 Random Forest
- Trees = 100, MaxDepth = 5  
- FeatureSubset = auto, Seed = 42  

### 3.2 Logistic Regression
- MaxIter = 100, Reg = L2, Tol = 1e-6  

### 3.3 Gradient Boosting Trees
- Iterations = 20, Depth = 4, LearningRate = 0.1, Seed = 42  

### 3.4 Multi-Layer Perceptron (MLP)
- Architecture = 21 → 32 → 16 → 2  
- Iterations = 50, Block = 128  

---

## 4️⃣ Results

### 4.1 Performance Summary

| Algorithm | VMs | Accuracy | AUC-ROC | F1 Score | Train Time (s) |
|------------|-----|-----------|----------|-----------|----------------|
| Random Forest | 1 | **86.80 %** | 83.57 % | 82.55 % | 103.25 |
| Random Forest | 2 | 86.80 % | 83.57 % | 82.55 % | 107.26 |
| Logistic Regression | 1 | 86.56 % | 81.41 % | 82.27 % | **13.43** |
| Logistic Regression | 2 | 86.56 % | 81.41 % | 82.27 % | 14.16 |
| Gradient Boosting | 1 | 86.55 % | 81.75 % | **83.51 %** | 66.91 |
| Gradient Boosting | 2 | 86.55 % | 81.75 % | 83.51 % | 75.35 |
| MLP (DNN) | 1 | 86.61 % | **81.85 %** | 81.98 % | 47.56 |
| MLP (DNN) | 2 | 86.61 % | 81.85 % | 81.98 % | 49.84 |

---

### 4.2 Speedup Analysis

| Algorithm | Speedup (1 VM / 2 VMs) | Observation |
|------------|-----------------------|--------------|
| Random Forest | 0.96× (4 % slower) | Overhead > benefit |
| Logistic Regression | 0.95× (5 % slower) | Coordination cost |
| Gradient Boosting Trees | 0.89× (11 % slower) | Sequential trees limit parallelism |
| MLP (DNN) | 0.95× (5 % slower) | Sync updates add delay |

---

## 5️⃣ Analysis and Discussion

### 5.1 Algorithm Comparison
- **Accuracy:** All ≈ 86.5 – 86.8 %; dataset may have natural ceiling.  
- **Training Speed:** Logistic Regression fastest (13 s), Random Forest slowest (103 s).  

### 5.2 Distributed Performance

#### 5.2.1 Counter-Intuitive Finding
Adding a second VM **reduced speed** (4–11 % slower) → coordination overhead outweighs parallel benefits.  

#### 5.2.2 Causes of Overhead
- Low CPU count (1 core/VM)  
- Network latency & synchronization delays  
- Serialization costs (5–15 % shuffle overhead)  
- Moderate dataset (< 1 M records) fits in memory  
- **Amdahl’s Law:** 15–20 % sequential portion limits max speedup ≈ 1.2×  

#### 5.2.3 Algorithm-Specific Trends
- Gradient Boosting → largest slowdown (11 %) due to sequential tree building  
- Random Forest → slightly better as trees are independent  
- Logistic Regression & MLP → iterative sync overhead ≈ 5 %  

### 5.3 Best Practices for Distributed ML
Use distributed clusters only when:  
- Dataset ≫ 1 M records  
- ≥ 8 CPU cores per worker  
- ≥ 10 Gbps network interconnect  
- Algorithms are embarrassingly parallel  
- Infrastructure cost < time saved  

For moderate datasets (100 K–1 M records), a single multi-core machine is faster and simpler.  

---

## 6️⃣ Conclusions

### 6.1 Key Findings
- All models → ≈ 87 % accuracy ceiling  
- Logistic Regression = 7.7× faster than Random Forest  
- 2-VM setup slower by 4–11 %  
- Distributed ML needs large data, multi-core nodes, and fast network  

### 6.2 Practical Implications
For datasets ≈ 250 K rows:  
➡️ Prefer single-machine Spark jobs for efficiency and lower costs.  

### 6.3 Future Work
- Test 5–10 M records  
- Scale VMs to 4–8 cores  
- Hyperparameter tuning  
- Compare Spark vs Dask/Ray and GPU training  
- Explore feature engineering to exceed 87 % accuracy  

### 6.4 Final Remarks
Distributed computing is not always faster — it depends on data scale and resources.  
A well-optimized single machine can outperform a cluster when the dataset fits in memory.  

---

## 📚 References
1. Apache Spark MLlib (2024). *Machine Learning Library.* <https://spark.apache.org/mllib/>  
2. CDC (2015). *Behavioral Risk Factor Surveillance System Survey Data.* U.S. DHHS.  
3. Breiman L. (2001). *Random Forests.* Machine Learning 45(1): 5–32.  
4. Friedman J.H. (2001). *Greedy Function Approximation: A Gradient Boosting Machine.* Ann. Stat. 29(5): 1189–1232.  
5. Rumelhart D.E., Hinton G.E., Williams R.J. (1986). *Learning Representations by Back-propagating Errors.* Nature 323(6088): 533–536.  
6. Zaharia M. et al. (2016). *Apache Spark: A Unified Engine for Big Data Processing.* CACM 59(11): 56–65.  
7. Amdahl G.M. (1967). *Validity of the Single Processor Approach to Achieving Large-Scale Computing Capabilities.* AFIPS 30: 483–485.  
8. Dean J., Ghemawat S. (2008). *MapReduce: Simplified Data Processing on Large Clusters.* CACM 51(1): 107–113.  

---

✅ **How to Cite**  
Meteku M.N. & Owusu D. (2025). *Predicting-Diabetes-Status-from-Lifestyle-and-Health-Indicators-with-Spark: A Comparative Study of Distributed Machine Learning Algorithms.* Michigan Technological University.  
