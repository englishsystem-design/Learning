# SMK English Learning Platform

A professional English learning platform for vocational high schools (SMK).

- **Frontend** — GitHub Pages, one file: `index.html`
- **Backend** — Google Apps Script Web App, one file: `Code.gs`
- **Database** — Google Sheets
- **File storage** — Google Drive
- **AI (optional)** — OpenAI, called only from the backend

There are exactly two roles: **TEACHER** and **STUDENT**. There is no admin role. All technical configuration is done through Apps Script *Script Properties* and the setup functions in `Code.gs`.

```
SMK-ENGLISH-PLATFORM/          (GitHub repository → GitHub Pages)
├── index.html
└── README.md

SMK-ENGLISH-BACKEND/           (Google Apps Script project)
└── Code.gs
```

---

## 1. Create the Google Sheet

1. Go to <https://sheets.google.com> and create a blank spreadsheet.
2. Name it, for example, `SMK English Database`.
3. Copy the spreadsheet ID from the URL:
   `https://docs.google.com/spreadsheets/d/`**`THIS_IS_THE_ID`**`/edit`
4. Keep the ID. You will paste it into `SPREADSHEET_ID`.

Do not create any sheets or headers by hand. `setupSystem()` creates all 21 sheets and their headers, and it never deletes or clears existing data.

## 2. Create the Google Drive folder

1. Go to <https://drive.google.com> and create a folder, for example `SMK English Files`.
2. Open it and copy the folder ID from the URL:
   `https://drive.google.com/drive/folders/`**`THIS_IS_THE_ID`**
3. Keep the ID. You will paste it into `DRIVE_FOLDER_ID`.

All uploaded lesson materials and student submissions are stored in this folder. The spreadsheet holds only the metadata (file ID, name, MIME type, size, owner, entity).

## 3. Create the Apps Script project

1. Go to <https://script.google.com> and choose **New project**.
2. Rename the project to `SMK-ENGLISH-BACKEND`.
3. Delete the contents of the default `Code.gs` file.
4. Paste the entire contents of this repository's `Code.gs` into it and save.

## 4. Create the Script Properties

In the Apps Script editor: **Project Settings → Script Properties → Add script property**.

| Property | Required | Example | Meaning |
|---|---|---|---|
| `SPREADSHEET_ID` | Yes | `1AbC...xyz` | The database spreadsheet from step 1. |
| `DRIVE_FOLDER_ID` | Yes | `1DeF...uvw` | The upload folder from step 2. |
| `TIMEZONE` | Recommended | `Asia/Jakarta` | Application timezone. Defaults to `Asia/Jakarta`. |
| `APP_VERSION` | Recommended | `1.0.0` | Shown on the profile page and in `healthCheck()`. |
| `OPENAI_API_KEY` | Only for AI | `sk-...` | Secret. Never appears in the frontend, in responses or in logs. |
| `OPENAI_MODEL` | Only for AI | `gpt-4o-mini` | Model used for AI requests. |
| `AI_ENABLED` | Only for AI | `true` | `true` or `false`. Master switch. |
| `AI_DAILY_LIMIT` | Only for AI | `300` | Maximum AI requests per day for the whole school. |
| `AI_USER_DAILY_LIMIT` | Only for AI | `20` | Maximum AI requests per day per person. |
| `AI_MAX_INPUT_LENGTH` | Only for AI | `4000` | Maximum characters accepted in one AI request. |
| `AI_MAX_OUTPUT_TOKENS` | Only for AI | `700` | Maximum tokens returned by the model. |
| `AI_TIMEOUT_MS` | Only for AI | `25000` | Request timeout in milliseconds. |

`SERVER_SECRET` is generated automatically the first time it is needed. Do not create or edit it.

The platform runs fully without any of the AI properties. If `AI_ENABLED` is `false` or `OPENAI_API_KEY` is missing, the AI panels say so and every other feature keeps working.

## 5. Run setupSystem()

1. In the Apps Script editor, choose the function `setupSystem` and press **Run**.
2. Authorize the script when prompted (Sheets, Drive, external requests).
3. Check the execution log. It lists the sheets that were created and confirms the Drive folder.

`setupSystem()` is safe to run again at any time. It creates what is missing, repairs headers, and never removes rows.

## 6. Run healthCheck()

1. Choose `healthCheck` and press **Run**.
2. Open **Execution log** and read the JSON result.

A healthy deployment shows:

```
"spreadsheet": "OK"
"missingSheets": []
"headerMismatch": []
"drive": "OK"
"ok": true
```

It also shows `aiEnabled` and `aiConfigured` as `true` or `false`. It never returns the API key, any password hash or any session token.

## 7. Create the first teacher account

There is no admin role and no public sign-up. The first teacher is created from the editor only.

1. Open `Code.gs` and find `createFirstTeacher()`.
2. Change `EMAIL`, `NAME` and `PASSWORD` at the top of the function.
3. Save, select `createFirstTeacher`, press **Run**.
4. Read the log to confirm the account was created.
5. Sign in to the web app with that email and password, then change the password from the profile menu.

`createFirstTeacher()` refuses to run if a teacher already exists, and it is not reachable through the HTTP API, so no visitor can create a teacher account.

To add more teachers later, edit the three constants at the top of `createTeacherAccount()` and run that function.

## 8. Deploy as a Web App

1. In the Apps Script editor choose **Deploy → New deployment**.
2. Select type **Web app**.
3. Description: `SMK English API v1`.
4. **Execute as**: *Me* (the deployment owner). The backend owns the spreadsheet and the Drive folder.
5. **Who has access**: *Anyone*. This is required because the GitHub Pages frontend calls the API anonymously over HTTPS; the platform's own session tokens, role checks and ownership checks do the authorisation.
6. Press **Deploy**, authorize, then copy the **Web app URL**. It ends in `/exec`.

