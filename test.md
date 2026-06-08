# 🔮 HAWIYAT-ENIGMA VIBECODING TEST — CHALLENGE v3.0
## Full-Stack + DevOps Edition

> **Feed this entire document to the LLM under test. Nothing else. Observe.**

---

## CONTEXT & MISSION

You are a senior full-stack engineer, UI/UX engineer, and DevOps engineer. Your task is to build **"Hawiyat Hub"** — a full-stack multi-tenant productivity platform — from scratch, deploy it to production on AWS, and configure the entire infrastructure stack.

**Three phases:**
1. **Phase 1** — Build the application (Next.js + Kanban + Auth + RBAC + Notifications)
2. **Phase 2** — Deploy to AWS (EC2, security, Cloudflare, DNS)
3. **Phase 3** — Scale to cluster (k3s, Traefik, monitoring, CI/CD notifications)

This is a timed full-stack + infrastructure engineering challenge. Quality, completeness, and attention to detail matter. Think before you code. Design before you deploy.

---

## PHASE 1 — APPLICATION

### TECH STACK — MANDATORY

| Layer | Technology | Version |
|---|---|---|
| Framework | Next.js (App Router) | 15.x |
| Language | TypeScript | 5.x |
| UI Components | shadcn/ui | latest |
| Styling | Tailwind CSS | v4 |
| Database | SQLite via `better-sqlite3` | latest |
| ORM / Query | Direct `better-sqlite3` (no Prisma/Drizzle) | — |
| Icons | `lucide-react` | latest |
| Font | Plus Jakarta Sans (Google Fonts / `next/font`) | — |
| Form Validation | `zod` + `react-hook-form` | latest |
| State | React built-ins (`useState`, `useOptimistic`) | — |
| Theme | `next-themes` | latest |
| Auth | NextAuth.js / Auth.js | latest |
| Notifications | `resend` (email) + `twilio` or `whatsapp-web.js` (WhatsApp) | latest |

**No other dependencies are permitted** unless explicitly noted below.

---

### PRODUCT SPECIFICATION

#### App Name: **Hawiyat Hub**
#### Tagline: *"Your workspace. Your team. Your tasks."*

---

### PROJECT STRUCTURE — REQUIRED

```
hawiyat-hub/
├── app/
│   ├── layout.tsx                   # Root layout with font, theme provider
│   ├── page.tsx                     # Landing → redirects to /dashboard
│   ├── auth/
│   │   ├── login/page.tsx           # Login page
│   │   ├── register/page.tsx        # Registration with workspace creation
│   │   └── callback/route.ts        # OAuth callback
│   ├── dashboard/
│   │   └── page.tsx                 # Dashboard with workspace overview
│   ├── board/
│   │   └── page.tsx                 # Main Kanban board page
│   ├── settings/
│   │   ├── page.tsx                 # User profile settings
│   │   ├── workspace/page.tsx       # Workspace settings
│   │   ├── members/page.tsx         # Member management (RBAC)
│   │   └── notifications/page.tsx   # Notification preferences
│   ├── api/
│   │   ├── auth/
│   │   │   └── [...nextauth]/route.ts
│   │   ├── tasks/
│   │   │   ├── route.ts             # GET (list), POST (create)
│   │   │   └── [id]/
│   │   │       └── route.ts         # PATCH (update), DELETE (delete)
│   │   ├── workspaces/
│   │   │   ├── route.ts             # CRUD workspaces
│   │   │   └── [id]/
│   │   │       ├── members/route.ts # Member management
│   │   │       └── invite/route.ts  # Invite users
│   │   └── notifications/
│   │       ├── route.ts             # GET notification settings
│   │       └── send/route.ts        # POST trigger notification
│   ├── invite/
│   │   └── [token]/page.tsx         # Accept workspace invite
│   └── error.tsx                    # Global error boundary
├── components/
│   ├── auth/
│   │   ├── LoginForm.tsx
│   │   ├── RegisterForm.tsx
│   │   └── WorkspaceSwitcher.tsx
│   ├── board/
│   │   ├── KanbanBoard.tsx
│   │   ├── KanbanColumn.tsx
│   │   ├── TaskCard.tsx
│   │   └── TaskForm.tsx
│   ├── layout/
│   │   ├── Header.tsx
│   │   ├── Sidebar.tsx
│   │   └── StatsPanel.tsx
│   ├── notifications/
│   │   ├── NotificationBell.tsx
│   │   ├── EmailSettings.tsx
│   │   └── WhatsAppSettings.tsx
│   ├── rbac/
│   │   ├── RoleBadge.tsx
│   │   ├── MemberList.tsx
│   │   └── InviteDialog.tsx
│   └── ui/                          # shadcn/ui components (auto-generated)
├── lib/
│   ├── db.ts                        # SQLite singleton + schema init
│   ├── actions.ts                   # Next.js Server Actions
│   ├── types.ts                     # Shared TypeScript types
│   ├── auth.ts                      # Auth.js configuration
│   ├── rbac.ts                      # Role/permission definitions & guards
│   ├── notifications/
│   │   ├── email.ts                 # Resend email integration
│   │   ├── whatsapp.ts              # WhatsApp/Twilio integration
│   │   └── index.ts                 # Notification dispatcher
│   └── validations.ts               # Zod schemas for all inputs
├── hooks/
│   ├── useTasks.ts
│   ├── useWorkspace.ts
│   ├── useNotifications.ts
│   └── useRBAC.ts
├── middleware.ts                     # Auth + RBAC middleware
├── public/
├── .env.local
├── components.json
├── next.config.ts
├── tailwind.config.ts
└── package.json
```

---

### CORE FEATURES — YOU MUST BUILD ALL OF THESE

#### 1. 🔐 Authentication & Multi-Workspace

**Auth (Auth.js / NextAuth.js):**
- Email/password authentication with credentials provider
- Magic link sign-in (Resend email transport)
- OAuth: Google and GitHub providers
- Session management with JWT strategy
- Session token includes: `userId`, `workspaceId`, `role`

