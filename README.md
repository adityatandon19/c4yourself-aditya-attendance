# C 4 Yourself Attendance — GitHub Pages frontend (Cloudflare D1)

This folder updates the original Firebase app to use your existing Cloudflare Worker + D1 backend.

## Deploy

1. In the GitHub repo `adityatandon19/c4yourself-aditya-attendance`, replace the **root** `index.html` with this `index.html`.
2. Upload `data/classes.json` (as `data/classes.json`, not in the root). Keep your existing `logo.png` unchanged.
3. Wait for GitHub Pages to redeploy. Open `https://adityatandon19.github.io/c4yourself-aditya-attendance/`.
4. At Teacher access, enter the **ADMIN_KEY** stored as a Cloudflare Worker Secret (never upload your secret to GitHub).
5. In Class rosters, import or paste each class's roster: one row per student, `Roll Number, Full Name`. The roster is stored in D1, not in GitHub. The Worker requires at least one student before creating a session.
6. In Daily schedule, choose the date, class and time slot. Click Generate attendance QR.
7. Students scan QR, enter roll number and email, and send attendance to D1. Refresh takes place every ~5 seconds.

## Configuration

`index.html` points to:

    https://c4yourself-attendance-api.adityat1904.workers.dev

Public classes and schedules are in `data/classes.json`; changing that file updates the class/timetable dropdowns on the next app load. Keep class IDs stable once rosters have been imported, or the app will not match the saved D1 rosters.

**Do not put students' names, university rolls, emails, your ADMIN_KEY or any secrets in public GitHub files.** Use the teacher roster import screen for students.

## Compatibility with the deployed API

This frontend assumes the Worker code from your Step 7 (the one with `/api/health`, `/api/join/:id`, `/api/rosters/:classId`, `/api/sessions`, and `/api/holidays`). No Worker code changes are required for importing rosters, starting QR attendance, live refresh or viewing history.

### Known limitations

- The Worker currently counts `verified`, `unverified` and `override` as present; `remote` and `denied` are saved but not counted as present.
- QR + roll/email cannot prove student identity or prevent a student from submitting another student's roll. Consider verified student login for stricter use.
- Geofencing uses device-provided location; it is not fraud-proof. Default geo-fencing is OFF until you actively enable it.
- Holiday flags in this frontend are **local to the current browser/device**, not synced across devices. The deployed Worker's holiday PUT implementation has a bug; this avoids relying on it until fixed.
- The Worker returns the **current** class roster for past sessions. If a roster is replaced later, old attendance displays may no longer reflect the original student list. Export important session CSVs before major roster changes.
- Keep the D1 database and worker deployment active for persistence; GitHub Pages alone is static.
- Student email is collected but not verified against an institutional identity system. Roll numbers are looked up against the D1 roster.
- Teacher secret is held in page memory only; a refresh requires entering it again. Active session IDs (not the key) are remembered for the browser tab.

## Quick troubleshooting

- `ok: true` from `/api/health` confirms D1 connectivity, not teacher authentication.
- `Teacher authorization required`: make sure the Cloudflare Secret is named exactly `ADMIN_KEY` (your deployment used this name, not TEACHER_KEY), and paste the **value** into the login field.
- `Upload the class roster first`: go to Class rosters and save at least one student for that class.
- `Cannot load ./data/classes.json`: ensure you uploaded `classes.json` inside the `data` folder.
- `Origin not allowed`: deploy to the intended GitHub Pages site (not a local `file://` preview). The Worker restricts browser origins to `https://adityatandon19.github.io`.
