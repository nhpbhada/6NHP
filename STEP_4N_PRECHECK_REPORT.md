# STEP 4N — Production Live User Journey Preflight

## Audit performed on uploaded project

The uploaded project was inspected directly before any live production write test.

### Important finding

The previous security audits protected NWBDCP mutation endpoints, but several NWBDCP read endpoints were still accessible without NWBDCP authentication. These included dashboard, water sources, dispatched samples, retests, reports and storage-related reads. NWBDCP export endpoints were also unauthenticated.

Because these endpoints can expose operational water-source/sample/laboratory information, this was treated as a production security blocker before live-user testing.

## Changes applied

### Server
- Protected NWBDCP read endpoints with server-side `nwbdcpRequireAuth(ROLES.STAFF)`.
- Protected storage file serving with NWBDCP authentication.
- Protected storage integrity check with ADMIN authorization.
- Protected NWBDCP export endpoints with STAFF authorization.
- Logout now authenticates the current JWT and revokes the server-verified session ID; client-supplied session IDs are ignored.

### Frontend
- Converted NWBDCP protected reads to `nwbdcpFetch()` so the JWT is sent.
- After successful Google login, protected NWBDCP data is reloaded automatically.
- Converted NWBDCP export links to authenticated blob downloads.
- Laboratory report preview now retrieves the file through authenticated NWBDCP API access rather than direct unauthenticated file URLs.
- Updated anonymous NWBDCP authentication wording.

### Dependency
- Added `google-auth-library` as an explicit application dependency because `services/nwbdcpAuth.js` imports it directly.

## NVBDCP protection

The following NVBDCP core files were byte-for-byte unchanged:
- Form.html
- defaultData.js
- data/store.js
- services/reports.js
- services/firebaseStore.js
- firestore.rules

## Data integrity

Uploaded project NWBDCP JSON baseline:
- Water Sources: 156
- Dispatches: 12
- Sample Results: 12
- Retests: 0
- Audit Logs: 8

No project data files were modified by the patch.

## Verification

Passed:
- server.js syntax check
- services/nwbdcpAuth.js syntax check
- services/nwbdcpDriveStorage.js syntax check
- services/nwbdcpStore.js syntax check
- services/nwbdcpSupabase.js syntax check
- route protection static audit
- frontend raw NWBDCP read audit
- NVBDCP core-file byte comparison

Not completed in this environment:
- full `npm ci` / live application startup, because package installation requires network access not available in the execution environment.
- actual Google Login
- actual Cloud Run request testing
- actual Supabase/Google Drive live write testing

Therefore this is a code-level production hardening patch, not a claim of successful live deployment.
