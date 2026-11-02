# CSC Attendance Portal

A web portal for Computer Science students to track class attendance.

- **Students** sign up, pick their courses, and see how many classes they attended or missed, and whether they are still allowed to write the continuous assessment.
- **Teachers and course delegates** create classes, mark attendance, manage delegates, and download an Excel sheet of their students.

Built with plain HTML, CSS and JavaScript. There is no server: the database and login are handled by Firebase (Authentication and Firestore).

---

## Project structure

```
CSC_Attendance_Portal/
├── index.html                 Login page
├── signup.html                Sign-up page
├── assets/                    Images and icons
├── css/
│   └── base.css               Shared styles
├── js/
│   ├── firebase-service.js    BACKEND: all Firebase code lives here only
│   ├── guard.js               Redirects users who are not logged in or have the wrong role
│   ├── ui.js                  Shared helpers: loading, error messages, confirmations
│   └── pages/                 One script per page
├── student/                   Pages for students
├── teacher/                   Pages for teachers and course delegates
├── firestore.rules            BACKEND: database security rules
└── README.md
```

## Who owns what

| Person | Owns | Does not edit |
|---|---|---|
| Frontend | Every `.html` file, `css/`, `js/pages/`, `js/guard.js`, `js/ui.js`, `assets/` | `js/firebase-service.js`, `firestore.rules` |
| Backend | `js/firebase-service.js`, `firestore.rules`, the Firebase console | Page layout and styling |

The frontend never writes Firebase code. Every database action is a function exported from `js/firebase-service.js`.

---

## Getting started (everyone)

### 1. Install the tools

- [Visual Studio Code](https://code.visualstudio.com/)
- The **Live Server** extension for Visual Studio Code
- [Git](https://git-scm.com/) (if you share the project through GitHub)

### 2. Get the project

If it is on GitHub:

```bash
git clone <repository-link>
cd CSC_Attendance_Portal
```

Otherwise, copy the project folder onto your computer.

### 3. Run it

1. Open the project folder in Visual Studio Code.
2. Right-click `index.html` and choose **Open with Live Server**.
3. The site opens in your browser, usually at `http://127.0.0.1:5500`.

> **Do not double-click the HTML files.** The scripts use `import`, which browsers block when a page is opened directly from the file system. Always use Live Server (or any local web server).

Until the backend person has finished the Firebase setup below, login and sign-up will show errors. That is expected. The frontend person can build layouts in the meantime.

---

## Firebase setup (backend person, one time)

### 1. Create the project

1. Go to the [Firebase console](https://console.firebase.google.com/) and click **Add project**.
2. Name it, for example `csc-attendance-portal`. Google Analytics is not needed.

### 2. Turn on login

1. Open **Build → Authentication → Get started**.
2. Under **Sign-in method**, enable **Email/Password**.

Students log in with their UB number. The code converts it into a hidden email behind the scenes, so no extra setup is needed.

### 3. Create the database

1. Open **Build → Firestore Database → Create database**.
2. Pick a location close to your users and start in **production mode**.

### 4. Connect the website to Firebase

1. In **Project settings → General**, scroll to **Your apps** and click the web icon (`</>`).
2. Register the app, then copy the `firebaseConfig` values.
3. Paste them into the `firebaseConfig` object at the top of `js/firebase-service.js`.

### 5. Publish the security rules

1. Open **Firestore Database → Rules**.
2. Replace everything with the contents of `firestore.rules`.
3. Click **Publish**.

These rules are what stop students from marking their own attendance or making themselves delegates. Do not skip this step.

### 6. Add the courses

Open **Firestore Database → Data → Start collection**, name it `courses`, and add one document per course with these fields:

| Field | Type | Example |
|---|---|---|
| `code` | string | `CSC301` |
| `title` | string | `Operating Systems` |

Let Firestore generate the document identifiers. Students will pick from these when signing up.

### 7. Approve teachers

New teacher accounts start with `approved: false` and cannot do anything until approved.

1. Open **Firestore Database → Data → users**.
2. Find the teacher's document.
3. Change `approved` from `false` to `true`.

### 8. Allow the local address

In **Authentication → Settings → Authorized domains**, `localhost` is allowed by default. If Live Server uses `127.0.0.1` and login is blocked, add `127.0.0.1` to the list.

---

## Things to know

- **Confirm the attendance threshold.** `CA_MINIMUM_PERCENT` at the top of `js/firebase-service.js` is set to 75. Change it to match the department's real rule.
- **Unmarked students count as absent.** Students marked as excused count as attended.
- **First-use index prompt.** The first time the student list loads for a course, Firestore may show an error in the browser console containing a link. Open the link and click **Create index**. It takes a minute or two, and this happens once.
- **Excel download.** The teacher students page needs the SheetJS script tag. The link is in a comment above `downloadCourseSheet` in `js/firebase-service.js`.
- **Hosting later.** The project can be deployed free on Firebase Hosting without changing the structure.

## Roles at a glance

- **Student:** student pages only.
- **Approved teacher:** teacher pages for the courses they teach.
- **Course delegate:** a student whose `authorizedCourses` list is not empty. They use the student pages, and the teacher pages for those courses only. Only a teacher can assign or remove a delegate.

## Working together

- Each person edits only their own files (see **Who owns what**), which avoids conflicts.
- If a backend function is missing or behaves wrongly, the frontend person sends the backend person the function name and what they expected.
- If the browser shows an error, check the console first (right-click, **Inspect**, **Console**). Backend errors have readable messages.
- Full details for the frontend person are in `CSC_Portal_Frontend_Guide.pdf`.