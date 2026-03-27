# File Share Permissions Bypass

## Issue

Users were able to access restricted folders through a shared parent directory:

`\\SRV01\Shares`

This allowed indirect access to subfolders that were intended to be restricted.

## Cause

The parent directory (`C:\Shares`) was shared with broad permissions, allowing traversal into subfolders despite restrictions on individual shares.

## Resolution

Removed sharing from the parent directory:

1. Right-click `C:\Shares`
2. Properties → Sharing  → Advanced Sharing
3. Uncheck "Share this folder"
4. Apply changes

## Key Takeaway

Sharing parent directories can unintentionally expose restricted resources and create security risks. Always share only the specific folders required.

