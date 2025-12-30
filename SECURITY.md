# Security Policy

## Overview

This project is maintained as a personal, open-source project intended to demonstrate software architecture, engineering practices, and secure-by-design principles. I took this topic, researched it, planned out my approach, selected appropriate software including third-party packages and pushed myself to develop something functional within a ring-fenced amount of time (typically between 1-4 weeks). I'm busy with my life, I have a full-time job, family and hobbies which involve stepping away from the computer screen. All code is written with the best of intentions.

While reasonable care is taken to follow modern security best practices, this project is **not intended for production use in safety-critical or high-risk environments**.

---

## Supported Versions

Only the latest version on the `main` branch is supported for security fixes.

| Version | Supported |
|--------|-----------|
| main   | ✅        |
| older tags / forks | ❌ |

---

## Reporting a Vulnerability

If you discover a security vulnerability, please **do not open a public issue**.

Please report security issues using **GitHub’s Private Vulnerability Reporting feature**.

Please include:
- A clear description of the issue
- Steps to reproduce (proof-of-concept if possible)
- Potential impact and attack scenario
- Affected versions or commits

You can expect an acknowledgement within **7 days**.

---

## Disclosure Process

This project follows a **responsible disclosure** model:

1. The report is acknowledged
2. The issue is assessed and validated
3. A fix is developed and released
4. Public disclosure occurs after a fix is available

No guarantees are made regarding timelines, but good-faith reports will be handled professionally.

---

## Security Considerations

This project aims to demonstrate the following security principles:

- Least privilege and explicit trust boundaries
- Secure defaults and configuration validation
- Dependency hygiene and regular updates
- Defensive input validation and error handling
- Automated testing and static analysis

Security is treated as a **design concern**, not an afterthought.

---

Thank you for helping to improve the security of this project.
