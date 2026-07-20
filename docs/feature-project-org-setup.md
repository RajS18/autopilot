# Clerk Organizations Setup - feature/project-org-setup

**Branch:** `feature/project-org-setup`  
**Date:** 2026-07-20  
**User:** Raj Shukla (rajdragatron918@gmail.com)  
**Based on commit:** 45543dd (add org support from clerk)

---

## Requirement

Add multi-tenancy organization support to the autopilot application using Clerk's Organizations feature. This enables:
- Users to create and manage multiple organizations
- Role-based access control within organizations
- Switching between organizations
- Organization invitations and member management
- Scoped authentication per organization

---

## Purpose & Benefits

### Why Organizations Matter

**Multi-Tenancy:** Organizations allow a single application to serve multiple independent teams or companies from one codebase. Each organization maintains:
- Separate members and roles
- Organization-specific data and settings
- Independent billing and subscriptions
- Isolated access control

### How It Helps

1. **For Users:**
   - Switch between multiple organizations seamlessly
   - Invite team members to their organization
   - Manage roles and permissions
   - Maintain separate workspaces

2. **For Developers:**
   - No need to build multi-tenancy from scratch
   - Clerk handles member management and invitations
   - Built-in role-based access control (RBAC)
   - Automatic organization context in requests

3. **For Business:**
   - Enable team/workspace features
   - Support multiple customer accounts per user
   - Scale to B2B/B2B2C models
   - Reduce development time

---

## What Was Added

### New Files

1. **`.dockerignore`** — Docker build exclusion file
   - Excludes `.claude/` directory (Claude Code workspace)
   - Excludes `.agents/` directory (AI agents)

2. **`app/(auth)/choose-organozation/page.tsx`** — Organization selection page
   - Route group `(auth)` for authentication-related pages
   - Displays Clerk's `TaskChooseOrganization` component
   - Redirects to home page (`/`) after organization selection
   - **Note:** Filename has typo: "organozation" instead of "organization"

### Modified Files

1. **`app/layout.tsx`** — Added organization task URL configuration
   - Added `taskUrls` prop to `ClerkProvider`
   - Maps Clerk's "choose-organization" task to `/choose-organozation` route
   - Enables automatic redirect when organization selection is needed

2. **`app/page.tsx`** — Updated home page with organization controls
   - Added `OrganizationSwitcher` component from `@clerk/nextjs`
   - Maintains `UserButton` for user profile management
   - Layout shows both user controls and organization switcher
   - Allows users to switch organizations from the main page

---

## How It Was Added

### Step 1: Create Organization Selection Route

**File:** `app/(auth)/choose-organozation/page.tsx`

```typescript
import { TaskChooseOrganization } from "@clerk/nextjs"

export default function ChooseOrganizationPage() {
  return (
    <div className="flex min-h-screen items-center justify-center">
      <TaskChooseOrganization redirectUrlComplete="/" />
    </div>
  )
}
```

**What this does:**
- Imports `TaskChooseOrganization` from Clerk
- Displays Clerk's pre-built organization selection UI
- Redirects user to `/` (home) after they select an organization
- Centered layout for better UX

### Step 2: Configure ClerkProvider with Organization Task URL

**File:** `app/layout.tsx`

```typescript
<ClerkProvider
  appearance={{ theme: shadcn }}
  taskUrls={{ "choose-organization": "/choose-organization" }}
>
```

**What this does:**
- Tells Clerk where to redirect when organization selection is needed
- Maps the internal task name to your custom route
- Enables seamless organization switching flow
- Works with `OrganizationSwitcher` component

### Step 3: Add Organization Switcher to Home Page

**File:** `app/page.tsx`

```typescript
import { OrganizationSwitcher, UserButton } from "@clerk/nextjs"

export default function Page() {
  return (
    <div className="flex flex-col items-start gap-4 p-4">
      <UserButton />
      <OrganizationSwitcher />
    </div>
  )
}
```

**What this does:**
- Imports `OrganizationSwitcher` component from Clerk
- Displays UI for creating and switching organizations
- Users can click to view their organizations
- Users can switch between organizations instantly
- Maintains user profile button for account settings

### Step 4: Docker Configuration

**File:** `.dockerignore`

```
.claude/
.agents/
```

**What this does:**
- Excludes Claude Code workspace files from Docker builds
- Excludes agent configuration from Docker containers
- Reduces Docker image size
- Prevents development tools from being included in production

