# encryption-and-decryption

Below is a **clean, professional, GitHub-ready README** version of your content, rewritten with **proper headings, sub-headings, architecture explanation, best practices, and missing DevOps points added**.


---

# 🔐 Client-Side Encryption Using AWS KMS, AWS CLI & OpenSSL

## 📌 Overview

This project demonstrates **client-side encryption outside AWS** using **AWS KMS**, **AWS CLI**, and **OpenSSL**.
Sensitive data is encrypted locally before being stored or transmitted, ensuring AWS never sees plaintext data.

In this guide, we cover:

* Creating a **Customer Managed Key (CMK)**
* Generating **data keys using AWS KMS**
* Encrypting data using **OpenSSL**
* Securely decrypting data using **KMS + OpenSSL**

> 🔔 **Note**: In a follow-up guide, we will cover the **AWS Encryption SDK** with hands-on examples.

---

## 🧱 High-Level Architecture

```
+-------------------+         +----------------------+
|   Client Machine  |         |      AWS KMS         |
|                   |         |                      |
|  OpenSSL          |<------->|  Customer Managed    |
|  AWS CLI          |  API     |  Key (CMK)           |
|                   |         |                      |
+---------+---------+         +----------+-----------+
          |                              |
          | Encrypt / Decrypt            |
          |                              |
+---------v---------+
| Encrypted Data    |
| + Encrypted Data  |
|   Key             |
+-------------------+
```

### 🔍 Key Points

* **Plaintext data never leaves the client**
* AWS KMS is only used to **protect data keys**
* Suitable for **compliance-heavy environments**

---

## 🛠️ Prerequisites

* AWS Account
* AWS CLI installed and configured
* OpenSSL installed
* IAM permissions:

  * `kms:CreateKey`
  * `kms:GenerateDataKey`
  * `kms:Decrypt`

---

## 🔑 Step 01 — Create a Customer Managed Key (CMK)

### Steps (AWS Console)

1. Login to AWS Console
2. Navigate to **AWS KMS**
3. Select region **us-east-1 (N. Virginia)**
4. Click **Create key**
5. Select **Symmetric encryption**
6. Assign an **alias** (example: `alias/youtube`)
7. Configure:

   * Key administrators
   * Key usage permissions
8. Review and create the CMK

### 💡 Why Symmetric CMK?

* Faster and cost-effective
* Recommended by AWS for most encryption use cases

---

## 🔐 Step 02 — Generate Data Keys Using AWS CLI

CMKs can encrypt data **up to 4 KB only**.
For larger data, AWS KMS generates **data keys**.

### Command

```bash
aws kms generate-data-key \
  --key-id alias/youtube \
  --key-spec AES_256 \
  --region us-east-1
```

### Sample Response (Mock)

```json
{
  "KeyId": "arn:aws:kms:us-east-1:123456789:key/xxxx",
  "Plaintext": "BASE64_PLAINTEXT_KEY",
  "CiphertextBlob": "BASE64_ENCRYPTED_KEY"
}
```

---

##️## 💾 Store Data Keys Securely

### Decode and save keys

```bash
echo "BASE64_PLAINTEXT_KEY" | base64 --decode > datakey
echo "BASE64_ENCRYPTED_KEY" | base64 --decode > encrypted-datakey
```

⚠️ **Important**

* `datakey` → Used only temporarily
* `encrypted-datakey` → Safe to store alongside encrypted data

---

## 🔒 Step 03 — Encrypt Data Using OpenSSL

### Create sensitive data

```bash
echo "My database password" > passwords.txt
```

### Encrypt using OpenSSL

```bash
openssl enc \
  -in passwords.txt \
  -out passwords-encrypted.txt \
  -e -aes256 \
  -k fileb://./datakey
```

### Immediately delete plaintext key

```bash
rm datakey
```

✔ Prevents accidental data exposure
✔ Follows security best practices

---

## 🔓 Step 04 — Decrypt Data Using AWS KMS

### Request plaintext data key from KMS

```bash
aws kms decrypt \
  --ciphertext-blob fileb://./encrypted-datakey \
  --region us-east-1
```

### Sample Output

```json
{
  "Plaintext": "BASE64_PLAINTEXT_KEY",
  "KeyId": "arn:aws:kms:us-east-1:123456789:key/xxxx"
}
```

---

