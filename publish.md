Yes. For your **Universal Package** (`package/`) you can publish directly from your Mac CLI to the Azure Artifacts feed `newfeed`, without running the pipeline.

### 1. Go to project

```bash
cd /Users/atul/Downloads/azure-artifacts

ls package
```

You should see:

```text
app.py
config.json
README.txt
```

### 2. Login to Azure DevOps

If Azure CLI is already installed:

```bash
az login
```

Install the Azure DevOps extension if needed:

```bash
az extension add --name azure-devops
```

Check:

```bash
az devops --version
```

### 3. Configure your Azure DevOps organization/project

```bash
az devops configure --defaults \
organization=https://dev.azure.com/cloudnautic \
project=project
```

Verify:

```bash
az devops configure --list
```

### 4. Login to Azure DevOps with PAT

Create a PAT in Azure DevOps with appropriate **Packaging read/write** permission, then:

```bash
az devops login --organization https://dev.azure.com/cloudnautic
```

It will ask:

```text
Token:
```

Paste your PAT and press Enter.

### 5. Publish `package/` to `newfeed`

Start with a new version, for example `1.0.10`:

```bash
az artifacts universal publish \
  --organization https://dev.azure.com/cloudnautic \
  --project project \
  --scope project \
  --feed newfeed \
  --name cloudnautic-tools \
  --version 1.0.10 \
  --description "Published manually from Mac CLI" \
  --path ./package
```

Expected flow:

```text
package/
├── app.py
├── config.json
└── README.txt
        │
        ▼
az artifacts universal publish
        │
        ▼
Azure DevOps
        │
        ▼
project
        │
        ▼
newfeed
        │
        ▼
cloudnautic-tools
Version: 1.0.10
```

### 6. Verify in Azure Artifacts

Open your Azure DevOps project → **Artifacts** → `newfeed` → `cloudnautic-tools`.

You should see version:

```text
cloudnautic-tools
└── 1.0.10
```

### 7. Download it again for testing

```bash
mkdir downloaded-package

az artifacts universal download \
  --organization https://dev.azure.com/cloudnautic \
  --project project \
  --scope project \
  --feed newfeed \
  --name cloudnautic-tools \
  --version 1.0.10 \
  --path ./downloaded-package
```

Then:

```bash
ls downloaded-package
python3 downloaded-package/app.py
```

Expected:

```text
Hello from Azure Artifacts Pipeline
```

**Important:** Universal Package versions are immutable. If `1.0.10` already exists, use a new version such as `1.0.11`.