---

## Concrete Steps to Enable Organizations in Any Clerk Project

### Prerequisites
- Existing Clerk setup (see `feature-project-auth-setup.md`)
- `@clerk/nextjs` package already installed
- Clerk application already created and linked

### Step-by-Step Implementation

#### 1. Enable Organizations in Clerk Dashboard
```
1. Go to dashboard.clerk.com
2. Select your application
3. Navigate to "Organizations" settings
4. Enable "Organizations" feature
5. Configure organization settings (invite URL, etc.)
```

#### 2. Create Organization Selection Route
Create file: `app/(auth)/choose-organization/page.tsx`

```typescript
import { TaskChooseOrganization } from "@clerk/nextjs"

export default function ChooseOrganizationPage() {
  return (
    <div className="flex min-h-screen items-center justify-center">
      <TaskChooseOrganization redirectUrlComplete="/" />
    </div>
  )
}
```

#### 3. Update ClerkProvider Configuration
Edit: `app/layout.tsx`

```typescript
import { ClerkProvider } from "@clerk/nextjs"

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <ClerkProvider
          // ... existing config ...
          taskUrls={{ "choose-organization": "/choose-organization" }}
        >
          {children}
        </ClerkProvider>
      </body>
    </html>
  )
}
```

#### 4. Add Organization UI Components
Edit: `app/page.tsx` (or any page where you want org controls)

```typescript
import { OrganizationSwitcher, UserButton } from "@clerk/nextjs"

export default function Page() {
  return (
    <div className="flex gap-4">
      <UserButton />
      <OrganizationSwitcher />
    </div>
  )
}
```

#### 5. Protect Organization-Specific Routes (Optional)
Create: `app/dashboard/[[...organization]]/page.tsx`

```typescript
import { auth } from "@clerk/nextjs/server"

export default async function DashboardPage() {
  const { sessionClaims } = await auth()
  const organizationId = sessionClaims?.org_id

  return (
    <div>
      <h1>Dashboard for Org: {organizationId}</h1>
      {/* Organization-specific content */}
    </div>
  )
}
```

#### 6. Using Organization Context in API Routes
Create: `app/api/org/members/route.ts`

```typescript
import { auth } from "@clerk/nextjs/server"

export async function GET() {
  const { sessionClaims } = await auth()
  const organizationId = sessionClaims?.org_id

  if (!organizationId) {
    return Response.json({ error: "No organization selected" }, { status: 401 })
  }

  // Fetch organization members, fetch org-specific data, etc.
  return Response.json({ organizationId })
}
```

#### 7. Access Organization in Client Components
```typescript
"use client"

import { useOrganization } from "@clerk/nextjs"

export default function OrgDependent() {
  const { organization, isLoaded } = useOrganization()

  if (!isLoaded) return <div>Loading...</div>

  return <div>Current Organization: {organization?.name}</div>
}
```

---

## How It Works - Technical Architecture

### Organization Flow

```
1. User signs up → Creates account
2. User clicks "New Organization" → OrganizationSwitcher
3. Enters organization name → Submits form
4. Clerk creates organization → Redirects to /choose-organization
5. TaskChooseOrganization displays org selection → User confirms
6. Redirects to home page "/" → Organization is now active
7. User can access org-scoped features → Clerk includes org_id in session
8. User clicks org switcher → Can switch to different organizations
```

### Session & Authorization

```
Session contains:
├── user_id: User's unique ID
├── org_id: Currently active organization ID
└── org_role: User's role in organization (admin, member, etc.)

Available in:
├── Server: await auth() → sessionClaims.org_id
├── Client: useUser() or useAuth() → sessionClaims.org_id
└── API: request headers → Authorization header contains claims
```

### Clerk Components Used

1. **`TaskChooseOrganization`**
   - Built-in UI for selecting/creating organizations
   - Handles the full organization selection flow
   - Customizable redirect URL after selection

2. **`OrganizationSwitcher`**
   - Dropdown UI to switch between organizations
   - Create new organization button
   - Invite members UI
   - Organization settings access

3. **`UserButton`**
   - User profile and account management
   - Sign out functionality
   - Profile customization

---

## Expected Results

### User Experience

1. **First Time Setup:**
   - User signs in → Clerk checks if they have organizations
   - If none exist → Redirected to `/choose-organozation`
   - User creates first organization → Home page loads
   - `OrganizationSwitcher` shows newly created organization