## 🔁 Restore Plaintext Data Key

```bash
echo "BASE64_PLAINTEXT_KEY" | base64 --decode > datakey
```

---

## 🔓 Decrypt the Data

```bash
openssl enc \
  -in passwords-encrypted.txt \
  -out passwords-decrypted.txt \
  -d -aes256 \
  -k fileb://./datakey
```

🎉 The original plaintext data is now restored securely.

---

## 🔐 Security Best Practices (Added)

* ❌ Never store plaintext data keys
* ✅ Use IAM least-privilege access
* ✅ Rotate CMKs periodically
* ✅ Enable CloudTrail logging for KMS
* ✅ Store encrypted data keys alongside encrypted data
* ✅ Use AWS Encryption SDK for production workloads

---

## 📦 When to Use This Approach?

| Use Case                | Recommended           |
| ----------------------- | --------------------- |
| Client-side encryption  | ✅ Yes                 |
| Compliance (PCI, HIPAA) | ✅ Yes                 |
| Large data payloads     | ✅ Yes                 |
| Simple app secrets      | ❌ Use Secrets Manager |

---

## 🚀 Future Enhancements

* Replace OpenSSL with **AWS Encryption SDK**
* Automate encryption using CI/CD pipelines
* Integrate with AWS Secrets Manager
* Add envelope encryption examples

---

## 📚 References

* AWS KMS Documentation
* OpenSSL Documentation
* AWS Encryption SDK

---

## ✅ Conclusion

This project demonstrates a **secure envelope encryption pattern** using AWS KMS and OpenSSL, ensuring:

* Full control over encryption keys
* Zero plaintext exposure to AWS
* Compliance-ready data protection

---

Below is a **GitHub-ready, professional README section** for **AWS Encryption SDK**, aligned with your previous KMS + OpenSSL README.
It includes **clear headings, sub-headings, architecture diagrams, best practices, and DevOps context**.

You can **paste this directly into your README.md** under a new section like
👉 `## AWS Encryption SDK – Client-Side Encryption`

---

# 🔐 AWS Encryption SDK – Client-Side Encryption

## 📌 What is AWS Encryption SDK?

The **AWS Encryption SDK** is a **client-side encryption library** that simplifies encrypting and decrypting data using **industry-standard cryptography and AWS best practices**.

Instead of manually handling:

* Encryption algorithms
* Data key lifecycle
* Integrity verification

…the AWS Encryption SDK **handles everything for you**.

> 💡 You only provide a **master key (KMS CMK)** — the SDK automatically performs **envelope encryption**.

---

## 🧱 High-Level Architecture (AWS Encryption SDK)

