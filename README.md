**Azure Artifacts hands-on lab**. We’ll use a **Universal Package** because it’s the easiest package type for demonstrating publish, consume, versions, pipelines, delete, and recovery.

### Lab Details

```text
Organization : https://dev.azure.com/cloudnautic
Project      : project
Feed         : newfeed
Package      : cloudnautic-tools
Version      : 1.0.0
```

Azure Artifacts feeds can contain npm, NuGet, Maven, Python, Cargo, and Universal Packages. Microsoft also supports upstream sources for caching packages from public registries. ([Microsoft Learn][1])

## 1. Introduction to Azure Artifacts

```text
Azure DevOps
    |
    +-- Artifacts
         |
         +-- Feed: newfeed
                |
                +-- cloudnautic-tools
                        |
                        +-- 1.0.0
                        +-- 1.0.1
                        +-- 1.0.2
```

Azure Artifacts is used to:

```text
Store Packages
Share Packages
Version Packages
Control Access
Consume Packages
Publish Packages using Pipelines
```

---

## 2. Package Types

Azure Artifacts supports packages such as:

```text
NuGet        -> .NET
npm          -> Node.js
Maven        -> Java
Python       -> Python packages
Universal    -> Any files/scripts/binaries
Cargo        -> Rust
```

For training, use:

```text
Universal Package
```

because you can package normal files without building a language-specific library. ([Microsoft Learn][1])

---

# 3. Create Azure Artifacts Feed

Navigate:

```text
Azure DevOps
   ↓
Project
   ↓
Artifacts
   ↓
Create Feed
```

Enter:

```text
Name       : newfeed
Visibility : Members of project/organization
Scope      : Project
Upstream   : Enabled
```

Click:

```text
Create
```

A feed is the central repository where packages are stored. ([Microsoft Learn][1])

---

# 4. Private vs Public Feed

### Private

```text
Developer
    |
Authenticate
    |
Azure Artifacts
    |
Private Feed
```

Only authorized users can access packages.

### Public

```text
Internet User
     |
     ↓
Public Project
     |
Public Feed
```

Important: Microsoft is retiring Azure DevOps public projects. Existing public projects are scheduled to become private starting in **2027**, so public feeds should not be the basis of a new long-term public package distribution strategy. ([Microsoft Learn][1])

For training use:

```text
Private Project
+
Private Feed
```

---

# 5. Feed Permissions

Open:

```text
Artifacts
→ newfeed
→ Feed Settings ⚙️
→ Permissions
```

Main roles:

| Role                     | Permission                        |
| ------------------------ | --------------------------------- |
| Feed Reader              | Download packages                 |
| Feed and Upstream Reader | Download + save upstream packages |
| Feed Publisher           | Publish packages                  |
| Feed Owner               | Full management/delete            |

([Microsoft Learn][2])

For your training user:

```text
Atul
↓
Feed Publisher
```

For administration:

```text
Atul
↓
Feed Owner
```

---

# 6. Install Azure CLI

Check:

```bash
az --version
```

Login:

```bash
az login
```

Install/update Azure DevOps extension:

```bash
az extension add --name azure-devops
```

or:

```bash
az extension update --name azure-devops
```

Configure defaults:

```bash
az devops configure --defaults \
organization=https://dev.azure.com/cloudnautic \
project=project
```

Check:

```bash
az devops configure --list
```

The Universal Package commands are provided through the Azure DevOps CLI extension. ([Microsoft Learn][3])

---

# 7. Create Universal Package

Create folder:

```bash
mkdir cloudnautic-tools
cd cloudnautic-tools
```

Create files:

```bash
echo "Cloudnautic Azure Artifacts Demo" > README.txt
```

Create:

```bash
cat > app.py <<'EOF'
print("Hello from Azure Artifacts")
EOF
```

Create another file:

```bash
cat > config.json <<'EOF'
{
  "environment": "development",
  "application": "cloudnautic"
}
EOF
```

Check:

```bash
ls
```

Expected:

```text
README.txt
app.py
config.json
```

---

# 8. Publish Package to Azure Artifacts

Go one directory outside the package folder:

```bash
cd ..
```

Set variables:

```bash
ORG="https://dev.azure.com/cloudnautic"
PROJECT="project"
FEED="newfeed"
PACKAGE="cloudnautic-tools"
VERSION="1.0.0"
```

Publish:

```bash
az artifacts universal publish \
--organization "$ORG" \
--project "$PROJECT" \
--scope project \
--feed "$FEED" \
--name "$PACKAGE" \
--version "$VERSION" \
--path ./cloudnautic-tools \
--description "Cloudnautic training package"
```

Microsoft's CLI uses `az artifacts universal publish` with organization, project, feed, package name, version and package directory for project-scoped feeds. ([Microsoft Learn][4])

Verify:

```text
Azure DevOps
→ Artifacts
→ newfeed
→ cloudnautic-tools
→ 1.0.0
```

---

# 9. Consume Package from Feed

Create download folder:

```bash
mkdir downloaded-package
```

Download package:

