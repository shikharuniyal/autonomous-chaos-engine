# DARWIN Installation Guide for Windows

## Quick Start (Automated)

Open **PowerShell as Administrator** and run:

```powershell
cd e:\#EditorCodes\banglore_Hackthon
.\setup\install_windows.ps1
```

Then verify:
```powershell
.\setup\verify_installation.ps1
```

---

## Manual Installation (Step-by-Step)

### 1. Docker Desktop (Already Installed ✓)

You already have Docker 29.1.3 installed!

Verify: `docker --version`

---

### 2. Minikube

**Download:**
```powershell
# Create tools directory
mkdir C:\darwin-tools
cd C:\darwin-tools

# Download minikube
Invoke-WebRequest -Uri "https://storage.googleapis.com/minikube/releases/latest/minikube-windows-amd64.exe" -OutFile "minikube.exe" -UseBasicParsing
```

**Add to PATH:**
```powershell
$env:Path += ";C:\darwin-tools"
[Environment]::SetEnvironmentVariable("Path", $env:Path, "User")
```

**Verify:**
```powershell
minikube version
```

---

### 3. Kubectl

**Download:**
```powershell
cd C:\darwin-tools
Invoke-WebRequest -Uri "https://dl.k8s.io/release/v1.28.0/bin/windows/amd64/kubectl.exe" -OutFile "kubectl.exe" -UseBasicParsing
```

**Verify:**
```powershell
kubectl version --client
```

---

### 4. Helm

**Download:**
```powershell
cd C:\darwin-tools
Invoke-WebRequest -Uri "https://get.helm.sh/helm-v3.13.0-windows-amd64.zip" -OutFile "helm.zip" -UseBasicParsing

# Extract
Expand-Archive -Path helm.zip -DestinationPath . -Force
Move-Item .\windows-amd64\helm.exe . -Force
Remove-Item windows-amd64 -Recurse -Force
Remove-Item helm.zip -Force
```

**Verify:**
```powershell
helm version
```

---

### 5. Go 1.22

**Download & Install:**

1. Download: https://go.dev/dl/go1.22.0.windows-amd64.msi
2. Run the installer
3. Restart PowerShell

**Verify:**
```powershell
go version
```

---

### 6. Node.js 20 LTS

**Download & Install:**

1. Download: https://nodejs.org/dist/v20.11.0/node-v20.11.0-x64.msi
2. Run the installer (includes NPM)
3. Restart PowerShell

**Verify:**
```powershell
node --version
npm --version
```

---

### 7. Python 3.11+ (Already Installed ✓)

You already have Python 3.12.12 installed!

**Verify:**
```powershell
python --version
```

**Install required packages:**
```powershell
pip install kubernetes scikit-learn torch psycopg2-binary neo4j redis nats-py prometheus-client pyyaml joblib
```

---

## Alternative: Using Chocolatey

If you have Chocolatey installed, run:

```powershell
choco install minikube kubernetes-cli kubernetes-helm golang nodejs -y
```

To install Chocolatey first:
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

---

## Post-Installation: Start Kubernetes

### 1. Start Minikube

```powershell
minikube start `
  --cpus=4 `
  --memory=8192 `
  --disk-size=40gb `
  --driver=docker `
  --kubernetes-version=v1.28.0
```

**Wait 3-5 minutes** for cluster to initialize.

**Verify:**
```powershell
kubectl get nodes
# Should show: minikube   Ready    control-plane   <time>   v1.28.0
```

### 2. Enable Minikube Addons

```powershell
minikube addons enable metrics-server
minikube addons enable ingress
```

### 3. Install Istio

```powershell
# Download Istio
cd C:\darwin-tools
Invoke-WebRequest -Uri "https://github.com/istio/istio/releases/download/1.20.0/istio-1.20.0-win.zip" -OutFile "istio.zip" -UseBasicParsing
Expand-Archive -Path istio.zip -DestinationPath . -Force

# Add to PATH
$env:Path += ";C:\darwin-tools\istio-1.20.0\bin"

# Install Istio
cd e:\#EditorCodes\banglore_Hackthon
.\setup\install_istio.ps1
```

### 4. Install Databases (Helm Charts)

```powershell
.\setup\install_databases.ps1
```

---

## Verification Checklist

Run this to verify everything:

```powershell
cd e:\#EditorCodes\banglore_Hackthon
.\setup\verify_installation.ps1
```

Expected output:
```
Checking Docker... ✓ Docker version 29.1.3
Checking Minikube... ✓ minikube version: v1.32.0
Checking Kubectl... ✓ Client Version: v1.28.0
Checking Helm... ✓ v3.13.0
Checking Python... ✓ Python 3.12.12
Checking Go... ✓ go version go1.22.0 windows/amd64
Checking Node.js... ✓ v20.11.0
Checking NPM... ✓ v10.2.4

=== All tools installed successfully! ===
```

---

## Troubleshooting

### Minikube won't start

```powershell
# Delete old cluster
minikube delete

# Restart
minikube start --cpus=4 --memory=8192 --driver=docker
```

### PATH not updating

Restart PowerShell or terminal after installation.

### Docker not working

1. Start Docker Desktop
2. Verify: `docker ps`

### Permission denied

Run PowerShell **as Administrator**

---

## Next Steps

After all tools are installed:

1. **Clone or create project structure:**
   ```powershell
   cd e:\#EditorCodes\banglore_Hackthon
   ```

2. **Install Istio:**
   ```powershell
   .\setup\install_istio.ps1
   ```

3. **Install databases:**
   ```powershell
   .\setup\install_databases.ps1
   ```

4. **Build microservices:**
   ```powershell
   .\scripts\build_services.ps1
   ```

5. **Deploy DARWIN:**
   ```powershell
   .\scripts\deploy.ps1
   ```

---

## Resource Requirements

Ensure your machine has:

- **CPU:** 4+ cores
- **RAM:** 16GB total (8GB for minikube)
- **Disk:** 40GB+ free space
- **OS:** Windows 10/11

---

## Quick Reference Commands

```powershell
# Start cluster
minikube start

# Stop cluster
minikube stop

# Delete cluster
minikube delete

# Check cluster status
kubectl get nodes
kubectl get pods -A

# View Kubernetes dashboard
minikube dashboard

# Access services
minikube service <service-name> -n darwin

# View logs
kubectl logs <pod-name> -n darwin

# Port forward
kubectl port-forward svc/<service-name> 8080:8080 -n darwin
```

---

**You're now ready to build DARWIN!** 🚀
