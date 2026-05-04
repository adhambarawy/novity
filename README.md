# Amazon Q Developer VS Code Extension - Proxy Issue

## Issue Summary

The Amazon Q Developer VS Code extension (v2.1.0) is authenticated and connected via AWS IAM Identity Center but cannot send or receive chat messages. The extension returns `ECONNREFUSED` errors when attempting to call the AWS CodeWhisperer API endpoints.

## Evidence from Logs

The Amazon Q Logs output panel shows the following error on every chat attempt:

```
"code": "ECONNREFUSED"
"name": "Error"
"cause": { "code": "ECONNREFUSED" }
"attempts": 3
"totalRetryDelay": 106
```

This confirms the outbound HTTPS connection to AWS is being refused by the corporate proxy/firewall.

## What is Working

- ✅ Authentication via IAM Identity Center is successful (token valid until August 2026)
- ✅ Kiro CLI (the terminal-based version) works correctly on the same machine
- ✅ The VS Code extension correctly shows "Connected with IAM Identity Center" in the menu

## What is Not Working

- ❌ Amazon Q chat in VS Code returns no response to any message
- ❌ All API calls from the VS Code extension language server are blocked

## Root Cause

The VS Code extension makes outbound HTTPS calls to AWS CodeWhisperer endpoints from the VS Code language server process. These calls are being blocked by the BHP corporate proxy, while the Kiro CLI (which uses a different network path) is not affected.

## Action Required from IT

Please whitelist the following AWS endpoints in the BHP corporate proxy/firewall for outbound HTTPS (port 443) traffic from my machine:

- `codewhisperer.us-east-1.amazonaws.com`
- `*.amazonaws.com` (specifically us-east-1 region endpoints)
- `q.us-east-1.amazonaws.com`

This is required for the BHP-approved Amazon Q Developer VS Code extension to function. I am one of 102 BHP employees recently added to the Amazon Q Developer programme (confirmed via BHP WebEx space announcement).

## Machine Details

| Field | Value |
|-------|-------|
| Hostname | METC005483 |
| User | elbaad (Adham Elbarawy) |
| Location | 480 Queen Level 11 |
| OS | Windows |
| VS Code Version | 1.118.1 |
| Amazon Q Extension Version | 2.1.0 |
| AWS Region | us-east-1 |
| IAM Identity Center Start URL | https://bhp.awsapps.com/start |

## Authentication Details

- **Status**: Connected with IAM Identity Center
- **Token Expiry**: August 2026
- **Region**: us-east-1
