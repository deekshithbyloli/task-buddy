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
### 5. Supabase Schema Explanation
# 1. Profiles Table

```
create table profiles (
  id uuid references auth.users on delete cascade not null primary key,
  updated_at timestamp with time zone,
  username text unique,
  full_name text,
  avatar_url text,
  website text,

  constraint username_length check (char_length(username) >= 3)
);

-- Enable Row Level Security (RLS)
alter table profiles enable row level security;

-- Policies
create policy "Public profiles are viewable by everyone." on profiles
  for select using (true);

create policy "Users can insert their own profile." on profiles
  for insert with check ((select auth.uid()) = id);

create policy "Users can update own profile." on profiles
  for update using ((select auth.uid()) = id);

-- Trigger to create a profile when a new user signs up
create function public.handle_new_user()
returns trigger
set search_path = ''
as $$
begin
  insert into public.profiles (id, full_name, avatar_url)
  values (new.id, new.raw_user_meta_data->>'full_name', new.raw_user_meta_data->>'avatar_url');
  return new;
end;
$$ language plpgsql security definer;

create trigger on_auth_user_created
  after insert on auth.users
  for each row execute procedure public.handle_new_user();
```

# 2. Tasks Table 
```
create table tasks (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users on delete cascade not null,
  title text not null,
  description text,
  category text check (category in ('Work', 'Personal')) not null,
  due_date date not null,
  status text check (status in ('To Do', 'In Progress', 'Completed')) not null,
  position integer not null default 0,
  created_at timestamp with time zone default now(),
  updated_at timestamp with time zone default now()
);

-- Enable RLS and policies
alter table tasks enable row level security;

create policy "Users can view their own tasks." on tasks
  for select using (auth.uid() = user_id);

create policy "Users can insert their own tasks." on tasks
  for insert with check (auth.uid() = user_id);

create policy "Users can update their own tasks." on tasks
  for update using (auth.uid() = user_id);

create policy "Users can delete their own tasks." on tasks
  for delete using (auth.uid() = user_id);

-- Indexes for sorting
create index on tasks (due_date);
create index on tasks (status, position);

-- Trigger to update `updated_at` timestamp
create function public.handle_task_update()
returns trigger
set search_path = ''
as $$
begin
  new.updated_at = now();
  return new;
end;
$$ language plpgsql security definer;

create trigger on_task_updated
  before update on tasks
  for each row execute procedure public.handle_task_update();
```
# 3.Task Tags Table
```
create table task_tags (
  id uuid default gen_random_uuid() primary key,
  task_id uuid references tasks on delete cascade not null,
  tag text not null
);

-- Index for faster tag queries
create index on task_tags (task_id);
```

# 4.Task Attachments Table
```
create table task_attachments (
  id uuid default gen_random_uuid() primary key,
  task_id uuid references tasks on delete cascade not null,
  file_url text not null,
  uploaded_at timestamp with time zone default now()
);

-- Set up Storage for task attachments
insert into storage.buckets (id, name)
  values ('task_attachments', 'task_attachments');

-- Policies for task attachments
create policy "Users can upload task attachments." on storage.objects
  for insert with check (bucket_id = 'task_attachments');

create policy "Users can view their own task attachments." on storage.objects
  for select using (bucket_id = 'task_attachments');``
```

# 5. Activity Logs Table
```
create table activity_logs (
  id uuid default gen_random_uuid() primary key,
  task_id uuid references tasks on delete cascade not null,
  user_id uuid references auth.users on delete cascade not null,
  action text check (action in ('Created', 'Updated', 'Deleted')) not null,
  description text not null,
  created_at timestamp with time zone default now()
);

-- Enable RLS and policies
alter table activity_logs enable row level security;

create policy "Users can view their own activity logs." on activity_logs
  for select using (auth.uid() = user_id);

create policy "Users can insert activity logs for their own tasks." on activity_logs
  for insert with check (auth.uid() = user_id);
```

# 6. Activity Log Function and Trigger
# Task Management App
```
create function public.log_task_activity()
returns trigger
language plpgsql
as $$
declare
  action_text text;
  description_text text;
begin
  if (tg_op = 'INSERT') then
    action_text := 'Created';
    description_text := 'Task "' || new.title || '" was created.';
  elsif (tg_op = 'UPDATE') then
    action_text := 'Updated';
    description_text := 'Task "' || new.title || '" was updated.';
  end if;

  insert into activity_logs (task_id, user_id, action, description)
  values (coalesce(new.id, old.id), coalesce(new.user_id, old.user_id), action_text, description_text);

  return new;
end;
$$ security definer;

create trigger on_task_change
  after insert or update on tasks
  for each row
  execute function public.log_task_activity();
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
The app is deployed on Netlify. You can access the live version here:  
[Live Demo](https://taskbuddyy.netlify.app/)

---

## Screenshots
![Task Management App  Home Screenshot](/public/home.png)
![Task Management Schema Screenshot](/public/schema.png)
![Task Management list Screenshot](/public/task-1.png)
![Task Management board Screenshot](/public/task-2.png)

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

---

## Repository
Check out the source code on GitHub:  
[Task Buddy GitHub Repository](https://github.com/deekshithbyloli/task-buddy.git)