![Image](https://docs.aws.amazon.com/images/encryption-sdk/latest/developer-guide/images/encrypt-workflow-with-cache.png?utm_source=chatgpt.com)

![Image](https://docs.aws.amazon.com/images/encryption-sdk/latest/developer-guide/images/envelope-encryption-70.png?utm_source=chatgpt.com)

![Image](https://docs.aws.amazon.com/images/encryption-sdk/latest/developer-guide/images/keyring-decrypt.png?utm_source=chatgpt.com)

### 🔍 Architecture Explanation

```
Client Application
 ├─ AWS Encryption SDK
 │   ├─ KMS Keyring
 │   ├─ Data Key Generation
 │   ├─ Integrity Validation
 │
 ├─ Encrypted Payload
 └─ Encryption Context
        │
        ▼
     AWS KMS (CMK)
```

### Key Highlights

* Plaintext data **never leaves the client**
* SDK automatically:

  * Generates data keys
  * Encrypts data keys with KMS CMK
  * Validates data integrity
* Ideal for **distributed systems & microservices**

---

## 🌍 Supported Languages

AWS Encryption SDK currently supports:

* JavaScript (Node.js & Browser)
* Java
* Python
* C

👉 In this example, **Node.js (JavaScript SDK)** is used.

---

## ❓ When to Use AWS Encryption SDK?

### ✅ Use AWS Encryption SDK When:

* Encrypting data **before storing** in:

  * Databases
  * S3
  * Message queues
* Encrypting data in **microservices**
* Data flows through **multiple systems**
* You want **tamper detection** automatically

### ❌ Do NOT Use When:

* You need DynamoDB-specific encryption
* You need S3-specific encryption features

---

## 🔄 AWS Encryption SDK vs Other Clients

| Encryption Client          | Use Case                   |
| -------------------------- | -------------------------- |
| AWS Encryption SDK         | General-purpose encryption |
| DynamoDB Encryption Client | DynamoDB attributes        |
| S3 Encryption Client       | S3 object-level encryption |

⚠️ These clients are **NOT compatible** with each other.

---

## ⚙️ How AWS Encryption SDK Works (Envelope Encryption)

![Image](https://docs.aws.amazon.com/images/encryption-sdk/latest/developer-guide/images/envelope-encryption-70.png?utm_source=chatgpt.com)

![Image](https://docs.aws.amazon.com/images/kms/latest/developerguide/images/generate-data-key.png?utm_source=chatgpt.com)

### Flow Summary

1. SDK requests data key from KMS using CMK
2. KMS returns encrypted data key
3. SDK encrypts data locally
4. SDK attaches encryption context
5. Encrypted data is stored or transmitted
6. On decryption, SDK:

   * Verifies integrity
   * Decrypts data key
   * Returns plaintext safely

---

## 🛠️ Step 01 — Install AWS Encryption SDK (Node.js)

```bash
npm install @aws-crypto/client-node
```

---

## 📦 Step 02 — Import SDK Modules

```js
const {
  KmsKeyringNode,
  encrypt,
  decrypt
} = require("@aws-crypto/client-node");
```

### 🔑 Why Keyring?

AWS Encryption SDK uses **Keyrings** to:

* Manage data keys
* Encrypt data keys using KMS CMKs
* Perform envelope encryption automatically

---

## 🔐 Step 03 — Configure KMS Keyring

```js
const masterKeyId =
  "arn:aws:kms:us-east-1:123456:key/xxxx";

const keyring = new KmsKeyringNode({
  masterKeyId
});
```

💡 You can configure **multiple CMKs** for higher security and redundancy.

---

## 🧾 Step 04 — Create Encryption Context (Recommended)

Encryption context provides **tamper detection**.

```js
const encryptionContext = {
  accountId: "100",
  purpose: "youtube demo",
  country: "Sri Lanka"
};
```

✔ Not secret
✔ Stored with encrypted data
✔ Must match during decryption

---

## 🔒 Step 05 — Encrypt Data

```js
const plaintext = "My passwords for sensitive data";

const { result } = await encrypt(
  keyring,
  plaintext,
  { encryptionContext }
);
```

### Output

* `result` → Encrypted data (binary)
* Automatically protected with integrity checks

---

## 🔓 Step 06 — Decrypt Data

```js
const {
  plaintext,
  messageHeader
} = await decrypt(keyring, result);
```

### Returned Values

* `plaintext` → Original data
* `messageHeader` → Encryption metadata

---

## ✅ Step 07 — Verify Encryption Context

```js
const originalContext = {
  accountId: "100",
  purpose: "youtube demo",
  country: "Sri Lanka"
};

Object.entries(originalContext).forEach(([key, value]) => {
  if (messageHeader.encryptionContext[key] !== value) {
    throw new Error("Data integrity check failed!");
  }
});

console.log("Encryption context verified. Data is intact.");
```

✔ Detects tampering
✔ Prevents unauthorized modification

---

## 🔐 Security Best Practices (Added)

* ✅ Always use encryption context
* ✅ Use IAM least-privilege for KMS
* ✅ Enable CloudTrail for KMS
* ✅ Rotate CMKs regularly
* ❌ Never log plaintext data
* ❌ Never hardcode CMK ARNs in code (use env vars)

---

## 🧠 DevOps & Architecture Use Cases

* Encrypt data in **CI/CD pipelines**
* Secure secrets before pushing to storage
* Encrypt payloads between microservices
* Compliance-ready encryption for regulated workloads

---

## 📌 Conclusion

AWS Encryption SDK provides a **safe, simple, and production-ready** approach to client-side encryption by:

* Abstracting cryptography complexity
* Enforcing envelope encryption
* Providing built-in integrity validation
* Integrating seamlessly with AWS KMS

This makes it ideal for **modern DevOps, cloud-native, and microservices architectures**.

---

## 🚀 Next Enhancements

* Add CI/CD automation
* Integrate with AWS Secrets Manager
* Browser-side encryption example
* Multi-CMK keyring setup

---


