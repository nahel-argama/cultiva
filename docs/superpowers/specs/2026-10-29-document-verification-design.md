# Document Verification Design

## Intent

Let signed-in users submit an identity document and see whether backend verification has completed.

## Goals

- Add document upload to the dashboard's **Meu Perfil** tab.
- Show **Pendente** while document validation is pending, and **Verificado** when approved.
- Automatically poll the document status after upload and prevent resubmission while it is pending.

## Non-goals

- Change the registration payload or backend APIs.
- Add a client-side timeout that treats a still-pending validation as failed.

## Constraints

- `POST /auth/upload-document` requires an authenticated request with multipart field `document`.
- Backend accepts JPEG, JPG, PNG, and WebP images up to 5 MB, and returns `202` when verification is queued.
- `GET /auth/signup/document/status` returns `data.status` (`pending`, `approved`, or `failed`) and `data.message`; status is cached by the backend for up to 24 hours.
- `/auth/me` remains the source for the shared verified user profile.

## Approaches

- **Upload during registration:** create the account, then upload using its new token. A failed upload needs a second recovery flow.
- **Upload in Meu Perfil (chosen):** use the existing authenticated upload endpoint and allow resubmission in the same place. This matches the user's preference and gives upload failures a direct recovery path.

## Behavior

- Registration stays unchanged; the user submits an identity document from **Meu Perfil** after signing in.
- After a successful upload, wait 8 seconds and poll `GET /auth/signup/document/status` every 5 seconds while the profile view stays open.
- Keep document selection and upload disabled while status is `pending`. Continue polling until the backend returns a terminal status; a pending result after 24 seconds is not a validation failure.
- On `approved`, refresh `/auth/me`, show a success toast, and hide upload controls. On `failed`, show the backend message in an error toast and allow another upload.
- On status `404`, stop polling and show the backend error. Retry transient request errors on the next poll.
- Follow backend validation and show request errors in the profile.