**Multi-Workspace:**
- Each user can create or join multiple workspaces
- Workspace selector in sidebar (shadcn `Select` or `DropdownMenu`)
- All tasks, members, and settings are scoped per workspace
- Workspace slug used in URL: `/board?workspace=my-team`
- Invite users via email magic link + workspace token
- Workspace roles: `owner`, `admin`, `member`, `viewer`

**Pages:**
- `/auth/login` — Login with email/password or OAuth
- `/auth/register` — Registration creates first workspace automatically
- `/auth/callback` — OAuth callback handler
- `/invite/[token]` — Accept workspace invitation, auto-join

---

#### 2. 👥 RBAC — Role-Based Access Control

**Roles & Permissions:**

| Permission | owner | admin | member | viewer |
|---|---|---|---|---|
| View tasks | ✅ | ✅ | ✅ | ✅ |
| Create tasks | ✅ | ✅ | ✅ | ❌ |
| Edit tasks | ✅ | ✅ | ✅ | ❌ |
| Delete tasks | ✅ | ✅ | ❌ | ❌ |
| Move tasks | ✅ | ✅ | ✅ | ❌ |
| Manage members | ✅ | ✅ | ❌ | ❌ |
| Invite users | ✅ | ✅ | ❌ | ❌ |
| Change roles | ✅ | ❌ | ❌ | ❌ |
| Delete workspace | ✅ | ❌ | ❌ | ❌ |
| Edit workspace settings | ✅ | ✅ | ❌ | ❌ |
| Configure notifications | ✅ | ✅ | ❌ | ❌ |
| Export data | ✅ | ✅ | ✅ | ❌ |

**Implementation:**
- `lib/rbac.ts` — Permission map and guard functions
- `middleware.ts` — Route protection with role checks
- Server Action guards: each action checks `session.workspaceId` + `session.role`
- UI guards: conditional rendering based on user role
- `useRBAC()` hook for client-side permission checks
- Member management page with role dropdown (shadcn `Select`)
- Invite dialog with role selection

---

#### 3. 📋 Kanban Task Board (Enhanced)

A three-column Kanban board scoped to the active workspace:
- **Backlog**, **In Progress**, **Done**

**Task Card:**
- Title (required, max 60 chars)
- Description (optional, max 200 chars)
- Priority badge: `LOW` / `MEDIUM` / `HIGH` / `CRITICAL`
- Assignee avatar (if multi-user workspace)
- Due date with overdue indicator
- Unique task ID per workspace: `WS-001`, `WS-002` (where WS = workspace prefix)
- Created timestamp (relative time)
- Colored left-border by priority
- Edit/Delete/Move dropdown (respects RBAC)

**Board interactions:**
- "+ Add Task" per column
- "Move to →" dropdown (shadcn `DropdownMenu`)
- Delete with confirmation (shadcn `AlertDialog`)
- Edit via pencil icon (shadcn `Dialog`)
- Double-click title inline edit (contenteditable)
- Assigned user selector (multi-user workspaces)

---

#### 4. ➕ Task Creation / Edit Modal

shadcn `Dialog` with:
- Title input (zod validated)
- Description textarea
- Priority selector (color-coded shadcn `Select`)
- Assignee selector (workspace members, if multi-user)
- Due date picker (shadcn `Popover` + `Calendar`)
- Cancel / Save buttons
- Escape to close, backdrop click to close
- Edit mode pre-fills all fields

---

#### 5. 🔍 Search & Filter Bar

Persistent bar with:
- **Live search** — debounced 150ms, case-insensitive, filters by title/description/assignee
- **Priority filter** — shadcn `Select` (All, Low, Medium, High, Critical)
- **Column filter** — shadcn `Select` (All Columns, Backlog, In Progress, Done)
- **Assignee filter** — shadcn `Select` (workspace members, shown only in multi-user)
- **"Clear Filters"** button (ghost variant)
- Live count badge: `"Showing 4 of 9 tasks"`

---

#### 6. 📊 Stats Dashboard

Collapsible panel (shadcn `Collapsible`) below header:
- Total tasks count
- Tasks per column with % progress bars
- Overdue tasks count (destructive color if > 0)
- High + Critical tasks count
- Completion rate progress bar: Done / Total
- Tasks per assignee (multi-user)
- Panel open/close persisted in `localStorage`

---

#### 7. 🌗 Dark / Light Mode Toggle

- Toggle button in header (moon/sun `lucide-react`)
- `next-themes` with OS preference default
- Persist in `localStorage`
- Smooth `transition-colors duration-200`
- No flash (SSR-safe via `suppressHydrationWarning`)

---

#### 8. 📬 Email & WhatsApp Notifications

**Email (Resend):**
- Welcome email on registration
- Workspace invite email with magic link
- Task assigned notification
- Task @mention notification
- Daily digest (tasks due today, overdue tasks)
- Build error notifications (Phase 3 integration)

**WhatsApp (Twilio API / whatsapp-web.js):**
- Task assigned notification
- Task due reminder (24h before due date)
- Task overdue alert
- Build failure alert (Phase 3 integration)

**Notification Settings Page (`/settings/notifications`):**
- Toggle each notification type on/off
- Configure WhatsApp number
- Configure email preference
- Per-workspace notification rules

