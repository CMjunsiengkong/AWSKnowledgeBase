---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: AWS CLI & CloudShell
version: C (by service)
source_chapters: [04]
related: [IAM, EC2]
tags: [aws, saa-c03, cli, sdk, access-keys, cloudshell]
---

# AWS CLI, SDK and CloudShell

Concept-only note from chapter 04. Install and `aws configure` steps are in [[04 - IAM & AWS CLI]] (Version B). Related: [[IAM]].

## 1. Three ways to access AWS
(src: 04/08-AWS Access Keys, CLI and SDK)

| Method | Protected by | Use |
|---|---|---|
| Management Console | username + password (+ MFA) | web UI |
| CLI (command line) | access keys | commands from a shell; scripts and automation |
| SDK | access keys | API calls from application code |

## 2. Access keys
(src: 04/08, 04/12-AWS CLI Hands On)
- Generated in the console by each user, who is responsible for them. **Access key ID = like a username; secret access key = like a password.** Never share them.
- The secret key is **shown only once** at creation (download it).
- AWS shows recommended alternatives (for CLI: **CloudShell** or CLI v2 with **IAM Identity Center** authentication).
- The CLI has exactly the same permissions as the user whose keys it uses: removing the user's permissions makes the same calls fail in console and CLI.

> [!tip] Exam
> Access keys power CLI and SDK; keep them secret. For AWS services use IAM roles instead (see [[IAM]]).

## 3. AWS CLI
(src: 04/08, 04/09-AWS CLI Setup on Windows, 04/10-AWS CLI Setup on Mac OS X, 04/11-AWS CLI Setup on Linux)
- Tool to interact with AWS services via commands starting with `aws` (e.g. `aws s3 cp`); direct access to public APIs; scriptable; open source (GitHub); alternative to the console.
- **Use CLI version 2** (improved performance/capabilities and installer, same API as v1). Installers: MSI (Windows), PKG (macOS), zip + install script as root (Linux). Verify with `aws --version`.

## 4. SDK
(src: 04/08)
- Language-specific libraries embedded in application code to call AWS APIs: JavaScript, Python, PHP, .NET, Ruby, Java, Go, Node.js, C++; plus **mobile** (Android, iOS) and **IoT device** SDKs.
- The AWS CLI itself is built on the **AWS SDK for Python (Boto)**.

## 5. AWS CloudShell
(src: 04/14-AWS CloudShell, 04/13 no transcript)
- Browser-based **terminal in AWS**, **free to use**, started from the console toolbar icon; AWS CLI preinstalled.
- **Not available in every region**: check the CloudShell region availability.
- Credentials are those of the signed-in console identity (no key setup). Default region = the region you are currently in (override with `--region`).
- **Persistent storage**: files in your home directory survive restarts of CloudShell.
- Features: font size/theme settings, **upload and download files**, multiple tabs and split panes.

> [!tip] Exam
> CloudShell = free browser terminal using your console credentials, region-limited, with persistent home storage.

[verify] CloudShell region availability and persistence details were stated at recording time and may have changed. Unconfirmed, not checked against AWS docs.

## Not included here
- Hands-on narration (OS installs, `aws configure`, `aws iam list-users`, CloudShell tour).
- Lectures without transcript: 04/13 AWS CloudShell- Region Availability.
