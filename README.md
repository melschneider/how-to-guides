# 📚 How-To Guides for Computational Scientists  

Welcome to my **"how-to-blog"** repository! This collection of blog posts provides **practical guidance** on key topics for **scientific computing**, **data visualization**, and **cross-platform software development** in Python and R.  

Each guide covers **common challenges**, **best practices**, and **real-world solutions** to help you streamline your workflows. 🚀  

---

## 📖 **Blog Posts**  

### 🏗️ **1. Bundling R & Python Packages in One Application**  
🔗 **[Read the full guide](./bundling_r_and_python.md)**  

### **Overview**  
Managing **both R and Python packages** in a single application is challenging because **PyPI (Python Package Index) only supports Python**.  

This guide explains:  
✅ Why **PyPI is insufficient** for R & Python integration.  
✅ How **Conda** provides a **cross-platform** solution.  
✅ Best practices for **dependency management**, including:  
   - Using **conda-forge** instead of Anaconda’s default channel.  
   - Using **mamba** for faster environment setup.  
   - **Avoiding pip/conda conflicts** when installing packages.

✅ **4 installation methods**, from **fully reproducible Docker builds** to flexible manual setups.  

---

### 📊 **2. Choosing a Visualization Platform for a Scientific App**  
🔗 **[Read the full guide](./scientific_visualization_platforms.md)**  

### **Overview**  
Choosing the right **GUI framework or visualization platform** is crucial for scientific applications.  

This guide compares:  
✅ **PyQt/PySide** (Powerful, but complex for web apps).  
✅ **Dash** (Good for dashboards, but requires more setup).  
✅ **Streamlit** (Easy to use, great community, but can slow down large apps).  
✅ **Shiny (R & PyShiny)** (Ideal for reactive UI, but limited Python adoption).  
✅ **Jupyter Notebook** (Best for interactive analysis, but not a full GUI).  

The guide also covers:  
🔹 **Deployment strategies**: Web-based vs. local apps.  
🔹 **The growing role of WebAssembly (WASM)** in visualization tools.  
🔹 **Moorhen**: A WASM-based version of **Coot** for molecular modeling.  

---

### 🗂️ **3. Cross-Platform Directory Selection for Streamlit Apps**  
🔗 **[Read the full guide](./streamlit_directory_picker.md)**  

### **Overview**  
Selecting folders in a **cross-platform** way is **harder than expected**, especially in **Docker**.  

This guide explains:  
✅ **Native folder dialogs** for macOS (`AppleScript`), Linux (`Tkinter`), and Windows (`Win32 API`).  
✅ Why **Docker lacks a GUI** and why X11-based solutions don’t work well.  
✅ A **web-based alternative using Streamlit** for directory selection.  
✅ How to **optimize Streamlit's behavior** with `on_change` callbacks to prevent full-page reruns.  

---