**Notification Schema:**
```sql
CREATE TABLE IF NOT EXISTS notification_settings (
  id            INTEGER PRIMARY KEY AUTOINCREMENT,
  workspace_id  TEXT NOT NULL,
  user_id       TEXT NOT NULL,
  email_enabled INTEGER DEFAULT 1,
  whatsapp_enabled INTEGER DEFAULT 0,
  whatsapp_number TEXT DEFAULT '',
  notify_on_assign INTEGER DEFAULT 1,
  notify_on_due     INTEGER DEFAULT 1,
  notify_on_mention INTEGER DEFAULT 1,
  notify_on_overdue INTEGER DEFAULT 1,
  notify_build_errors INTEGER DEFAULT 1,
  FOREIGN KEY (workspace_id) REFERENCES workspaces(id),
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

---

#### 9. 💾 Data Persistence — SQLite via Server Actions

**Database Schema** (`lib/db.ts`):

```sql
CREATE TABLE IF NOT EXISTS users (
  id            TEXT PRIMARY KEY,
  email         TEXT NOT NULL UNIQUE,
  name          TEXT,
  avatar_url    TEXT,
  created_at    TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE TABLE IF NOT EXISTS accounts (
  id            INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id       TEXT NOT NULL,
  provider      TEXT NOT NULL,
  provider_account_id TEXT NOT NULL,
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE IF NOT EXISTS workspaces (
  id            TEXT PRIMARY KEY,
  name          TEXT NOT NULL,
  slug          TEXT NOT NULL UNIQUE,
  owner_id      TEXT NOT NULL,
  created_at    TEXT NOT NULL DEFAULT (datetime('now')),
  FOREIGN KEY (owner_id) REFERENCES users(id)
);

CREATE TABLE IF NOT EXISTS workspace_members (
  id            INTEGER PRIMARY KEY AUTOINCREMENT,
  workspace_id  TEXT NOT NULL,
  user_id       TEXT NOT NULL,
  role          TEXT NOT NULL DEFAULT 'member',
  invited_at    TEXT NOT NULL DEFAULT (datetime('now')),
  joined_at     TEXT,
  FOREIGN KEY (workspace_id) REFERENCES workspaces(id),
  FOREIGN KEY (user_id) REFERENCES users(id),
  UNIQUE(workspace_id, user_id)
);

CREATE TABLE IF NOT EXISTS tasks (
  id            INTEGER PRIMARY KEY AUTOINCREMENT,
  task_id       TEXT NOT NULL,
  workspace_id  TEXT NOT NULL,
  title         TEXT NOT NULL,
  description   TEXT DEFAULT '',
  priority      TEXT NOT NULL DEFAULT 'MEDIUM',
  status        TEXT NOT NULL DEFAULT 'backlog',
  assignee_id   TEXT,
  due_date      TEXT,
  created_at    TEXT NOT NULL DEFAULT (datetime('now')),
  updated_at    TEXT NOT NULL DEFAULT (datetime('now')),
  FOREIGN KEY (workspace_id) REFERENCES workspaces(id),
  FOREIGN KEY (assignee_id) REFERENCES users(id),
  UNIQUE(task_id, workspace_id)
);

CREATE TABLE IF NOT EXISTS notification_settings (
  id            INTEGER PRIMARY KEY AUTOINCREMENT,
  workspace_id  TEXT NOT NULL,
  user_id       TEXT NOT NULL,
  email_enabled INTEGER DEFAULT 1,
  whatsapp_enabled INTEGER DEFAULT 0,
  whatsapp_number TEXT DEFAULT '',
  notify_on_assign INTEGER DEFAULT 1,
  notify_on_due     INTEGER DEFAULT 1,
  notify_on_mention INTEGER DEFAULT 1,
  notify_on_overdue INTEGER DEFAULT 1,
  notify_build_errors INTEGER DEFAULT 1,
  FOREIGN KEY (workspace_id) REFERENCES workspaces(id),
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**Server Actions** (`lib/actions.ts`):
- `createTask(data)` — validates RBAC, inserts, returns full task
- `updateTask(id, data)` — patches, validates permissions
- `deleteTask(id)` — soft/hard delete with permission check
- `moveTask(id, status)` — updates status
- `inviteUser(email, workspaceId, role)` — creates invite, sends email
- `acceptInvite(token)` — joins workspace
- `updateMemberRole(workspaceId, userId, role)` — owner only
- `removeMember(workspaceId, userId)` — admin+
- `updateNotificationSettings(settings)` — saves per-user config
- `sendTestNotification(type)` — sends test email/WhatsApp
- All actions: `"use server"`, zod validated, `revalidatePath()`

---

### UI / UX REQUIREMENTS

Same design system as v2.0 with these additions:

- **Loading states**: Skeleton components for board, member list, settings
- **Empty states**: Per-workspace empty states, "No members yet" state
- **Error boundaries**: Per-workspace error handling
- **Responsive sidebar**: Collapsible on mobile, persistent on desktop

---

### ACCESSIBILITY

Same WCAG AA requirements as v2.0 plus:
- All RBAC-controlled elements have `aria-disabled` when action is denied
- Notification toasts announced via `aria-live="polite"`
- Workspace switcher has proper `role="listbox"` pattern
- Invite flow respects tab order and focus management

---

## PHASE 2 — AWS INFRASTRUCTURE & CLOUDFLARE

### 10. ☁️ AWS EC2 Deployment

#### 10.1 Prerequisites
- AWS account with admin access
- AWS CLI configured locally
- SSH key pair created (`hawiyat-kp.pem`)
- Domain `hawiyat.cloud` registered (Route53 or external registrar)

#### 10.2 EC2 Instance — Detailed Configuration

| Parameter | Value |
|---|---|
| **AMI** | Ubuntu 24.04 LTS (HVM, SSD) — `ami-0e86e20dae9224db8` (us-east-1) |
| **Instance Type** | `t3.medium` (2 vCPU, 4 GiB RAM) |
| **Storage** | 20 GB gp3 root volume + 30 GB gp3 data volume |
| **Network** | Default VPC, public subnet |
| **Security Group** | See firewall rules below |
| **Key Pair** | `hawiyat-kp` |
| **User Data** | Bootstrap script (see below) |
| **IAM Role** | `hawiyat-ec2-role` with SSM + S3 read access |
| **Monitoring** | CloudWatch detailed monitoring enabled |
| **Termination Protection** | Enabled |

#### 10.3 Security Group — Firewall Rules

| Type | Protocol | Port | Source | Description |
|---|---|---|---|---|
| SSH | TCP | 22 | `195.201.0.0/16` (Cloudflare WARP) | Admin SSH |
| HTTP | TCP | 80 | `0.0.0.0/0` | HTTP redirect to HTTPS |
| HTTPS | TCP | 443 | `0.0.0.0/0` | Traefik/Next.js |
| K3s API | TCP | 6443 | `10.0.0.0/8` | Kubernetes API (internal) |
| K3s Flannel | UDP | 8472 | `10.0.0.0/8` | Flannel VXLAN (internal) |
| K3s Metrics | TCP | 10250 | `10.0.0.0/8` | Kubelet metrics (internal) |
| NodePort | TCP | 30000-32767 | `10.0.0.0/8` | Kubernetes NodePort (internal) |
| Traefik Admin | TCP | 8080 | `195.201.0.0/16` | Traefik dashboard |
| Cloudflare IPs | TCP | 443 | `173.245.48.0/20, 103.21.244.0/22, ...` | Cloudflare origin pull |

> **Note**: Apply full list of Cloudflare origin IP ranges from https://www.cloudflare.com/ips-v4

#### 10.4 EC2 User Data — Bootstrap Script

```bash
#!/bin/bash
set -e

# System updates
apt-get update && apt-get upgrade -y
apt-get install -y curl wget git ufw fail2ban unattended-upgrades

# Install Docker
curl -fsSL https://get.docker.com | bash
systemctl enable docker && systemctl start docker

# Install Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | bash
apt-get install -y nodejs
npm install -g pm2

# Install k3s
curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644

# Configure UFW
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 443/tcp
ufw allow 6443/tcp
ufw --force enable

# Configure fail2ban
cat > /etc/fail2ban/jail.local << 'EOF'
[sshd]
enabled = true
port = 22
maxretry = 3
bantime = 3600
findtime = 600
EOF
systemctl restart fail2ban

# Mount data volume
mkfs.ext4 /dev/xvdb || true
mkdir -p /data
mount /dev/xvdb /data || true
echo '/dev/xvdb /data ext4 defaults 0 0' >> /etc/fstab

# Clone application
cd /data
git clone https://github.com/Hawiyat-Org/hawiyat-hub.git
cd hawiyat-hub
npm install
npm run build

# Create systemd service for Next.js
cat > /etc/systemd/system/hawiyat-hub.service << 'EOF'
[Unit]
Description=Hawiyat Hub Next.js Application
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/data/hawiyat-hub
ExecStart=/usr/bin/npm run start
Restart=always
RestartSec=10
Environment=PORT=3000
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable hawiyat-hub
systemctl start hawiyat-hub

# Enable automatic security updates
cat > /etc/apt/apt.conf.d/20auto-upgrades << 'EOF'
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
APT::Periodic::AutocleanInterval "7";
EOF

echo "Bootstrap complete. Hawiyat Hub deployed."
```

#### 10.5 AWS CLI Commands — Provisioning

```bash
# Create security group
aws ec2 create-security-group \
  --group-name hawiyat-hub-sg \
  --description "Hawiyat Hub security group" \
  --vpc-id vpc-xxxxxxxx

# Add firewall rules (see table above)
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxx \
  --protocol tcp --port 22 \
  --cidr 195.201.0.0/16

aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxx \
  --protocol tcp --port 80 --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxx \
  --protocol tcp --port 443 --cidr 0.0.0.0/0

# Create and run EC2 instance
aws ec2 run-instances \
  --image-id ami-0e86e20dae9224db8 \
  --instance-type t3.medium \
  --key-name hawiyat-kp \
  --security-group-ids sg-xxxxxxxx \
  --subnet-id subnet-xxxxxxxx \
  --block-device-mappings "[{\"DeviceName\":\"/dev/sda1\",\"Ebs\":{\"VolumeSize\":20,\"VolumeType\":\"gp3\"}},{\"DeviceName\":\"/dev/xvdb\",\"Ebs\":{\"VolumeSize\":30,\"VolumeType\":\"gp3\"}}]" \
  --iam-instance-profile Name=hawiyat-ec2-role \
  --monitoring Enabled=true \
  --disable-api-termination \
  --user-data file://bootstrap.sh \
  --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=hawiyat-hub-master}]"

# Allocate Elastic IP
aws ec2 allocate-address --domain vpc
aws ec2 associate-address \
  --instance-id i-xxxxxxxx \
  --allocation-id eipalloc-xxxxxxxx
```

---

#### 10.6 OS Hardening Checklist

- [ ] Disable root password SSH login (`PermitRootLogin prohibit-password`)
- [ ] Create admin sudo user (`hawiyat-admin`)
- [ ] Configure SSH key-only authentication
- [ ] Install and configure `fail2ban`
- [ ] Enable `ufw` with strict rules
- [ ] Enable automatic security updates (unattended-upgrades)
- [ ] Disable unused services (`cups`, `avahi-daemon`, etc.)
- [ ] Set kernel parameters (`sysctl`): `net.ipv4.tcp_syncookies=1`, `net.ipv4.conf.all.rp_filter=1`
- [ ] Install `rkhunter` and `aide` for intrusion detection
- [ ] Configure `auditd` for system call auditing
- [ ] Set up `logwatch` for daily log summaries
- [ ] Harden SSH config: disable X11 forwarding, set `MaxAuthTries 3`, `ClientAliveInterval 300`
- [ ] Enable `apparmor` profiles

---

### 11. 🌐 Cloudflare — DNS, Proxy & Anti-DDoS

#### 11.1 DNS Records

| Type | Name | Value | Proxy |
|---|---|---|---|
| A | `kanban.hawiyat.cloud` | `<EC2 Elastic IP>` | Proxied (orange cloud) |
| A | `www.kanban.hawiyat.cloud` | `<EC2 Elastic IP>` | Proxied |
| A | `api.kanban.hawiyat.cloud` | `<EC2 Elastic IP>` | Proxied |
| CNAME | `*.kanban.hawiyat.cloud` | `kanban.hawiyat.cloud` | Proxied |
| TXT | `_dmarc.kanban.hawiyat.cloud` | `"v=DMARC1; p=quarantine; rua=mailto:admin@hawiyat.cloud"` | — |
| TXT | `hawiyat.cloud` | `"v=spf1 include:_spf.google.com ~all"` | — |

#### 11.2 Cloudflare Configuration

| Setting | Value |
|---|---|
| **SSL/TLS** | Full (strict) |
| **Minimum TLS Version** | 1.3 |
| **Always Use HTTPS** | ON |
| **HTTP Strict Transport Security (HSTS)** | ON (max-age=31536000, includeSubDomains) |
| **Auto Minify** | ON (HTML, CSS, JS) |
| **Brotli** | ON |
| **Early Hints** | ON |
| **HTTP/2** | ON |
| **HTTP/3 (QUIC)** | ON |
| **0-RTT Connection Resumption** | ON |
| **Orange-to-Cloud IP** | ON (restrict origin access to Cloudflare IPs only) |

#### 11.3 DDoS Protection — Cloudflare Settings

| Protection | Setting |
|---|---|
| **DDoS Managed Ruleset** | ON (high sensitivity) |
| **Rate Limiting** | 120 requests / 10 seconds per IP |
| **WAF (Web Application Firewall)** | ON (Core Ruleset, OWASP Paranoia Level 2) |
| **Bot Fight Mode** | ON |
| **Security Level** | Medium (challenge on suspicious) |
| **Challenge Passage** | 30 minutes |
| **Browser Integrity Check** | ON |
| **IP Access Rules** | Block all non-Cloudflare IPs to origin |
| **User Agent Blocking** | Block known bad bots |

#### 11.4 Origin Server — Restrict to Cloudflare Only

On the EC2 instance, configure the web server to only accept requests from Cloudflare IPs:

```bash
# Fetch Cloudflare IP ranges
curl -s https://www.cloudflare.com/ips-v4 -o /tmp/cf-ips-v4.txt
curl -s https://www.cloudflare.com/ips-v6 -o /tmp/cf-ips-v6.txt

# Configure UFW to allow only Cloudflare to ports 80/443
ufw delete allow 80/tcp
ufw delete allow 443/tcp
while read ip; do ufw allow from "$ip" to any port 80 proto tcp; done < /tmp/cf-ips-v4.txt
while read ip; do ufw allow from "$ip" to any port 443 proto tcp; done < /tmp/cf-ips-v4.txt
while read ip; do ufw allow from "$ip" to any port 80 proto tcp; done < /tmp/cf-ips-v6.txt
while read ip; do ufw allow from "$ip" to any port 443 proto tcp; done < /tmp/cf-ips-v6.txt
```

---

## PHASE 3 — K3S CLUSTER & TRAEFIK

### 12. 🔄 K3s Cluster — Master + Worker Nodes

#### 12.1 Architecture

```
                  Internet
                     |
                [Cloudflare]
                     |
               [Traefik LB]
                     |
          [k3s Master Node]  ← t3.medium (existing EC2, repurposed)
              /          \
     [k3s Worker 1]   [k3s Worker 2]  ← t3.small each
```

#### 12.2 Provision Worker Nodes

```bash
# After Phase 2 EC2 is verified working, create two additional instances
aws ec2 run-instances \
  --image-id ami-0e86e20dae9224db8 \
  --instance-type t3.small \
  --key-name hawiyat-kp \
  --security-group-ids sg-xxxxxxxx \
  --subnet-id subnet-xxxxxxxx \
  --block-device-mappings "[{\"DeviceName\":\"/dev/sda1\",\"Ebs\":{\"VolumeSize\":20,\"VolumeType\":\"gp3\"}}]" \
  --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=hawiyat-hub-worker-1}]"

aws ec2 run-instances \
  --image-id ami-0e86e20dae9224db8 \
  --instance-type t3.small \
  --key-name hawiyat-kp \
  --security-group-ids sg-xxxxxxxx \
  --subnet-id subnet-xxxxxxxx \
  --block-device-mappings "[{\"DeviceName\":\"/dev/sda1\",\"Ebs\":{\"VolumeSize\":20,\"VolumeType\":\"gp3\"}}]" \
  --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=hawiyat-hub-worker-2}]"
```

#### 12.3 Install k3s on Master Node

```bash
# On master node (existing EC2)
curl -sfL https://get.k3s.io | sh -s - server \
  --write-kubeconfig-mode 644 \
  --node-name master-1 \
  --tls-san kanban.hawiyat.cloud \
  --disable traefik \
  --disable local-storage \
  --disable servicelb \
  --cluster-cidr 10.42.0.0/16 \
  --service-cidr 10.43.0.0/16 \
  --cluster-domain cluster.local \
  --db-path /data/k3s/db \
  --etcd-s3 \
  --etcd-s3-bucket hawiyat-k3s-backup \
  --etcd-s3-region us-east-1 \
  --etcd-s3-folder backups \
  --kube-apiserver-arg "--request-timeout=300s"

# Get node token (needed for workers)
cat /var/lib/rancher/k3s/server/node-token
```

#### 12.4 Join Workers to Cluster

```bash
# On each worker node
curl -sfL https://get.k3s.io | K3S_URL=https://<master-private-ip>:6443 \
  K3S_TOKEN=<node-token> \
  sh -s - agent \
  --node-name worker-1

# Verify cluster
kubectl get nodes
kubectl get pods --all-namespaces
```

#### 12.5 Node Labels & Taints

```bash
# Label nodes
kubectl label node master-1 node-role.kubernetes.io/master=true
kubectl label node worker-1 node-role.kubernetes.io/worker=true
kubectl label node worker-2 node-role.kubernetes.io/worker=true

# Taint master to reserve for system workloads
kubectl taint nodes master-1 node-role.kubernetes.io/master=true:NoSchedule
```

---

### 13. 🚦 Traefik — Ingress Controller & Proxy

#### 13.1 Install Traefik via Helm

```bash
# Add Traefik Helm repo
helm repo add traefik https://traefik.github.io/charts
helm repo update

# Create values file
cat > traefik-values.yaml << 'EOF'
deployment:
  replicas: 2
  kind: Deployment
ports:
  web:
    port: 80
    expose: true
    exposedPort: 80
    redirectTo: websecure
  websecure:
    port: 443
    expose: true
    exposedPort: 443
    tls:
      enabled: true
      certResolver: le
  traefik:
    port: 8080
    expose: true
    exposedPort: 8080
ingressRoute:
  dashboard:
    enabled: true
    matchRule: Host(`traefik.kanban.hawiyat.cloud`)
    middlewares:
      - auth-basic
providers:
  kubernetesCRD:
    enabled: true
  kubernetesIngress:
    enabled: true
    publishedService:
      enabled: true
certificatesResolvers:
  le:
    acme:
      email: admin@hawiyat.cloud
      storage: /data/traefik/acme.json
      dnsChallenge:
        provider: cloudflare
        resolvers:
          - "1.1.1.1:53"
          - "8.8.8.8:53"
      httpChallenge:
        entryPoint: web
metrics:
  prometheus:
    addEntryPointsLabels: true
    addServicesLabels: true
accessLog: true
EOF

# Deploy Traefik
helm install traefik traefik/traefik \
  --namespace traefik \
  --create-namespace \
  -f traefik-values.yaml
```

#### 13.2 Traefik Middleware — Auth & Security

```yaml
# middleware-security.yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: security-headers
  namespace: traefik
spec:
  headers:
    frameDeny: true
    contentTypeNosniff: true
    browserXssFilter: true
    referrerPolicy: "strict-origin-when-cross-origin"
    permissionsPolicy: "camera=(), microphone=(), geolocation=()"
    customFrameOptionsValue: "DENY"
    customResponseHeaders:
      X-Robots-Tag: "noindex, nofollow"
    sslRedirect: true
    stsSeconds: 31536000
    stsIncludeSubdomains: true
    stsPreload: true
---
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: rate-limit
  namespace: traefik
spec:
  rateLimit:
    average: 120
    burst: 200
    period: 10s
    sourceCriterion:
      ipStrategy:
        depth: 1
---
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: compress
  namespace: traefik
spec:
  compress:
    excludedContentTypes:
      - text/event-stream
---
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: ip-whitelist
  namespace: traefik
spec:
  ipWhiteList:
    sourceRange:
      - "173.245.48.0/20"
      - "103.21.244.0/22"
      - "103.22.200.0/22"
      - "103.31.4.0/22"
      - "141.101.64.0/18"
      - "108.162.192.0/18"
      - "190.93.240.0/20"
      - "188.114.96.0/20"
      - "197.234.240.0/22"
      - "198.41.128.0/17"
      - "162.158.0.0/15"
      - "104.16.0.0/13"
      - "104.24.0.0/14"
      - "172.64.0.0/13"
      - "131.0.72.0/22"
```

#### 13.3 IngressRoute — Hawiyat Hub

```yaml
# ingress-hawiyat-hub.yaml
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: hawiyat-hub
  namespace: default
spec:
  entryPoints:
    - websecure
  routes:
    - kind: Rule
      match: Host(`kanban.hawiyat.cloud`) && PathPrefix(`/`)
      services:
        - name: hawiyat-hub
          port: 3000
      middlewares:
        - name: security-headers
          namespace: traefik
        - name: rate-limit
          namespace: traefik
        - name: compress
          namespace: traefik
        - name: ip-whitelist
          namespace: traefik
  tls:
    certResolver: le
    options:
      name: tls-options
      namespace: traefik
---
apiVersion: traefik.io/v1alpha1
kind: TLSOption
metadata:
  name: tls-options
  namespace: traefik
spec:
  minVersion: VersionTLS13
  cipherSuites:
    - TLS_AES_128_GCM_SHA256
    - TLS_AES_256_GCM_SHA384
    - TLS_CHACHA20_POLY1305_SHA256
  sniStrict: true
```

#### 13.4 Deploy Hawiyat Hub on K3s

```yaml
# deployment-hawiyat-hub.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hawiyat-hub
  namespace: default
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: hawiyat-hub
  template:
    metadata:
      labels:
        app: hawiyat-hub
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                        - hawiyat-hub
                topologyKey: kubernetes.io/hostname
      containers:
        - name: hawiyat-hub
          image: ghcr.io/hawiyat-org/hawiyat-hub:latest
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: "production"
            - name: DATABASE_PATH
              value: "/data/hawiyat.db"
            - name: AUTH_SECRET
              valueFrom:
                secretKeyRef:
                  name: hawiyat-secrets
                  key: auth-secret
            - name: RESEND_API_KEY
              valueFrom:
                secretKeyRef:
                  name: hawiyat-secrets
                  key: resend-api-key
            - name: TWILIO_ACCOUNT_SID
              valueFrom:
                secretKeyRef:
                  name: hawiyat-secrets
                  key: twilio-account-sid
            - name: TWILIO_AUTH_TOKEN
              valueFrom:
                secretKeyRef:
                  name: hawiyat-secrets
                  key: twilio-auth-token
            - name: TWILIO_WHATSAPP_NUMBER
              valueFrom:
                secretKeyRef:
                  name: hawiyat-secrets
                  key: twilio-whatsapp-number
          resources:
            requests:
              cpu: 250m
              memory: 256Mi
            limits:
              cpu: 500m
              memory: 512Mi
          livenessProbe:
            httpGet:
              path: /api/health
              port: 3000
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /api/health
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5
          volumeMounts:
            - name: data
              mountPath: /data
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: hawiyat-data-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: hawiyat-hub
  namespace: default
spec:
  selector:
    app: hawiyat-hub
  ports:
    - port: 3000
      targetPort: 3000
  type: ClusterIP
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: hawiyat-data-pvc
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: local-path
```

#### 13.5 Horizontal Pod Autoscaler

```yaml
# hpa-hawiyat-hub.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hawiyat-hub
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hawiyat-hub
  minReplicas: 3
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

---

### 14. 🔔 Build Error Notifications

#### 14.1 CI/CD Pipeline — GitHub Actions + Notification

```yaml
# .github/workflows/deploy.yml
name: Deploy Hawiyat Hub

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: hawiyat-org/hawiyat-hub

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Install dependencies
        run: npm ci

      - name: Type check
        run: npx tsc --noEmit

      - name: Lint
        run: npm run lint

      - name: Build
        run: npm run build

      - name: Log in to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }},${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest

      - name: Deploy to K3s
        run: |
          kubectl set image deployment/hawiyat-hub hawiyat-hub=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
          kubectl rollout status deployment/hawiyat-hub

      - name: Notify — Success
        if: success()
        run: |
          curl -X POST https://api.resend.com/emails \
            -H "Authorization: Bearer ${{ secrets.RESEND_API_KEY }}" \
            -H "Content-Type: application/json" \
            -d '{
              "from": "deploy@hawiyat.cloud",
              "to": ["team@hawiyat.cloud"],
              "subject": "✅ Hawiyat Hub Deployment Successful",
              "html": "<h2>Deployment Complete</h2><p>Commit: ${{ github.sha }}</p><p>Branch: ${{ github.ref_name }}</p>"
            }'

      - name: Notify — Build Failure (Email + WhatsApp)
        if: failure()
        run: |
          # Email notification
          curl -X POST https://api.resend.com/emails \
            -H "Authorization: Bearer ${{ secrets.RESEND_API_KEY }}" \
            -H "Content-Type: application/json" \
            -d '{
              "from": "deploy@hawiyat.cloud",
              "to": ["team@hawiyat.cloud"],
              "subject": "❌ Hawiyat Hub Build Failed",
              "html": "<h2>Build Failed</h2><p>Commit: ${{ github.sha }}</p><p>Branch: ${{ github.ref_name }}</p><p>Author: ${{ github.actor }}</p><a href=\"${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}\">View Logs</a>"
            }'

          # WhatsApp notification via Twilio
          curl -X POST https://api.twilio.com/2010-04-01/Accounts/${{ secrets.TWILIO_ACCOUNT_SID }}/Messages.json \
            -u "${{ secrets.TWILIO_ACCOUNT_SID }}:${{ secrets.TWILIO_AUTH_TOKEN }}" \
            -d "From=whatsapp:${{ secrets.TWILIO_WHATSAPP_NUMBER }}" \
            -d "To=whatsapp:${{ secrets.DEV_OPS_WHATSAPP }}" \
            -d "Body=❌ Hawiyat Hub build FAILED on ${{ github.ref_name }} by ${{ github.actor }}. View: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