```bash
az artifacts universal download \
--organization "$ORG" \
--project "$PROJECT" \
--scope project \
--feed "$FEED" \
--name "$PACKAGE" \
--version "1.0.0" \
--path ./downloaded-package
```

Check:

```bash
ls downloaded-package
```

Run:

```bash
python3 downloaded-package/app.py
```

Expected:

```text
Hello from Azure Artifacts
```

Universal Packages are downloaded using `az artifacts universal download`; Microsoft notes there isn't a direct Universal Package download API endpoint. ([Microsoft Learn][5])

---

# 10. Download Latest Package

Instead of specifying the exact version:

```bash
az artifacts universal download \
--organization "$ORG" \
--project "$PROJECT" \
--scope project \
--feed "$FEED" \
--name "$PACKAGE" \
--version "*" \
--path ./latest-package
```

`*` means:

```text
Latest available version
```

You can also use:

```text
1.*
1.2.*
```

to retrieve the latest matching version. ([Microsoft Learn][6])

---

# 11. Create New Package Version

Modify:

```bash
echo "Version 1.0.1" >> cloudnautic-tools/README.txt
```

Publish:

```bash
VERSION="1.0.1"
```

```bash
az artifacts universal publish \
--organization "$ORG" \
--project "$PROJECT" \
--scope project \
--feed "$FEED" \
--name "$PACKAGE" \
--version "$VERSION" \
--path ./cloudnautic-tools
```

Feed:

```text
cloudnautic-tools

1.0.0
1.0.1
```

---

# 12. Package Versioning

Recommended semantic version format:

```text
MAJOR.MINOR.PATCH
```

Example:

```text
1.0.0
│ │ │
│ │ └── Patch
│ └──── Minor
└────── Major
```

Examples:

```text
1.0.0 → First release
1.0.1 → Bug fix
1.1.0 → New feature
2.0.0 → Major/breaking change
```

---

# 13. Package Views

Azure Artifacts provides these default views:

```text
@Local
@Prerelease
@Release
```

A common lifecycle:

```text
Developer
   |
Publish
   ↓
@Local
   |
Testing
   ↓
@Prerelease
   |
Approved
   ↓
@Release
```

Packages are published to the base feed and appear in `@Local`; views are then used to expose selected validated versions. ([Microsoft Learn][7])

Promote using portal:

```text
Artifacts
→ Package
→ Version
→ Promote
→ @Prerelease
```

Then after validation:

```text
Promote
→ @Release
```

---

# 14. Upstream Sources

Architecture:

```text
Application
     |
     ↓
Azure Artifacts Feed
     |
     +---- Own Packages
     |
     +---- Upstream
             |
             +-- npmjs.com
             +-- NuGet.org
             +-- PyPI
             +-- Maven Central
```

Configure:

```text
Artifacts
→ newfeed
→ Feed Settings
→ Upstream Sources
→ Add Upstream
```

For example:

```text
Type   : Public source
Source : npmjs.com
```

or:

```text
NuGet.org
PyPI
Maven Central
```

When an authorized Collaborator or higher installs a package from an upstream source for the first time, Azure Artifacts can save a copy into the feed. ([Microsoft Learn][8])

---

# 15. Simple Azure Pipeline Project

Repository:

```text
AzureArtifacts/
│
├── package/
│   ├── app.py
│   ├── config.json
│   └── README.txt
│
└── azure-pipelines.yml
```

Example `app.py`:

```python
print("Hello from Azure Artifacts Pipeline")
```

`README.txt`:

```text
Cloudnautic Universal Package
```

---

# 16. Publish Package Using Azure Pipeline

Create:

```yaml
trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:

- task: CopyFiles@2
  displayName: Prepare Package
  inputs:
    SourceFolder: '$(Build.SourcesDirectory)/package'
    Contents: '**'
    TargetFolder: '$(Build.ArtifactStagingDirectory)'

- task: UniversalPackages@0
  displayName: Publish Universal Package
  inputs:
    command: publish
    publishDirectory: '$(Build.ArtifactStagingDirectory)'
    vstsFeedPublish: 'project/newfeed'
    vstsFeedPackagePublish: 'cloudnautic-tools'
    versionOption: custom
    versionPublish: '1.0.$(Build.BuildId)'
    packagePublishDescription: 'Published using Azure Pipeline'
```

Microsoft's pipeline task for Universal Packages is `UniversalPackages@0`. ([Microsoft Learn][9])

Flow:

```text
Git Push
   ↓
Azure Repo
   ↓
Azure Pipeline
   ↓
Copy Files
   ↓
UniversalPackages@0
   ↓
Azure Artifacts
   ↓
cloudnautic-tools
```

---

# 17. Important Pipeline Permission

If pipeline publishing fails with:

```text
403
Unauthorized
TF400813
```

check:

```text
Artifacts
→ newfeed
→ Feed Settings
→ Permissions
```

Give the project's build service:

```text
Feed Publisher (Contributor)
```

Typical identity:

```text
project Build Service (cloudnautic)
```

A Publisher can publish/promote packages, while a Collaborator cannot publish new packages. ([Microsoft Learn][2])

---

# 18. Pipeline to Consume Package

