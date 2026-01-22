# Real-Time Chat App with Convex

## What is Convex?
Think of Convex as a **"Reactive Database."** In a normal app, if you want the screen to update when someone else sends a message, you usually have to set up complex WebSockets or polling.
In Convex:
- **No SQL**: You write your database logic in pure TypeScript.
- **Automatic Sync**: If data changes in the database, the UI updates automatically across all users.
- **Serverless**: Your backend code (queries/mutations) lives in the convex/ folder and runs in the cloud automatically.

You can learn more in the Official docs - [Understanding Convex](https://docs.convex.dev/understanding) 

## Setup & Installation
### 1. Clone & Install
Clone your project repository and install the dependencies:

```Bash
git clone https://github.com/get-convex/convex-tutorial.git
cd convex-tutorial
npm install
```
### 2. Authenticate & Initialize
During the setup, Convex will ask you to authenticate using your GitHub account.
- **Sign into Convex**: A browser window will open automatically. Sign in with GitHub and authorize Convex.
- **Accept Defaults**: Return to your terminal and accept the default project setup prompts.
- **Result**: This automatically creates your backend deployment and a folder called convex/ in your project where your backend code lives.

### 3. Run the Development Environment
Run this command and keep it running in the background throughout your development session:

```Bash
npm run dev
```

**Why keep it running?** This command does two things at once:
1. It runs the Frontend web server (Vite) at localhost:5173.
2. It runs the Convex CLI in the background, which watches your convex/ folder and instantly syncs any local changes to your cloud backend.

## ✍️ Mutations (Writing Data)
Mutations are used to change data (e.g., sending a new message). These are transactional, meaning they either succeed completely or fail completely, ensuring your database never gets stuck in a messy or partial state.

### First Mutation
Create a new file in your convex/ folder called chat.ts. This is where you'll write your Convex backend functions for this application.
Add this code to convex/chat.ts

```typescript
import { mutation } from "./_generated/server";
import { v } from "convex/values";

export const sendMessage = mutation({
  args: {
    user: v.string(),
    body: v.string(),
  },
  handler: async (ctx, args) => {
    console.log("This TypeScript function is running on the server.");
    await ctx.db.insert("messages", {
      user: args.user,
      body: args.body,
    });
  },
});
```
Then connect this mutation to your web app.

Update your **src/App.tsx** file like so:
```typescript
// Import `useMutation` and `api` from Convex.
import { useMutation } from "convex/react";
import { api } from "../convex/_generated/api";
```

```typescript
  // Replace the "TODO: Add mutation hook here." with:
  const sendMessage = useMutation(api.chat.sendMessage);
  ```

```typescript
// Replace "alert("Mutation not implemented yet");" with:
          await sendMessage({ user: NAME, body: newMessageText });
```
You'll notice new chat messages showing up live in the messages table in the [Convex Dashboard](https://dashboard.convex.dev).
Convex automatically created a messages table when you sent the first message.



## 📡 Queries (Reading Data)
Queries are used to fetch data. Because Convex is reactive, these are "Live Queries." You don't need to write code to refresh the page; as soon as a new message hits the database, the query pushes it to your UI automatically.

### First Query
Update your convex/chat.ts file like this:

```typescript
// Update your server import like this:
import { query, mutation } from "./_generated/server";
```
```typescript
// Add the following function to the file:
export const getMessages = query({
  args: {},
  handler: async (ctx) => {
    // Get most recent messages first
    const messages = await ctx.db.query("messages").order("desc").take(50);
    // Reverse the list so that it's in a chronological order.
    return messages.reverse();
  },
});
```
Now update **src/App.tsx** to read from your query:

```typescript
// Update your convex/react import like this:
import { useQuery, useMutation } from "convex/react";
```
```typescript
 // Replace the `const messages = ...` line with the following
  const messages = useQuery(api.chat.getMessages);
```
That's it. Now go back to your app and try sending messages.
Your app should be showing live updates as new messages arrive

## 🖥️ The "Live" Test
To see the power of a reactive backend, try this:
1. **Frontend**: Open http://localhost:5173.
2. **Dashboard**: Open your Convex Dashboard and go to the "Data" screen.
3. **The Side-by-Side**: Place your chat app and the dashboard windows side-by-side.
   - Send a message from your chat app.
   - Watch the magic: The record appears in the Dashboard "Data" table almost instantly.
  - Manual Edit: Try deleting or editing a record directly in the Dashboard and watch it disappear or change instantly in your app without a refresh!
## 💡 Pro-Tips for Learners
- **No Refetching**: You don't need useEffect or fetch() calls to get data. Use the useQuery hook, and Convex handles the data lifecycle for you.
- **Logs**: If your backend function isn't working, check the Logs tab in the Convex Dashboard to see the exact error output.
- **The convex/ Folder**: This is effectively your server. Any TypeScript file you put here becomes an API endpoint automatically.

You're just a few minutes away from having a chat app powered by Convex.

Follow the tutorial at
[docs.convex.dev/tutorial](https://docs.convex.dev/tutorial) for instructions.
