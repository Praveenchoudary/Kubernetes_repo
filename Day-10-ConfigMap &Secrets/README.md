---

## 📘 What is a ConfigMap?

A **ConfigMap** stores non-sensitive config data in key-value pairs. You can inject these into pods as environment variables or mounted files.

### ✅ Use Cases

- Setting environment modes: `ENV=production`
- App configs: `LOG_LEVEL=debug`
- Feature toggles: `ENABLE_FEATURE=true`

  ![image](https://github.com/user-attachments/assets/e3708ea9-192d-4b63-bd1d-d753748204e7)


---

## 🔐 What is a Secret?

A **Secret** stores sensitive data such as passwords, tokens, or keys. Data is base64-encoded and optionally encrypted at rest.

### ✅ Use Cases

- Database credentials
- TLS keys and certificates
- API tokens

  ![image](https://github.com/user-attachments/assets/b9c9d7dc-b419-4593-86d9-95af44aa2b19)


---