```yaml
trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:

- task: UniversalPackages@0
  displayName: Download Package
  inputs:
    command: download
    vstsFeed: 'project/newfeed'
    vstsFeedPackage: 'cloudnautic-tools'
    vstsPackageVersion: '1.0.0'
    downloadDirectory: '$(Build.SourcesDirectory)/downloaded'

- script: |
    ls -la downloaded
    python3 downloaded/app.py
  displayName: Test Package
```

Microsoft documents the same `UniversalPackages@0` task for package downloading. ([Microsoft Learn][9])

---

# 19. Delete Package

Portal:

```text
Artifacts
→ newfeed
→ cloudnautic-tools
→ Select Version
→ Delete
```

Example:

```text
cloudnautic-tools
    |
    ├── 1.0.0
    └── 1.0.1 ← Delete
```

Deleting/unpublishing package versions requires the appropriate higher feed permissions; the Feed Owner role has deletion rights. ([Microsoft Learn][2])

---

# 20. Recover Deleted Package

Navigate:

```text
Artifacts
→ newfeed
→ Recycle Bin
```

Select:

```text
cloudnautic-tools
```

then:

```text
Restore
```

Flow:

```text
Package
   |
 Delete
   ↓
Recycle Bin
   |
 Restore
   ↓
Feed
```

Azure Artifacts includes a Recycle Bin specifically as part of its package lifecycle model. ([Microsoft Learn][10])

---

# Complete Practice Flow

```text
1. Create Azure DevOps Project

2. Open Azure Artifacts

3. Create Feed
   newfeed

4. Configure Permissions

5. Enable Upstream Sources

6. Create Package
   cloudnautic-tools/

7. Create Files
   app.py
   config.json
   README.txt

8. Publish
   version 1.0.0

9. Verify Package

10. Download Package

11. Run Package

12. Modify Files

13. Publish
    version 1.0.1

14. Check Package Versions

15. Promote Version
    @Prerelease

16. Promote Version
    @Release

17. Create Azure Pipeline

18. Publish Package from Pipeline

19. Download Package from Pipeline

20. Delete Package

21. Open Recycle Bin

22. Recover Package
```

### Training sequence

```text
Azure Artifacts
      ↓
Package Types
      ↓
Feeds
      ↓
Private / Public
      ↓
Permissions
      ↓
Upstream Sources
      ↓
Create Package
      ↓
Publish Package
      ↓
Consume Package
      ↓
Pipeline Automation
      ↓
Versions
      ↓
Views
      ↓
Delete
      ↓
Recycle Bin
      ↓
Recover
```

This sequence is particularly suitable for a **60–90 minute live Azure Artifacts hands-on session**, because the same `cloudnautic-tools` Universal Package is reused from creation through recovery instead of switching between unrelated examples.

[1]: https://learn.microsoft.com/en-us/azure/devops/artifacts/concepts/feeds?view=azure-devops&utm_source=chatgpt.com "What are Azure Artifacts feeds? - Azure Artifacts | Microsoft Learn"
[2]: https://learn.microsoft.com/en-us/azure/devops/artifacts/feeds/feed-permissions?view=azure-devops&utm_source=chatgpt.com "Manage permissions - Azure Artifacts | Microsoft Learn"
[3]: https://learn.microsoft.com/en-us/cli/azure/artifacts/universal?view=azure-cli-latest&utm_source=chatgpt.com "az artifacts universal | Microsoft Learn"
[4]: https://learn.microsoft.com/da-dk/azure/devops/artifacts/quickstarts/universal-packages?view=azure-devops&utm_source=chatgpt.com "Publish Universal Packages in Azure Artifacts - Azure Artifacts | Microsoft Learn"
[5]: https://learn.microsoft.com/en-us/azure/devops/artifacts/quickstarts/download-universal-packages?view=azure-devops&utm_source=chatgpt.com "Download Universal Packages in Azure Artifacts - Azure Artifacts | Microsoft Learn"
[6]: https://learn.microsoft.com/bs-latn-ba/azure/DevOps/artifacts/quickstarts/download-universal-packages?view=azure-devops-2022&utm_source=chatgpt.com "Download Universal Packages in Azure Artifacts - Azure Artifacts | Microsoft Learn"
[7]: https://learn.microsoft.com/en-us/azure/devops/artifacts/concepts/views?view=azure-devops&utm_source=chatgpt.com "What are feed views? - Azure Artifacts | Microsoft Learn"
[8]: https://learn.microsoft.com/en-us/azure/devops/artifacts/how-to/set-up-upstream-sources?view=azure-devops&utm_source=chatgpt.com "Set up upstream sources for your feed - Azure Artifacts | Microsoft Learn"
[9]: https://learn.microsoft.com/en-us/azure/devops/pipelines/artifacts/universal-packages?view=azure-devops&utm_source=chatgpt.com "Publish & download Universal Packages - Azure Pipelines | Microsoft Learn"
[10]: https://learn.microsoft.com/en-us/azure/devops/artifacts/artifacts-key-concepts?view=azure-devops&utm_source=chatgpt.com "Azure Artifacts key concepts - Azure Artifacts | Microsoft Learn"