```

#### 14.2 Build Error Webhook — Internal API

```typescript
// app/api/notifications/build-error/route.ts
import { NextResponse } from 'next/server';
import { sendEmail } from '@/lib/notifications/email';
import { sendWhatsApp } from '@/lib/notifications/whatsapp';
import { db } from '@/lib/db';

export async function POST(request: Request) {
  try {
    const { project, branch, commit, author, status, logs } = await request.json();

    // Fetch all users with build error notifications enabled
    const subscribers = db.prepare(`
      SELECT u.email, ns.whatsapp_number
      FROM notification_settings ns
      JOIN users u ON u.id = ns.user_id
      WHERE ns.notify_build_errors = 1
        AND ns.workspace_id = ?
    `).all(project);

    const subject = status === 'success'
      ? `✅ Build succeeded: ${project}/${branch}`
      : `❌ Build FAILED: ${project}/${branch}`;

    const html = `
      <h2>Build ${status === 'success' ? 'Succeeded' : 'Failed'}</h2>
      <p><strong>Project:</strong> ${project}</p>
      <p><strong>Branch:</strong> ${branch}</p>
      <p><strong>Commit:</strong> ${commit}</p>
      <p><strong>Author:</strong> ${author}</p>
      ${logs ? `<pre>${logs}</pre>` : ''}
    `;

    for (const subscriber of subscribers) {
      await sendEmail({
        to: subscriber.email,
        subject,
        html,
      });

      if (subscriber.whatsapp_number) {
        await sendWhatsApp({
          to: subscriber.whatsapp_number,
          message: `${subject}\nCommit: ${commit}\nAuthor: ${author}`,
        });
      }
    }

    return NextResponse.json({ sent: subscribers.length });
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to send build notifications' },
      { status: 500 }
    );
  }
}
```

---

### 15. 📁 K3s Complete YAML Templates

All templates below go in `/ops/k3s/`:

```
ops/k3s/
├── traefik/
│   ├── traefik-values.yaml          # Helm values
│   ├── middleware-security.yaml     # Security headers, rate limit, compression
│   └── tls-options.yaml            # TLS 1.3 configuration
├── hawiyat-hub/
│   ├── deployment.yaml              # Deployment + Service + PVC
│   ├── ingressroute.yaml            # Traefik IngressRoute
│   ├── hpa.yaml                     # Horizontal Pod Autoscaler
│   ├── secrets.yaml                 # Encrypted secrets (SOPS)
│   └── configmap.yaml              # Environment config
├── monitoring/
│   ├── prometheus-stack.yaml        # kube-prometheus-stack Helm values
│   ├── grafana-dashboard.yaml       # Custom Grafana dashboard
│   └── alertmanager-config.yaml     # Alertmanager rules
├── storage/
│   ├── storage-class.yaml           # Local path provisioner
│   └── backup-cronjob.yaml          # etcd + DB backup CronJob
└── cluster/
    ├ namespace.yaml                 # All namespaces
    ├ rbac-cluster.yaml              # ClusterRole bindings
    └ resource-quotas.yaml           # Per-namespace quotas