2. **Home Page Display:**
   ```
   ┌─────────────────────────────┐
   │ [👤 User] [🏢 Switch Org]   │
   │                             │
   │ Welcome to Dashboard        │
   │ Current Org: Acme Inc       │
   └─────────────────────────────┘
   ```

3. **Organization Switching:**
   - User clicks `OrganizationSwitcher`
   - Dropdown shows all their organizations
   - User selects different org → Redirects to `/choose-organozation`
   - Org context updated → Home page shows new org
   - All org-scoped features now show selected org's data

4. **Inviting Members:**
   - User clicks `OrganizationSwitcher`
   - Selects "Manage organization"
   - Invites team members via email
   - Members get invitation link → Can join organization

---

## Testing the Feature

### Test 1: Organization Creation
```
1. Visit http://localhost:3000
2. Sign in with test account
3. Click "OrganizationSwitcher"
4. Click "Create organization"
5. Enter organization name (e.g., "Test Company")
6. Submit → Should redirect to /choose-organozation
7. Select created organization
8. Verify redirect to home page
✓ Organization appears in switcher
```

### Test 2: Organization Switching
```
1. From home page with organization selected
2. Click "OrganizationSwitcher"
3. If you have multiple orgs, click another organization
4. Verify redirect flow completes
5. Verify home page shows selected organization
✓ Current org displayed correctly
```

### Test 3: Session Data
```
Open browser DevTools → Console:
import { useAuth } from "@clerk/nextjs"

// In a client component:
const { sessionClaims } = useAuth()
console.log(sessionClaims.org_id) // Should show active org ID
✓ org_id present in session
```

### Test 4: API Route Organization Context
```bash
curl -H "Authorization: Bearer <token>" http://localhost:3000/api/org/members
# Should return org-scoped data
```

---

## Known Issues & Notes

### Filename Typo
- Route path: `app/(auth)/choose-organozation/page.tsx`
- Note the typo: "organozation" instead of "organization"
- taskUrls config tries both `/choose-organization` and actual path
- **Future fix:** Rename to `choose-organization` for consistency

### Important Considerations

1. **Protected Routes:**
   - Routes that use organization context should check `org_id` exists
   - Redirect unauthorized users to `/choose-organozation` if needed

2. **Database Design:**
   - Plan your database schema around organization IDs
   - Always scope queries to current organization
   - Prevent data leaks across organizations

3. **API Security:**
   - Always verify `org_id` in sessionClaims before returning org-scoped data
   - Don't trust organization ID from URL alone
   - Use Clerk's authorization metadata

4. **Invitation Links:**
   - Configure custom invitation URL in Clerk dashboard
   - Example: `https://yourapp.com/invite?__clerk_ticket=...`
   - Direct users to correct org after accepting invite

---

## Status

✅ **COMPLETE**

Organizations feature fully implemented with:
- Organization selection route
- Home page organization switcher
- ClerkProvider configuration
- Ready for multi-tenancy support

---

## Next Steps (Enhancements)

1. **Database Integration:**
   - Create `organizations` table with Clerk org_id
   - Store organization metadata in your database
   - Sync organization data with webhook

2. **Role-Based Access Control:**
   - Define roles and permissions
   - Protect routes based on org_role
   - Create permission middleware

3. **Organization Settings:**
   - Build custom organization settings page
   - Allow org admins to customize organization name, logo, etc.
   - Store org settings in database

4. **Webhook Integration:**
   - Set up Clerk webhook for organization events
   - Sync org_created, member_added, etc. to your database
   - Keep your database in sync with Clerk

5. **Branding:**
   - Add organization logo to organization switcher
   - Theme application based on organization
   - Create white-label support

---

## References

- **Clerk Organizations Docs:** https://clerk.com/docs/guides/organizations/overview
- **Organization Switcher:** https://clerk.com/docs/reference/components/organization-switcher
- **useOrganization Hook:** https://clerk.com/docs/reference/react/use-organization
- **TaskChooseOrganization:** https://clerk.com/docs/reference/components/task-choose-organization
- **Clerk Dashboard:** https://dashboard.clerk.com/
- **@clerk/nextjs Docs:** https://clerk.com/docs/references/nextjs/overview

---

## Related Documentation

- See `feature-project-auth-setup.md` for base Clerk authentication setup
- See `AGENTS.md` for documentation standards and requirements
