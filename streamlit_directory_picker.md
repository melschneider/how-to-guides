# 🚀 Implementing a Cross-Platform Directory Picker for a Locally Run Streamlit App  

## **Introduction**  
When developing a **Streamlit app** that runs **locally**, I needed a way to let users **select directories** where input and output files would be stored.  

At first, this seemed like a straightforward task—after all, file dialogs are a standard part of operating systems. However, I soon realized that achieving a **seamless and user-friendly directory selection** across **macOS, Windows, Linux, and Docker** was far from simple.  

### **🔹 The Goal**
- Provide a **smooth** directory selection experience on **all platforms**.  
- Ensure the **app remains locally run** (no external services required).  
- Minimize **user effort** while selecting directories.  
- Avoid **unnecessary page reloads** in **Streamlit**.  

---

## **🛠️ Initial Approach: Native File Dialogs for Each OS**  

The most intuitive way to select a directory is to use the **native file dialogs** provided by each operating system.  

### ✅ **macOS (AppleScript-Based Picker)**  

```python
import subprocess

def select_folder_mac():
    script = """
    tell application "System Events"
        activate
        set folderPath to choose folder with prompt "Please select a folder"
        set folderPath to POSIX path of folderPath
    end tell
    return folderPath
    """
    try:
        return subprocess.check_output(["osascript", "-e", script]).decode("utf-8").strip()
    except subprocess.CalledProcessError:
        return None
```

---

### ✅ **Linux (Tkinter-Based Picker)**  

```python
import tkinter as tk
from tkinter import filedialog

def select_folder_linux():
    root = tk.Tk()
    root.withdraw()
    root.wm_attributes('-topmost', 1)
    folder = filedialog.askdirectory()
    root.destroy()
    return folder
```

---

### ✅ **Windows (Win32 API-Based Picker)**  

```python
import ctypes
from ctypes import wintypes
from win32com.shell import shell, shellcon

def select_folder_windows():
    pidl, display_name, image_list = shell.SHBrowseForFolder(
        None, None, "Select a folder", shellcon.BIF_RETURNONLYFSDIRS
    )
    if pidl:
        return shell.SHGetPathFromIDList(pidl)
    return None
```
💡 **If PyWin32 is not available, Tkinter is used as a fallback.**  

---

## **🚧 The Problem: Running in Docker**  

While the above approaches worked **flawlessly** for macOS, Windows, and Linux, I encountered **major roadblocks** when running the **Streamlit app inside a Docker container**.  

💡 **Why?**  
- **Docker does not have a built-in GUI.**  
- **Tkinter requires an active display server (X11), which is missing in Docker.**  
- **Workarounds like Xvfb were too complicated for users.**  

At first, I attempted to use **Xvfb (X Virtual Framebuffer)** to simulate a display server:  

```dockerfile
RUN apt-get update && apt-get install -y xvfb
```

🔴 **Failed:** GUI applications **did not display properly** without additional permissions.  

Then, I tried **manually setting the DISPLAY environment variable**:  

```sh
export DISPLAY=:99
```

🔴 **Failed:** The user still had to **manually provide access to the X-server via X11**, which was **too much to ask** for a casual user.  

---

## **✅ Final Solution: A Web-Based Directory Picker for Docker**  

Since all GUI-based solutions required **too much manual configuration**, I decided that the **only viable option** for Docker was a **web-based directory picker** built directly into Streamlit.  

### **🚀 Platform Detection: Choosing the Best Picker for Each OS**  

```python
import streamlit as st
import os
import sys

def is_running_in_docker():
    return os.path.exists("/.dockerenv")

def select_folder():
    """Select a folder based on the OS or Docker environment."""
    if is_running_in_docker():
        return select_folder_browser()
    elif sys.platform == "darwin":
        return select_folder_mac()
    elif sys.platform.startswith("linux"):
        return select_folder_linux()
    elif sys.platform.startswith("win"):
        return select_folder_windows()
    else:
        st.warning("Unsupported OS")
        return None
```

---

## **🚀 Implementing a Browser-Based Folder Picker for Docker**  

```python
def select_folder_browser():
    """Web-based folder picker for Docker environments."""
    st.markdown("### 📁 Select a Folder")
    base_path = "/app/data"

    directories = [d for d in os.listdir(base_path) if os.path.isdir(os.path.join(base_path, d))]
    
    selected_dir = st.selectbox("Available Directories:", directories)

    if st.button("✅ Confirm Selection"):
        st.session_state["selected_folder"] = os.path.join(base_path, selected_dir)
        st.success(f"✅ Selected directory: `{st.session_state['selected_folder']}`")
```

---

## **🚀 Streamlit-Specific Optimizations**  

Since **Streamlit reruns the script** on every interaction, I needed to carefully manage **navigation and directory creation**:  

### **✅ Using `on_change` Callbacks for Navigation**
Instead of forcing a **full page reload**, I used `on_change` callbacks to **update only the directory picker UI**:  

```python
def update_navigation(folder_key):
    selected_dir = st.session_state[f"dir_select_{folder_key}"]
    current_path = st.session_state[f"current_path_{folder_key}"]
    new_path = os.path.join(current_path, selected_dir) if selected_dir != ".." else os.path.dirname(current_path)
    
    if os.path.isdir(new_path):
        st.session_state[f"current_path_{folder_key}"] = new_path
```

```python
st.selectbox("Navigate Directories:", [".."] + directories, key=f"dir_select_{folder_key}", on_change=update_navigation, args=(folder_key,))
```

### **❌ The Downside of `st.rerun()`**
Alternatively, **`st.rerun()`** can be used to **force a full rerun**:  
```python
st.rerun()
```
However, this **reloads everything**, causing **flickering** and **resetting UI elements**.  
✅ Using **callbacks instead** results in a **smoother experience**.  

---

## **📌 Conclusion**  
By detecting the **running environment** and selecting the most **user-friendly solution**, I was able to:  

✅ **Use native dialogs** on macOS, Windows, and Linux.  
✅ **Implement a web-based picker** for Docker.  
✅ **Optimize Streamlit behavior** to ensure **smooth navigation**.  

---

## **💾 Full Code Repository**
📌 Check out the complete implementation on **GitHub**:  
👉 **[GitHub Repo - EnsembleFlex](https://github.com/chembl/EnsembleFlex)**  

---

**I hope this post provides insights and help for your app development** 😊  