```

#### 15.1 Monitoring Stack

```yaml
# ops/k3s/monitoring/prometheus-stack.yaml
prometheus:
  prometheusSpec:
    retention: 15d
    retentionSize: 50GB
    resources:
      requests:
        cpu: 200m
        memory: 1Gi
    replicas: 1
    ruleSelectorNilUsesHelmValues: false
    serviceMonitorSelectorNilUsesHelmValues: false

grafana:
  adminPassword: admin
  ingress:
    enabled: true
    hosts:
      - monitoring.kanban.hawiyat.cloud
    annotations:
      kubernetes.io/ingress.class: traefik
  dashboardProviders:
    dashboardproviders.yaml:
      apiVersion: 1
      providers:
        - name: default
          orgId: 1
          folder: ""
          type: file
          disableDeletion: false
          editable: true
          options:
            path: /var/lib/grafana/dashboards/default
  dashboards:
    default:
      hawiyat-hub:
        json: |
          { "title": "Hawiyat Hub Dashboard", "panels": [...] }

alertmanager:
  config:
    global:
      resolve_timeout: 5m
    route:
      receiver: default
      routes:
        - match:
            severity: critical
          receiver: critical
    receivers:
      - name: default
        email_configs:
          - to: team@hawiyat.cloud
            from: alert@hawiyat.cloud
            smarthost: smtp.resend.com:587
            auth_username: "{{ RESEND_SMTP_USER }}"
            auth_password: "{{ RESEND_SMTP_PASSWORD }}"
      - name: critical
        webhook_configs:
          - url: "http://hawiyat-hub:3000/api/notifications/build-error"
            send_resolved: true
