# Document Verification Implementation Plan

> **For agentic workers:** Follow this plan task by task. The task uses the existing Vue Composition API and API service.

**Goal:** Let users upload their identity document from Meu Perfil and track backend validation status.

**Architecture:** Add API helpers for multipart upload and `GET /auth/signup/document/status`. `ProfileView.vue` polls the backend status, uses `useAuth().refreshProfile()` only after approval, and reports terminal status through the existing toast composable.

**Tech Stack:** Vue 3, JavaScript, Vite, existing `apiRequest`, `useAuth`, and `useToast`.

**Spec:** `docs/superpowers/specs/2026-10-29-document-verification-design.md`

## Global Constraints

- Keep registration unchanged; use the existing backend status endpoint.
- Upload multipart field `document` to `/auth/upload-document`.
- Accept JPEG, JPG, PNG, and WebP up to 5 MB, matching backend validation.
- The status endpoint returns `pending`, `approved`, or `failed` plus a message.
- Hide upload controls when verified; block upload while status is pending and allow retry after failed.

## Review Focus

- Invalid file type or file over 5 MB must not be sent; validate before calling the API.
- Upload errors must leave the user able to retry and show an error message.
- Pending validation may last longer than 24 seconds; do not report timeout as a backend failure.
- A successful status response of `approved` must refresh `/auth/me` so shared auth state updates.
- Keep multipart `Content-Type` unset so the browser supplies its boundary.

---

### Task 1: Add authenticated document API helpers

**Files:**
- Modify: `frontend/src/services/api.js`

**Interface:**
- Keep `uploadDocument(file)` using multipart field `document`.
- Add `getDocumentVerificationStatus()` for `GET /auth/signup/document/status`.
- Existing `apiRequest` attaches the stored bearer token and omits JSON content type for `FormData`.

- [x] Add and inspect the function; ensure request body contains the selected file under `document`.

### Task 2: Add profile verification controls

**Files:**
- Modify: `frontend/src/components/dashboard/views/ProfileView.vue`
- Consume: `uploadDocument(file)` and `getDocumentVerificationStatus()` from Task 1, plus existing `auth.refreshProfile()`.

- [x] Show **Pendente** and image upload controls when `auth.profileData.value.user.is_verified` is false; show **Verificado** and hide upload controls when true.
- [x] Validate image MIME type and 5 MB limit before upload; show the selected filename and keep controls retryable after failures.
- [x] After accepted upload, wait 8 seconds then poll status every 5 seconds. Keep upload disabled while status is pending.
- [x] On approval, refresh `/auth/me` and show success; on backend failure, show its message and allow retry. Do not impose a client timeout on pending status.

### Verification

- [x] Run `npm run build` in `frontend/`.
- [x] Run `git diff --check`.
- [x] Confirm registration and backend files remain unchanged by this frontend integration.