Test it by opening the URL in a browser. You should see a small JSON response containing `"success": true` and a `data` object with the service name, version and server time. This probe returns no private data and requires no token.

## 9. Create the GitHub repository

1. Create a new public repository, for example `SMK-ENGLISH-PLATFORM`.
2. Add `index.html` and `README.md` from this project.

## 10. Set API_BASE_URL

Open `index.html` and edit the single configuration line near the top of the script:

```js
var API_BASE_URL = 'PASTE_YOUR_APPS_SCRIPT_WEB_APP_EXEC_URL_HERE';
```

Replace it with the `/exec` URL from step 8:

```js
var API_BASE_URL = 'https://script.google.com/macros/s/AKfy.../exec';
```

This is the only value you ever change in the frontend. No key, password or secret belongs in this file.

## 11. Enable GitHub Pages

1. Repository **Settings → Pages**.
2. **Source**: *Deploy from a branch*.
3. **Branch**: `main`, folder `/ (root)`. Save.
4. Wait for the green confirmation and open the published URL.

## 12. Test the platform

Open the GitHub Pages URL and work through this list:

1. Sign in as the teacher you created.
2. Create a curriculum, then a unit inside it.
3. Create a lesson, add blocks, save the draft, assign it to a class, publish it.
4. Create a class and add a student account (Classes → Add student → New student account).
5. Create an assignment for that class.
6. Add questions to the question bank and publish them.
7. Build a quiz from those questions, assign classes, publish it.
8. Sign out, sign in as the student, open a lesson, hand in an assignment, take the quiz.
9. Sign back in as the teacher and grade the submission, then release the result.
10. Check Assessment and Analytics.
11. Test AI last, and only after the OpenAI properties are set.

## 13. Student accounts

Teachers create student accounts inside the app:

**Classes → open a class → New student account**, or **Classes → open a class → Add existing student** to enrol someone who already has an account.

- A student's password must be at least 8 characters and contain a letter and a number.
- Teachers can reset a student's password from the class roster. Passwords are stored only as salted hashes and are never displayed anywhere.
- Moving a student between classes uses **Transfer**, which changes the enrolment record. Student records are never duplicated.

## 14. OpenAI configuration

AI is optional. To switch it on:

1. Create an API key at <https://platform.openai.com>.
2. Add `OPENAI_API_KEY` as a Script Property.
3. Add `OPENAI_MODEL`, for example `gpt-4o-mini`.
4. Set `AI_ENABLED` to `true`.
5. Set `AI_DAILY_LIMIT`, `AI_USER_DAILY_LIMIT`, `AI_MAX_INPUT_LENGTH`, `AI_MAX_OUTPUT_TOKENS` and `AI_TIMEOUT_MS`.
6. Run `healthCheck()` and confirm `aiConfigured: true`.

OpenAI usage is billed by OpenAI. It is not free and the platform never claims that it is.

**To change the model**, edit `OPENAI_MODEL` and save. No redeployment is needed.

**To disable AI**, set `AI_ENABLED` to `false`. Every AI panel then shows a short notice and everything else — login, lessons, assignments, quizzes, classes, progress — continues to work exactly as before. The same is true if OpenAI is down, rate limited or out of quota.

Teacher AI output is always a draft. Nothing is saved until the teacher copies it into a lesson, a question or a vocabulary entry and saves it themselves.

## 15. Updating the frontend

1. Edit `index.html`.
2. Commit and push to `main`.
3. GitHub Pages republishes within about a minute.
4. Ask users to reload the page.

## 16. Updating the backend

1. Paste the new `Code.gs` into the Apps Script project and save.
2. Run `setupSystem()` again if the schema changed. Existing data is preserved.
3. **Deploy → Manage deployments → edit the existing deployment → Version: New version → Deploy.**
4. Editing the existing deployment keeps the same `/exec` URL, so `API_BASE_URL` does not change.

If you instead create a *new deployment*, you get a new URL and must update `API_BASE_URL` in `index.html`.

## 17. Optional demo data

`seedDemoData()` creates one curriculum, one unit, one published lesson with blocks and vocabulary, one class, one quiz, one assignment and one demo student (`demo.student@school.sch.id` / `Student2026`).

It refuses to run if the platform already has curriculum or lesson content, so it cannot overwrite real teaching material. Run it only on a test deployment.

## 18. Maintenance

- `cleanupSessions()` removes expired session records. Add a time-driven trigger for it (**Triggers → Add trigger → cleanupSessions → Time-driven → Day timer**) if you wish.
- `AuditLog` records logins, content changes, grading, submissions, quiz activity and AI usage. It never records passwords, password hashes, API keys or session tokens.
- Archive rather than delete. Lessons, questions, quizzes, assignments and classes all support an `ARCHIVED` status that preserves history.

## 19. Security notes

- The frontend is untrusted. Identity, role, ownership, lesson status, quiz score and attempt count are all decided by the backend.
- Session tokens are cryptographically random, expire, and are validated on the server on every request.
- DRAFT and ARCHIVED lessons are filtered out on the backend, so a student cannot reach them by guessing an ID.
- Quiz answer keys are never sent to the browser. Scores are computed on the server from the authoritative question records.
- The OpenAI key exists only in Script Properties. It is never in `index.html`, in GitHub, in the spreadsheet, in an API response or in a log.
- Never commit any key, password or token to the repository.