```

#### 15.2 Backup CronJob

```yaml
# ops/k3s/storage/backup-cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hawiyat-backup
  namespace: default
spec:
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: backup
              image: alpine:3.19
              env:
                - name: AWS_ACCESS_KEY_ID
                  valueFrom:
                    secretKeyRef:
                      name: aws-backup-credentials
                      key: access-key-id
                - name: AWS_SECRET_ACCESS_KEY
                  valueFrom:
                    secretKeyRef:
                      name: aws-backup-credentials
                      key: secret-access-key
              command:
                - /bin/sh
                - -c
                - |
                  apk add --no-cache aws-cli sqlite
                  cp /data/hawiyat.db /tmp/hawiyat-$(date +%Y%m%d-%H%M%S).db
                  gzip /tmp/hawiyat-*.db
                  aws s3 cp /tmp/hawiyat-*.db.gz s3://hawiyat-backups/db/
                  kubectl exec -n default deploy/hawiyat-hub -- \
                    sh -c "k3s etcd-snapshot save --s3 --s3-bucket=hawiyat-k3s-backup"
              volumeMounts:
                - name: data
                  mountPath: /data
          restartPolicy: OnFailure
          volumes:
            - name: data
              persistentVolumeClaim:
                claimName: hawiyat-data-pvc
