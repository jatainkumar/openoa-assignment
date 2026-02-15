# Jatain's Guide to OpenOA

## 1. Project Overview
**OpenOA** (Open Operational Assessment) is an open-source Python framework designed for assessing the performance of wind energy plants. It provides standardized methods for importing time-series data from wind plants (SCADA, meter data, etc.) and performing operational analysis (OA).

In essence, this project helps wind energy professionals answer questions like:
- "How much energy is my wind farm actually producing vs. what was expected?"
- "How much energy am I losing due to wakes, electrical inefficiencies, or turbine downtime?"

## 2. Architecture: "The Frontend & Backend"

While OpenOA is a Python library, we can think of its architecture in terms of a Backend (the logic) and a Frontend (the interface).

### The Backend: `openoa` Core Library
The "Backend" of this project is the `openoa` Python package located in the `openoa/` directory. This is the engine that powers all calculations.

*   **PlantData**: The central data structure (built on Pandas) that unifies SCADA, meter, and meteorological data into a standard format.
*   **Analysis Methodologies**:
    *   **MonteCarloAEP**: Calculates long-term Annual Energy Production.
    *   **WakeLosses**: Estimates energy lost due to turbine wakes.
    *   **ElectricalLosses**: Calculates transmission and collection system losses.
    *   **TurbineLongTermGrossEnergy**: Ideal energy if turbines were perfect.
*   **Utility Modules**: Tools for data cleaning, imputing missing values, and filtering outliers.

### The Frontend: Jupyter Notebooks (`examples/`)
The "Frontend" is the interactive layer where you visualize data and run analyses. This is primarily handled through **Jupyter Notebooks** located in the `examples/` directory.

*   **Visualization**: Plots for power curves, wind roses, and time-series data.
*   **Interactive Analysis**: Notebooks like `02a_plant_aep_analysis.ipynb` guide you step-by-step through an assessment.
*   **Custom Scripts**: You can build your own dashboards (using libraries like Streamlit or maintain these notebooks) to present findings.

---

## 3. How to Run (Local Development)

Follow these steps to get the project running on your local machine (Windows/Mac/Linux).

### Prerequisites
- **Anaconda or Miniconda** (Recommended for managing Python environments).
- **Git** (to clone the repo).

### Step 1: Clone and Environment Setup
Open your terminal (PowerShell or Bash) and run:

```bash
# 1. Create a fresh environment (Python 3.10 is stable)
conda create --name openoa-env python=3.10
conda activate openoa-env

# 2. Install the package in editable mode with all extras
# This installs dependencies for docs, examples, and development.
pip install -e ".[develop,docs,examples]"

# 3. Install pre-commit hooks (if you plan to contribute code)
pre-commit install
```

### Step 2: Unzip Example Data
The project comes with sample wind farm data (La Haute Borne). You need to unzip it to run the examples.

```bash
# Windows (PowerShell)
Expand-Archive examples/data/la_haute_borne.zip -DestinationPath examples/data/la_haute_borne

# Linux/Mac
unzip examples/data/la_haute_borne.zip -d examples/data/la_haute_borne/
```

### Step 3: Launch the "Frontend"
Start the Jupyter Lab server to interact with the project:

```bash
jupyter lab
```
This will open your web browser. Navigate to the `examples/` folder and open **`02a_plant_aep_analysis.ipynb`** to start your first analysis.

---

## 4. How to Deploy on Servers

To make OpenOA available to a team (hosting the "Frontend" and "Backend" on a server), you typically deploy it as a **JupyterHub** instance or a simpler **Remote Jupyter Server**.

### Option A: Simple Remote Server (For small teams/individuals)
This method runs Jupyter on a Linux server (e.g., AWS EC2, Azure VM) and lets you access it from your local browser.

1.  **Provision a Server**: Ubuntu 22.04 LTS is recommended.
2.  **SSH into Server**: `ssh user@your-server-ip`
3.  **Install Environment**: Follow "Step 1" above on the server.
4.  **Start Jupyter remotely**:
    ```bash
    jupyter lab --no-browser --port=8888 --ip=0.0.0.0
    ```
5.  **Access it**:
    *   Open your local browser to `http://your-server-ip:8888`.
    *   Enter the token generated in the terminal.

### Option B: Enterprise Deployment (Docker & JupyterHub)
For a robust "Production" deployment where multiple users have their own workspaces.

1.  **Dockerize the Application**:
    Create a `Dockerfile` to package OpenOA:
    ```dockerfile
    FROM jupyter/scipy-notebook:python-3.10
    
    USER root
    # Install system dependencies if needed
    RUN apt-get update && apt-get install -y git
    
    USER jovyan
    # Clone and install OpenOA
    RUN git clone https://github.com/NREL/OpenOA.git /home/jovyan/OpenOA
    WORKDIR /home/jovyan/OpenOA
    RUN pip install ".[examples]"
    ```

2.  **Build and Run**:
    ```bash
    docker build -t openoa-server .
    docker run -p 8888:8888 openoa-server
    ```

3.  **Cloud Deployment**:
    *   Deploy this Docker container to **AWS ECS**, **Google Cloud Run**, or **Azure Container Instances**.
    *   Map port 8888 to the internet (securely).

## 5. Key Commands Reference

| Command | Description |
| :--- | :--- |
| `pip install .` | Installs the library locally. |
| `pytest` | Runs the backend unit tests to ensure calculation accuracy. |
| `jupyter lab` | Starts the interactive frontend. |
| `pip install ".[reanalysis]"` | Installs extra features for downloading NASA MERRA2/ERA5 data. |

