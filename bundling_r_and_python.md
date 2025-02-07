# 🚀 Bundling R and Python Packages into One Application for Cross-Platform Access  

## **Introduction**  

When developing an application that relies on **both Python and R**, one of the main challenges is **dependency management**. Since **PyPI (the official Python Package Index) only supports Python**, I needed an alternative solution to bundle both **R and Python packages together** in a way that is **cross-platform, easy to install, and reproducible**.  

My **ideal solution** had to meet the following requirements:  
✅ **Cross-platform compatibility** (Linux, macOS, Windows, and even Docker)  
✅ **Reproducibility** (consistent dependencies across installations)  
✅ **Upgradability** (easy version control and dependency updates)  
✅ **Flexibility** (different levels of installation options for different users)  

---

## **❌ Why PyPI Doesn’t Work for R & Python Together**  

At first, I considered publishing my application as a **PyPI package** because of its advantages:  

- **Ease of installation**: `pip install mypackage` is simple for users.  
- **Dependency management**: Users automatically get the correct package versions.  
- **Cross-platform**: PyPI works on Windows, macOS, and Linux.  

However, the major **limitation** is that **PyPI only supports Python packages**, making it **impossible to bundle R packages directly**. Since my application required R packages alongside Python, I needed an alternative.  

---

## **✅ My Solution: Conda for Managing R & Python Packages**  

### **What is Conda?**  

[Conda](https://docs.conda.io/en/latest/) is an **open-source package and environment management system** that works across platforms. Unlike PyPI, Conda:  

- **Supports Python, R, and many other languages**.  
- **Manages entire environments** (not just individual packages).  

Conda provides **many R packages** directly, simply by **prefixing the package name with `r-`**. For example:  

```yaml
dependencies:
  - r-optparse  # R package 'optparse'
  - r-tidyverse # R package 'tidyverse'
  - python=3.9  # Python version
  - pandas      # Python package
```

This made it **easy to bundle R and Python together** into a **single Conda environment**.  

---

## **📌 Best Practices for Using Conda**  

While working with Conda, I learned several best practices to ensure **reproducibility, speed, and flexibility**.  

### **1️⃣ Use a Dedicated Environment for Each Project**  
```sh
conda create --name my_project python=3.9
```
This prevents **package conflicts** between projects.  

### **2️⃣ Use `conda-forge` as the Default Channel**  
- **Why?** Anaconda’s terms of service **restrict heavy commercial use**.  
- **Alternative?** `conda-forge`, a community-maintained repository.  
- **Configuration Example (environment.yml)**  

```yaml
channels:
  - conda-forge
  - bioconda
```

### **3️⃣ Avoid Mixing `pip` and `conda`**  
- **Why?** Pip packages are **not tracked** by Conda, causing conflicts.  
- **Exception?** If a package is only available via `pip`, install it carefully:  

```sh
python -m pip install package_name
```
- **Best Practice:** Specify pip dependencies **inside** `environment.yml`:  

```yaml
dependencies:
  - pip
  - pip:
    - ipython_genutils  # Example package available only via pip
```

### **4️⃣ Use `environment.yml` Instead of Manual Installations**  
Instead of installing packages one by one, create an **`environment.yml`** file:  

```yaml
name: my_project
channels:
  - conda-forge
  - bioconda
dependencies:
  - python=3.9
  - r-optparse
  - pandas
  - pip
  - pip:
    - ipython_genutils
```
Then install the environment in one step:  

```sh
conda env create -f environment.yml
```

### **5️⃣ Use `mamba` Instead of `conda` for Faster Installation**  
[Mamba](https://github.com/mamba-org/mamba) is a drop-in replacement for Conda that speeds up dependency resolution:  

```sh
mamba env create -f environment.yml
```

### **6️⃣ Use `conda-lock` for Full Reproducibility**  
**Why?** Regular `environment.yml` files allow flexibility, but for **100% reproducibility**, use [`conda-lock`](https://github.com/conda/conda-lock):  

```sh
conda-lock lock -f environment.yml
```
This ensures that **all users get the exact same package versions**.

---

## **🚀 Installation Options: Balancing Flexibility & Reproducibility**  

I provide **four different installation methods**, each with a **trade-off** between **reproducibility** and **flexibility**:  

### **1️⃣ Docker (Most Reproducible, Least Flexible)**  
✅ **Fully encapsulated environment**  
✅ **No dependency issues**  
❌ **Requires Docker installation**  

```dockerfile
FROM mambaorg/micromamba:1.5.8
COPY environment.yml /tmp/
RUN micromamba create --name my_project --file /tmp/environment.yml && \
    micromamba clean --all --yes
```
To run:  
```sh
docker build -t my_project .
docker run -it my_project
```

---

### **2️⃣ `conda-lock.yml` (Highly Reproducible, Mac & Linux Only)**  
✅ **Guaranteed package versions**  
✅ **Best for development teams**  
❌ **In my case I couldn't lock it for Windows (from macOS)**  

This produces the multi-platform conda-lock.yml (for Mac & Linux, excluding Windows):
```sh
conda-lock -f environment.yml -p osx-64 -p linux-64
```
This installs the environment from the lock file
```sh
conda-lock install -n my_project -f conda-lock.yml
```

---

### **3️⃣ `environment_versioned.yml` (Balanced Option)**  
✅ **Recommended for most users**  
✅ **Allows upgrades while maintaining stability**  
❌ **May result in minor version drift**  

How to generate the `environment_versioned.yml` file:
1. Export the actual environment to environment_export.yml by running
```sh
conda activate my_project
conda env export --no-builds > environment_export.yml
```
2. Manually add versions from the environment_export.yml to environment.yml and save it as environment_versioned.yml
(Reason: the environment_export.yml won’t work on other platforms)
```yaml
dependencies:
  - python=3.9
  - r-optparse=1.6.6
  - pandas=1.4.0
```

Installation:  
```sh
conda env create -f environment_versioned.yml
```

---

### **4️⃣ `environment.yml` (Most Flexible, Least Reproducible)**  
✅ **Users get latest package versions**  
✅ **Easiest to install**  
❌ **Less reproducibility**  

```sh
conda env create -f environment.yml
```

---

### **5️⃣ Manual Installation (Not Recommended)**  
✅ **No extra requirements**  
❌ **No guaranteed reproducibility**  
❌ **User must manually install all dependencies**  

```sh
conda create --name my_project python=3.9
conda install r-optparse pandas
```

---

## **🎯 Conclusion**  

To bundle **R and Python packages together** while maintaining **cross-platform compatibility**, Conda is the best solution.  

✅ **Reproducibility:** `conda-lock.yml` ensures identical environments.  
✅ **Upgradability:** Versioned `environment.yml` files allow controlled updates.  
✅ **Flexibility:** Multiple installation options provide user choice.  
✅ **Speed:** `mamba` significantly improves installation performance.  

If you’re working on a **multi-language project**, Conda **outperforms PyPI** in cases where you need to bundle **R and Python dependencies together**. 🚀  

---

## **💾 Full Code Repository**
📌 Check out the complete implementation on **GitHub**:  
👉 **[GitHub Repo - EnsembleFlex](https://github.com/chembl/EnsembleFlex)**  

---

**I hope this post provides insights and help for your developments!** 😊  

Author: Melanie Schneider