```

---

### 16. ✅ CLUSTER VERIFICATION CHECKLIST

After all three nodes are provisioned and k3s + Traefik are configured:

- [ ] `kubectl get nodes` — all 3 nodes `Ready`
- [ ] `kubectl get pods -n traefik` — Traefik pods running
- [ ] `kubectl get pods -n default` — hawiyat-hub pods running
- [ ] `kubectl get ingressroute` — ingress configured
- [ ] `curl -I https://kanban.hawiyat.cloud` — returns 200
- [ ] `kubectl get hpa` — autoscaler configured
- [ ] `kubectl get cronjob` — backup cronjob active
- [ ] Cloudflare dashboard — DNS proxied (orange cloud)
- [ ] Cloudflare dashboard — WAF enabled, rate limiting active
- [ ] `ufw status` — correct firewall rules on all nodes
- [ ] `kubectl logs -n traefik deploy/traefik` — no TLS errors
- [ ] GitHub Actions — deploy workflow runs successfully
- [ ] WhatsApp notification received on first deploy
- [ ] Email notification received on first deploy
- [ ] Fail2ban active (`fail2ban-client status sshd`)
- [ ] `etcd-snapshot` tested and restorable

---

## DELIVERABLE

A complete Next.js project + infrastructure config that:

1. Runs with `npm install && npm run dev` — no additional setup
2. Initializes SQLite + Auth + RBAC automatically
3. Seeds sample data if no tasks exist
4. Passes `npx tsc --noEmit` with zero TypeScript errors
5. Has no `eslint` warnings
6. Is deployable to AWS EC2 via the provided bootstrap script
7. Has Cloudflare DNS configured at `kanban.hawiyat.cloud`
8. Scales to a 3-node k3s cluster with Traefik ingress
9. Sends Email + WhatsApp notifications on build status
10. Contains complete YAML templates in `ops/k3s/`

