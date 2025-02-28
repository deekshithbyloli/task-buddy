# Task Management App

A responsive task management application built using **Next.js** and **Supabase**. This app allows users to efficiently create, organize, and track their tasks. It features user authentication via Supabase with Google Sign-In, drag-and-drop functionality, task categorization, due dates, and more.

## Features

1. **User Authentication**:

   - Sign in with Google using Supabase Authentication.
   - Manage user profiles.

2. **Task Management**:

   - Create, edit, and delete tasks.
   - Categorize tasks (e.g., Work, Personal).
   - Set due dates for tasks.
   - Drag-and-drop functionality to rearrange tasks.
   - Sort tasks by due dates (ascending/descending).

3. **Batch Actions**:

   - Delete multiple tasks at once.
   - Mark multiple tasks as complete.

4. **Task History and Activity Log**:

   - Track changes made to tasks (creation, edits, deletions).
   - Display an activity log for each task.

5. **File Attachments**:

   - Attach files or documents to tasks.
   - File upload feature in the task creation/editing form.

6. **Filter Options**:

   - Filter tasks by tags, category, and date range.
   - Search tasks by title.

7. **Board/List View**:

   - Switch between a Kanban-style board view and a list view.

8. **Responsive Design**:
   - Fully responsive design for mobile, tablet, and desktop.

## Tech Stack

- **Frontend**: Next.js, TypeScript, Tailwind CSS
- **Backend**: Supabase (Authentication, Database, Storage)
- **State Management**: Tanstack Query
- **Drag-and-Drop**: `@dnd-kit`

## Prerequisites

Before running the project, ensure you have the following:

1. **Node.js** installed (v16 or higher).
2. **Yarn** installed.
3. A **Supabase** project set up.
4. A **Google OAuth Client ID** for authentication.

## Setup Instructions

### 1. Create a Supabase Project

1. Go to [Supabase](https://supabase.com/) and create a new project.
2. After creating the project, navigate to the **Project Settings** > **API** section.
3. Copy the **Project URL** and **anon/public key**. These will be used to connect your app to Supabase.

### 2. Set Up Google Authentication

1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Create a new project or select an existing one.
3. Navigate to **APIs & Services** > **Credentials**.
4. Click **Create Credentials** and select **OAuth Client ID**.
5. Set the **Authorized Redirect URIs** to:

6. Copy the **Client ID** and **Client Secret**.

### 3. Enable Google Authentication in Supabase

1. Go to your Supabase project dashboard.
2. Navigate to **Authentication** > **Providers**.
3. Enable **Google** and paste the **Client ID** and **Client Secret** from Google Cloud Console.
4. Save the changes.

### 4. Set Up Environment Variables

Create a `.env.local` file in the root of your project and add the following variables:

```env
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key



```
```
yarn install
```
```
yarn dev
```

**The app will be running at**:  ``` http://localhost:3000 ```
### 5. Folder Structure

```

task-management-app/
├── components/ # Reusable components
├── hooks/ # Custom hooks
├── lib/ # Utility functions
├── pages/ # Next.js pages
├── styles/ # Global styles
├── types/ # TypeScript types
├── .env.local # Environment variables
├── next.config.js # Next.js configuration
├── package.json # Project dependencies
├── README.md # Project documentation
└── tsconfig.json # TypeScript configuration

```

## Challenges Faced

### Google Authentication
Setting up Google OAuth with Supabase required careful configuration of redirect URIs and client IDs.  
**Solution:** Followed Supabase and Google Cloud documentation to ensure proper setup.

### Drag-and-Drop Functionality
Implementing drag-and-drop for tasks was challenging due to state management and reordering logic.  
**Solution:** Used `@dnd-kit` library, which provided a robust and flexible API for drag-and-drop.

### Responsive Design
Ensuring the app worked seamlessly across devices required careful use of Tailwind CSS and media queries.  
**Solution:** Adopted a mobile-first approach and tested on multiple screen sizes.

---

## Deployment
The app is deployed on Vercel. You can access the live version here:  
[Live Demo](#)

---

## Screenshots
![Task Management App Screenshot](#)

---

## Contributing
Contributions are welcome! Please follow these steps:

1. Fork the repository.
2. Create a new branch:
    ```sh
    git checkout -b feature/your-feature
    ```
3. Commit your changes:
    ```sh
    git commit -m 'Add some feature'
    ```
4. Push to the branch:
    ```sh
    git push origin feature/your-feature
    ```
5. Open a pull request.

---

## License
This project is licensed under the MIT License. See the [LICENSE](./LICENSE) file for details.
