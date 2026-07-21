# Hallway Security Model

This prototype treats a student-operated kiosk as hostile. Browser code is never an authorization boundary.

## Required deployment controls

- Host teacher and kiosk interfaces on different origins and browser profiles.
- Run the kiosk in a managed, unprivileged operating-system kiosk account. Restrict navigation, extensions, downloads, removable media, firmware boot, and browser password storage.
- Put PocketBase behind HTTPS, restrict CORS, enable HSTS, configure SMTP, encrypt PocketBase settings, and restrict superuser access by IP and MFA.
- Never put a superuser token, teacher token, recovery key, or teacher password in Vite environment variables or kiosk storage.
- Keep collections locked. Client writes go through authenticated custom routes with strict body limits.
- Treat student IDs as identifiers, not proof of identity. Production should use opaque badges or separate PINs if impersonation is not an accepted risk.
- Fail closed when PocketBase is unavailable. Never report approval until a server transaction succeeds.

## Authentication boundary

Teachers use PocketBase email/password authentication and built-in MFA orchestration. PocketBase does not natively support authenticator-app TOTP. This repository uses PocketBase email OTP as the supported second factor and does not implement custom TOTP. Production TOTP requires an audited external identity provider or separately reviewed PocketBase extension.

Teacher tokens use an in-memory `BaseAuthStore`; refreshing destroys the session. Kiosks never receive teacher credentials. A teacher creates a single-use, eight-digit, five-minute link code on a trusted device. Redemption creates a restricted device principal. Revocation disables the device and rotates its PocketBase token key.

## Encryption boundary

The teacher password is processed by libsodium Argon2id and authenticated secretbox encryption in the teacher browser. PocketBase stores ciphertext. Encryption protects server storage and backups; it cannot protect plaintext on a compromised endpoint. The kiosk must receive only the minimum current data, not the full roster or historical vault.

If both the teacher password and recovery key are lost, encrypted student records cannot be recovered.
