# 🚀 NuGet + Artifactory CI/CD with Bamboo

This document describes how to configure and use an **Artifactory service account** for publishing NuGet packages via Bamboo.  
Service accounts provide **non-expiring API keys**, ensuring stable automation, security, and compliance.

---

## 🔐 Why Service Accounts Don’t Require Rotation

Service accounts are ideal for CI/CD pipelines because:

- 🔑 **Non-expiring API keys** eliminate pipeline failures due to credential expiry  
- ⚙️ **Stable automation** ensures uninterrupted builds  
- 🔒 **Scoped permissions** restrict access to specific repositories (`nuget-dev`, `nuget-staging`, `nuget-release`)  
- 📊 **Audit trail** logs every action for governance and compliance  

---

## ⚙️ NuGet Configuration and Complete CI/CD Flow

Below is the NuGet configuration used by Bamboo to authenticate with Artifactory, followed by the **end-to-end pipeline execution flow**.

### 📄 NuGet Configuration

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="Artifactory-NuGet" value="https://artifactory.company.com/artifactory/api/nuget/nuget-dev" />
  </packageSources>
  <packageSourceCredentials>
    <Artifactory-NuGet>
      <add key="Username" value="bamboo-ci-service" />
      <add key="ClearTextPassword" value="SERVICE_ACCOUNT_API_KEY" />
    </Artifactory-NuGet>
  </packageSourceCredentials>
</configuration>
```

---

### 🔄 Pipeline Execution Flow

The Bamboo pipeline executes the following steps in sequence:

---

#### 📥 Restore Dependencies

Restore all NuGet dependencies from Artifactory:

```bash
nuget restore MySolution.sln -ConfigFile NuGet.Config
```

---

#### 📦 Build and Package

Create a versioned NuGet package using Bamboo build number:

```bash
nuget pack MyProject.nuspec -OutputDirectory ./artifacts \
  -Properties BuildNumber=${bamboo.buildNumber}
```

📌 Generated artifact:
```
MyProject.1.0.${bamboo.buildNumber}.nupkg
```

---

#### 🔏 Digital Signing

Ensure package integrity and authenticity:

```bash
nuget sign ./artifacts/MyProject.1.0.${bamboo.buildNumber}.nupkg \
  -CertificatePath mycert.pfx -CertificatePassword $CERT_PASS
```

---

#### 🚀 Publish to Artifactory

Push the signed package to Artifactory:

```bash
nuget push ./artifacts/MyProject.1.0.${bamboo.buildNumber}.nupkg \
  -Source Artifactory-NuGet -ApiKey SERVICE_ACCOUNT_API_KEY
```

---

#### 📜 Generate SBOM (Software Bill of Materials)

Create SBOM in CycloneDX format for security and compliance:

```bash
dotnet sbom generate --format cyclonedx --output sbom.json
```

---

#### 📤 Upload SBOM

Attach SBOM to the artifact in Artifactory:

```bash
jfrog rt upload sbom.json nuget-dev/MyProject/1.0.${bamboo.buildNumber}/
```

---

#### 📊 Capture Build Metadata

Enable traceability and auditability:

```bash
jfrog rt bce MyProject ${bamboo.buildNumber}
jfrog rt bdi MyProject ${bamboo.buildNumber}
jfrog rt bp MyProject ${bamboo.buildNumber}
```

---

#### 🔁 Promote Artifact (Dev → Staging)

```bash
jfrog rt bpr MyProject ${bamboo.buildNumber} nuget-dev nuget-staging
```

---

#### 🏁 Promote Artifact (Staging → Production)

```bash
jfrog rt bpr MyProject ${bamboo.buildNumber} nuget-staging nuget-release
```

---

## 🏛 Governance and Compliance

Artifactory enforces enterprise-grade governance:

- ✅ **Policy enforcement** (via Xray) ensures only secure artifacts are promoted  
- 🧾 **Audit logs** track all actions under the service account  
- 🔍 **Traceability** links builds, dependencies, and deployments  

---

## 💡 Best Practices

- Use **service accounts** instead of personal credentials  
- Store `NuGet.Config` in **version control**  
- Maintain **immutable versioning** (unique version per build)  
- Use **promotion workflows** instead of rebuilding  
- Always attach **SBOMs and digital signatures**  
- Apply **least privilege access control**  

---

## 📌 Summary

This setup enables:

- 🔐 Secure authentication using service accounts  
- ⚙️ Fully automated CI/CD with Bamboo  
- 📜 Compliance via SBOM generation and digital signing  
- 📦 Immutable artifacts with controlled promotion  
- 🔍 End-to-end traceability and auditability  

---
