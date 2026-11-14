# Kindred School Registry

A responsive React dashboard for managing students, teachers, and employees.

## Features

- Sidebar navigation and live record-count cards for each directory.
- Search by name, roll number/ID, address, subject, or position.
- Default ordering or alphabetical sorting.
- Accessible, labeled add/edit forms, immutable record identifiers, and confirmed deletion.
- Loading placeholders, service connectivity indicators, retry controls, and save/error notifications.
- Mobile layouts, keyboard focus indicators, a skip-to-content link, and reduced-motion support.

Record totals use real API data, not sample records. An unavailable service shows
an explicit error and retains any previously loaded records. Its mutation controls
are disabled until connectivity recovers. The dashboard refreshes all directories
10 seconds after each background refresh completes.

## Development

```sh
npm ci
npm start
```

Manual development connects directly to locally running APIs:

| Directory | API base |
| --- | --- |
| Students | `http://localhost:5001` |
| Teachers | `http://localhost:5002` |
| Employees | `http://localhost:5003` |
| Profile photo | `http://localhost:5004` |

`npm start` serves the frontend on http://localhost:3000. Follow the manual
service-by-service steps in the [root README](../READEME.md).
Local thumbnails use signed URLs directly to http://localhost:9000.
The media API reads `FRONTEND_ORIGIN` from `profileservice/.env`.

For the full container stack, run `docker compose up -d --build --wait` from the
repository root and open http://localhost:8080. The frontend image uses
`/api/students`, `/api/teachers`, `/api/employees`, and `/api/profile` as its
build-time API bases from `.env.production`. Nginx proxies these and signed
`/media/` downloads. Docker excludes local `.env` files but includes
`.env.production`, which contains only public API paths.

Optional build-time overrides: `REACT_APP_STUDENT_API_URL`,
`REACT_APP_TEACHER_API_URL`, `REACT_APP_EMPLOYEE_API_URL`, and
`REACT_APP_PROFILE_API_URL`. Edit `.env.production` for container builds or
`.env` for local development. Rebuild the image after changing production
settings. These are public browser addresses, not Docker hostnames or secrets.

## Validation

```sh
CI=true npm test -- --watchAll=false
npm run build
```

Photos can be chosen when creating any directory entry, or uploaded/replaced/removed
using its photo button later. The shared school avatar uses the same asynchronous
thumbnail pipeline. Initials remain until a first thumbnail is ready; replacements
keep the last successful photo while processing. Status badges, reconnect, and
retry actions make failures explicit. A failed upload never resubmits the saved
registry entry. Limits: JPEG/PNG/WebP, 5 MiB, 20 megapixels (validated server-side).

Tests mock the backend services and cover directory counts, navigation, search,
sorting, all three CRUD workflows, rejected saves, failed connections, and empty
or malformed API responses. Profile photo tests cover upload, validation, removal,
expired presigned URLs, and an offline profile service.

## Structure

- [App](src/App.js): dashboard layout, API calls, notifications, and connectivity.
- [Registry configuration](src/registry.js): service routes and directory-specific fields.
- [RecordForm](src/components/RecordForm.js) and [RecordList](src/components/RecordList.js):
  shared UI for all three directories.
- [Photo](src/components/Photo.js), [ProfileAvatar](src/components/ProfileAvatar.js),
  and [profileApi](src/profileApi.js): shared photo controls and processing-state polling.
- [App styles](src/App.css): responsive layout, colors, and motion.

No external fonts, icon libraries, or new dependencies are required.