---

## EVALUATION CRITERIA — UPDATED

| Category | Weight | What We Look For |
|---|---|---|
| **Functional Completeness** | 15% | All Phase 1 features: Auth, RBAC, Kanban, Workspaces, Notifications |
| **Type Safety & Code Quality** | 10% | No `any`, clean separation, readable naming |
| **UI/UX Polish** | 10% | Design system, responsive, accessible, dark mode, empty/loading states |
| **Database & Server Actions** | 10% | Correct schema, parameterized queries, revalidation, seed data |
| **Auth & RBAC** | 15% | Multi-workspace, role guards, invite flow, session management |
| **Notifications** | 10% | Email + WhatsApp integration, settings UI, build error alerts |
| **AWS Deployment** | 10% | EC2 provisioned, bootstrapped, secured, firewall hardened |
| **Cloudflare Configuration** | 5% | DNS, proxy, anti-DDoS, origin restriction, HSTS, SSL/TLS |
| **K3s Cluster** | 10% | 3-node cluster, master/workers, labels, taints, HPA |
| **Traefik Ingress** | 5% | TLS 1.3, security middleware, rate limiting, Let's Encrypt |
| **Bonus Features** | 5% | Drag-and-drop, undo delete, optimistic UI, backup cronjob, monitoring |

---

## SCORING RUBRIC

| Score | Meaning |
|---|---|
| 0–30 | Incomplete or non-running app |
| 31–50 | Phase 1 partially complete (no auth, no RBAC) or Phase 2 missing |
| 51–70 | Phase 1 complete, Phase 2 partially done (EC2 up but no Cloudflare/k3s) |
| 71–85 | All phases functional; minor gaps in security or monitoring |
| 86–100 | Fully deployed, clustered, secured, monitored, with notifications and zero TS errors |

---

**Begin. Build the application. Deploy to AWS. Configure Cloudflare. Scale with k3s. Notify on build.**

