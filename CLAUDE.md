<!-- joyspace-kit start -->
# JoySpace Kit — Claude Code Skill

JoySpace document operations: read, search, list, upload images, and import markdown.

## Prerequisites

- Node.js >= 18
- HiOffice desktop client running (ports 8988-9006) for auth
- Network access to `apijoyspace.jd.com` (internal or VPN)

## Skill Scripts Location

All scripts are under the installed package root (`/Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit`). The `joyspace-kit init` command resolves the path automatically in the generated CLAUDE.md.

---

## 1. joyspace-read — Read a JoySpace Document

**When to use:** User pastes a `joyspace.jd.com/pages/xxx` URL and wants to read, summarize, quote, export, or back up the document. Or gives a pageId directly.

**Command:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyspace-read-doc/scripts/read_joyspace_doc.js --url "https://joyspace.jd.com/pages/<id>"
```

**Options:**

| Flag | Description | Default |
|------|-------------|---------|
| `--url` | Full JoySpace page URL | — |
| `--page-id` | Page id (alternative to --url) | — |
| `--save <path>` | Also write markdown to a local file | — |
| `--raw` | Include raw API responses (debugging) | false |

**Output:** JSON with `title`, `pageType`, `content` (reconstructed markdown), `link`, etc.

**Example — read and summarize:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyspace-read-doc/scripts/read_joyspace_doc.js --url "https://joyspace.jd.com/pages/avIBigHE3WXuUmspPCa6"
```

**Example — save to local file:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyspace-read-doc/scripts/read_joyspace_doc.js --url "https://joyspace.jd.com/pages/xxx" --save ./output.md
```

**Pitfalls:**
- `PAGE NOT EXIST` or `NEED_ACCESS_RIGHT` means the auth token's account lacks permission — not a script bug.
- Only `page_type=13` (markdown) is fully rendered. Sheets/mind maps/boards degrade to inline text; use `--raw` for those.
- First call via HiOffice legacy auth is slow (~1-3s port probe); subsequent calls reuse the token.

---

## 2. joyspace-search — Search JoySpace Documents

**When to use:** User wants to find a JoySpace page by keyword in title or content.

**Command:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyspace-search/scripts/search_joyspace.mjs --query "关键词" [--limit 20] [--json]
```

**Options:**

| Flag | Description | Default |
|------|-------------|---------|
| `--query` / `-q` | Search keyword (>= 3 chars) | required |
| `--limit` | Max results | 20 |
| `--start` | Offset for pagination | 0 |
| `--scope` | `global` or `received` | global |
| `--json` | Output raw JSON | false |

**Example:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyspace-search/scripts/search_joyspace.mjs --query "新品运营" --limit 10 --json
```

---

## 3. joyspace-list — List Recent Pages

**When to use:** User wants to see what they've been working on, find a doc by partial title, or browse recent JoySpace activity.

**Command:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyspace-list/scripts/list_joyspace.mjs [--limit 20] [--grep keyword] [--json]
```

**Options:**

| Flag | Description | Default |
|------|-------------|---------|
| `--limit` | Max results | 20 |
| `--grep` | Filter by title/preview substring | — |
| `--json` | Output raw JSON | false |

**Example:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyspace-list/scripts/list_joyspace.mjs --grep "PRD" --limit 5
```

---

## 4. joyspace-upload-image — Upload Image to JoySpace CDN

**When to use:** User wants to embed a local image into a JoySpace document. JoySpace only supports images by URL (no inline base64), and external image hosts often fail on the office network. This skill uploads to JoySpace's internal CDN.

**Command:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyspace-upload-image/scripts/upload_image.mjs --file /path/to/image.png --auto-scratch --delete-scratch
```

**Options:**

| Flag | Description | Default |
|------|-------------|---------|
| `--file` | Local image path (required) | — |
| `--page-id` | Upload to an existing page | — |
| `--auto-scratch` | Create temp scratch page, upload there | — |
| `--delete-scratch` | Delete scratch page after upload (CDN URL stays valid) | false |

**Output:** JSON with `imgUrl` (the stable CDN URL to use in markdown), `dimensions`.

**Example:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyspace-upload-image/scripts/upload_image.mjs --file ./screenshot.png --auto-scratch --delete-scratch
# Returns: { imgUrl: "https://apijoyspace.jd.com/v1/files/xxx/link", ... }
```

---

## 5. markdown-to-joyspace — Import Markdown as JoySpace Document

**When to use:** User wants to create a JoySpace doc from a `.md` file. Auto-uploads local image references to JoySpace's CDN.

**Command:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/markdown-to-joyspace/scripts/import_markdown_doc.js --file /path/to/doc.md
```

**Options:**

| Flag | Description | Default |
|------|-------------|---------|
| `--file` | Local markdown file (required) | — |
| `--title` | Document title | derived from filename or H1 |
| `--folder-id` | Target folder | personal space |
| `--team-id` | Target team | personal space |

**Example:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/markdown-to-joyspace/scripts/import_markdown_doc.js --file ./PRD.md --title "我的PRD"
```

---

## 6. hioffice-auth — Get Auth Token

**When to use:** Usually called automatically by other scripts. Run manually only for debugging.

**Command:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/hioffice-auth/scripts/hioffice-auth.mjs
```

**Output:** JSON `{ meToken: "ee.xxx...", authMode: "legacy" }`

---

## Common Workflows

### Read a JoySpace URL and summarize
```
User: "帮我看看这个文档 https://joyspace.jd.com/pages/xxx 在说什么"
→ Run joyspace-read with --url
→ Summarize the `content` field
```

### Search for a topic and read the top result
```
User: "搜一下全域通相关的文档"
→ Run joyspace-search --query "全域通"
→ From results, run joyspace-read --page-id <top pageId>
→ Summarize or quote
```

### Import a local markdown file to JoySpace
```
User: "把这个文件发布到JoySpace：./notes.md"
→ Run markdown-to-joyspace --file ./notes.md
→ Return the created doc URL
```

### Upload images and create a doc with screenshots
```
User: "把这个截图发到JoySpace"
→ Run joyspace-upload-image --file screenshot.png --auto-scratch --delete-scratch
→ Get imgUrl from result
→ Optionally create a doc with the image embedded
```

---

## Auth & Permissions

All scripts authenticate via the local HiOffice client. The auth chain:
1. `ME_TOKEN` / `SSO_TOKEN` env vars (if set)
2. `~/.joyclaw/openclaw.json` cookies
3. `JMECHAT_token` via tokenGrant
4. Local HiOffice legacy exchange (ports 8988-9006)

**If you get `NEED_ACCESS_RIGHT` or `PAGE NOT EXIST`:** The authenticated account doesn't have permission to that document. This is NOT a script error — the doc owner must grant access, or the user must log in with the correct account in HiOffice.

<!-- joyspace-kit start -->
# JoySpace Kit — Claude Code Skill

JoySpace document operations: read, search, list, upload images, and import markdown.

## Prerequisites

- Node.js >= 18
- HiOffice desktop client running (ports 8988-9006) for auth
- Network access to `apijoyspace.jd.com` (internal or VPN)

## Skill Scripts Location

All scripts are under the installed package root (`/Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit`). The `joyspace-kit init` command resolves the path automatically in the generated CLAUDE.md.

---

## 1. joyspace-read — Read a JoySpace Document

**When to use:** User pastes a `joyspace.jd.com/pages/xxx` URL and wants to read, summarize, quote, export, or back up the document. Or gives a pageId directly.

**Command:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyspace-read-doc/scripts/read_joyspace_doc.js --url "https://joyspace.jd.com/pages/<id>"
```

**Options:**

| Flag | Description | Default |
|------|-------------|---------|
| `--url` | Full JoySpace page URL | — |
| `--page-id` | Page id (alternative to --url) | — |
| `--save <path>` | Also write markdown to a local file | — |
| `--raw` | Include raw API responses (debugging) | false |

**Output:** JSON with `title`, `pageType`, `content` (reconstructed markdown), `link`, etc.

**Example — read and summarize:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyspace-read-doc/scripts/read_joyspace_doc.js --url "https://joyspace.jd.com/pages/avIBigHE3WXuUmspPCa6"
```

**Example — save to local file:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyspace-read-doc/scripts/read_joyspace_doc.js --url "https://joyspace.jd.com/pages/xxx" --save ./output.md
```

**Pitfalls:**
- `PAGE NOT EXIST` or `NEED_ACCESS_RIGHT` means the auth token's account lacks permission — not a script bug.
- Only `page_type=13` (markdown) is fully rendered. Sheets/mind maps/boards degrade to inline text; use `--raw` for those.
- First call via HiOffice legacy auth is slow (~1-3s port probe); subsequent calls reuse the token.

---

## 2. joyspace-search — Search JoySpace Documents

**When to use:** User wants to find a JoySpace page by keyword in title or content.

**Command:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyspace-search/scripts/search_joyspace.mjs --query "关键词" [--limit 20] [--json]
```

**Options:**

| Flag | Description | Default |
|------|-------------|---------|
| `--query` / `-q` | Search keyword (>= 3 chars) | required |
| `--limit` | Max results | 20 |
| `--start` | Offset for pagination | 0 |
| `--scope` | `global` or `received` | global |
| `--json` | Output raw JSON | false |

**Example:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyspace-search/scripts/search_joyspace.mjs --query "新品运营" --limit 10 --json
```

---

## 3. joyspace-list — List Recent Pages

**When to use:** User wants to see what they've been working on, find a doc by partial title, or browse recent JoySpace activity.

**Command:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyspace-list/scripts/list_joyspace.mjs [--limit 20] [--grep keyword] [--json]
```

**Options:**

| Flag | Description | Default |
|------|-------------|---------|
| `--limit` | Max results | 20 |
| `--grep` | Filter by title/preview substring | — |
| `--json` | Output raw JSON | false |

**Example:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyspace-list/scripts/list_joyspace.mjs --grep "PRD" --limit 5
```

---

## 4. joyspace-upload-image — Upload Image to JoySpace CDN

**When to use:** User wants to embed a local image into a JoySpace document. JoySpace only supports images by URL (no inline base64), and external image hosts often fail on the office network. This skill uploads to JoySpace's internal CDN.

**Command:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyspace-upload-image/scripts/upload_image.mjs --file /path/to/image.png --auto-scratch --delete-scratch
```

**Options:**

| Flag | Description | Default |
|------|-------------|---------|
| `--file` | Local image path (required) | — |
| `--page-id` | Upload to an existing page | — |
| `--auto-scratch` | Create temp scratch page, upload there | — |
| `--delete-scratch` | Delete scratch page after upload (CDN URL stays valid) | false |

**Output:** JSON with `imgUrl` (the stable CDN URL to use in markdown), `dimensions`.

**Example:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyspace-upload-image/scripts/upload_image.mjs --file ./screenshot.png --auto-scratch --delete-scratch
# Returns: { imgUrl: "https://apijoyspace.jd.com/v1/files/xxx/link", ... }
```

---

## 5. markdown-to-joyspace — Import Markdown as JoySpace Document

**When to use:** User wants to create a JoySpace doc from a `.md` file. Auto-uploads local image references to JoySpace's CDN.

**Command:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/markdown-to-joyspace/scripts/import_markdown_doc.js --file /path/to/doc.md
```

**Options:**

| Flag | Description | Default |
|------|-------------|---------|
| `--file` | Local markdown file (required) | — |
| `--title` | Document title | derived from filename or H1 |
| `--folder-id` | Target folder | personal space |
| `--team-id` | Target team | personal space |

**Example:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/markdown-to-joyspace/scripts/import_markdown_doc.js --file ./PRD.md --title "我的PRD"
```

---

## 6. hioffice-auth — Get Auth Token

**When to use:** Usually called automatically by other scripts. Run manually only for debugging.

**Command:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/hioffice-auth/scripts/hioffice-auth.mjs
```

**Output:** JSON `{ meToken: "ee.xxx...", authMode: "legacy" }`

---

## Common Workflows

### Read a JoySpace URL and summarize
```
User: "帮我看看这个文档 https://joyspace.jd.com/pages/xxx 在说什么"
→ Run joyspace-read with --url
→ Summarize the `content` field
```

### Search for a topic and read the top result
```
User: "搜一下全域通相关的文档"
→ Run joyspace-search --query "全域通"
→ From results, run joyspace-read --page-id <top pageId>
→ Summarize or quote
```

### Import a local markdown file to JoySpace
```
User: "把这个文件发布到JoySpace：./notes.md"
→ Run markdown-to-joyspace --file ./notes.md
→ Return the created doc URL
```

### Upload images and create a doc with screenshots
```
User: "把这个截图发到JoySpace"
→ Run joyspace-upload-image --file screenshot.png --auto-scratch --delete-scratch
→ Get imgUrl from result
→ Optionally create a doc with the image embedded
```

---

## Auth & Permissions

All scripts authenticate via the local HiOffice client. The auth chain:
1. `ME_TOKEN` / `SSO_TOKEN` env vars (if set)
2. `~/.joyclaw/openclaw.json` cookies
3. `JMECHAT_token` via tokenGrant
4. Local HiOffice legacy exchange (ports 8988-9006)

**If you get `NEED_ACCESS_RIGHT` or `PAGE NOT EXIST`:** The authenticated account doesn't have permission to that document. This is NOT a script error — the doc owner must grant access, or the user must log in with the correct account in HiOffice.

<!-- joyspace-kit end -->

<!-- joyme-tools start -->
# JoyMe Tools — Claude Code Skills

京Me 办公工具集：待办管理、消息发送、日历日程、慧记、会议室查询。

## Scripts Location

```
/Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/
├── joyme-joywork/scripts/joywork.mjs        # 待办管理
├── joyme-joychat/scripts/joychat.mjs        # 消息/聊天记录/摘要
├── joyme-joyday/scripts/joyday.mjs          # 日历日程
├── joyme-joyminutes/scripts/joyminutes.mjs  # 慧记
├── joyme-conference-room/scripts/conference-room.mjs  # 会议室
├── joyme-mail/scripts/mail.py               # 邮件 (Python)
├── joyme-joyfinance/scripts/joyfinance.py   # 小财神 (Python)
├── joyme-oa/scripts/                        # OA审批 (Python)
├── joyme-sheets/scripts/sheets.mjs          # JoySpace表格
├── joyme-tags/scripts/tags.mjs              # 文档标签
└── joyme-file-upload/scripts/file-upload.mjs # 文件上传
```

---

## 7. joyme-joywork — 待办/任务管理

**When to use:** 用户想创建、搜索、修改、催办待办任务。

**Command:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyme-joywork/scripts/joywork.mjs --action <action> [params]
```

**Actions:**

| Action | 功能 | 参数 |
|--------|------|------|
| `get_user_profile` | 获取当前用户信息 | 无 |
| `search_employee` | 搜索员工 | `--keyword` |
| `search_task` | 搜索待办 | `--title`, `--status` (1=进行中), `--endTime` (JSON范围) |
| `create_task` | 创建待办 | `--title`, `--endTime`, `--owners` (JSON数组), `--parentTaskId` |
| `update_task` | 修改待办 | `--taskId`, `--title`, `--remark`, `--endTime` |
| `update_task_status` | 完成/恢复 | `--taskId`, `--status` (1=完成, 0=恢复) |
| `share_task` | 分享待办 | `--taskId`, `--targets` (JSON) |
| `urge_task` | 催办 | `--taskId` |
| `update_task_progress` | 添加进展 | `--taskId`, `--content` |
| `summary_progress` | 获取进展列表 | `--taskId` |
| `get_todo_info` | 批量获取详情 | `--taskIds` (逗号分隔) |
| `add_executor` | 添加执行人 | `--taskId`, `--user` (JSON) |
| `del_executor` | 移除执行人 | `--taskId`, `--userId`, `--teamId` |

**Example:**
```bash
node .../joywork.mjs --action search_task --title "月卡"
node .../joywork.mjs --action create_task --title "完成PRD评审" --endTime "2026-06-15 18:00:00"
```

---

## 8. joyme-joychat — 消息发送

**When to use:** 用户想搜索同事、搜索群组、发送京Me消息。

**Command:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyme-joychat/scripts/joychat.mjs --action <action> [params]
```

**Actions:**

| Action | 功能 | 参数 |
|--------|------|------|
| `search_user` | 搜索用户 | `--keyword` |
| `search_group` | 搜索群组 | `--keyword` |
| `send_personal_message` | 发个人消息 | `--pin` (ERP), `--content`, `--app` (默认ee) |
| `send_group_message` | 发群消息 | `--groupId`, `--content` |
| `query_group_members` | 查群成员 | `--groupId` |
| `message_summary` | AI总结聊天消息 | `--pin`+`--app` 或 `--groupId`, `--startTime`, `--endTime`, `--unread` |

**Example:**
```bash
node .../joychat.mjs --action search_user --keyword "张三"
node .../joychat.mjs --action send_personal_message --pin "zhangsan" --content "你好"
node .../joychat.mjs --action message_summary --groupId "group_xxx" --startTime "2026-06-13 00:00:00"
```

**Note:** 消息发送使用AES加密，密钥自动获取，无需额外配置。

---

## 9. joyme-joyday — 日历/日程

**When to use:** 用户想查询忙闲、搜索/创建/修改日程会议。

**Command:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyme-joyday/scripts/joyday.mjs --action <action> [params]
```

**Actions:**

| Action | 功能 | 参数 |
|--------|------|------|
| `get_current_date` | 获取当前日期时间 | 无 |
| `get_user_profile` | 当前用户信息 | 无 |
| `search_user` | 搜索用户(获取userId) | `--keyword` |
| `query_free_busy` | 查忙闲 | `--users` (JSON), `--startDate`, `--endDate` (YYYY-MM-DD HH:mm:ss) |
| `search_appointment` | 搜索日程 | `--keyword`, `--startDate`, `--endDate` |
| `get_appointment_detail` | 日程详情 | `--scheduleId`, `--calendarId` |
| `get_appointment_attendees` | 参会人 | `--scheduleId` |
| `create_appointment` | 创建日程 | `--subject`, `--startDate`, `--endDate`, `--attendees` (JSON) |
| `modify_appointment` | 修改日程 | `--scheduleId`, `--subject`, `--startDate`, `--endDate` |
| `share_appointment` | 分享日程 | `--scheduleId`, `--targets` (JSON) |
| `summarize_appointments` | 日程汇总 | `--startDate`, `--endDate` |

**Example:**
```bash
node .../joyday.mjs --action query_free_busy --users '[{"userId":"xxx","teamId":"00046419"}]' --startDate "2026-06-14 09:00:00" --endDate "2026-06-14 18:00:00"
```

---

## 10. joyme-joyminutes — 慧记（会议纪要）

**When to use:** 用户想搜索、查看慧记内容。

**Command:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyme-joyminutes/scripts/joyminutes.mjs --action <action> [params]
```

**Actions:**

| Action | 功能 | 参数 |
|--------|------|------|
| `search_joyminutes` | 搜索慧记 | `--keyword`, `--timeFrom`, `--timeTo` (YYYY-MM-DD) |
| `get_detail` | 获取详情 | `--minutesId` |
| `get_asr` | 获取语音转写 | `--minutesId` |
| `share_joyminutes` | 分享 | `--minutesId`, `--targets` (JSON) |
| `create_joyminutes` | 创建 | `--mode` (record/translation) |

**Example:**
```bash
node .../joyminutes.mjs --action search_joyminutes --keyword "周会"
```

---

## 11. joyme-conference-room — 会议室查询

**When to use:** 用户想查询可用会议室。

**Command:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyme-conference-room/scripts/conference-room.mjs --action <action> [params]
```

**Actions:**

| Action | 功能 | 参数 |
|--------|------|------|
| `query_user_profile` | 获取用户默认城市 | 无 |
| `query_district_list` | 获取城市列表 | 无 |
| `query_workplace_list` | 获取职场列表 | `--districtCode` |
| `query_meeting_list` | 查询可用会议室 | `--date` (YYYY-MM-DD), `--startTime` (HHMM如900), `--endTime` (HHMM如1800), `--districtCode`, `--workplaceCode` |

**Workflow:**
```bash
# 1. 先查城市代码
node .../conference-room.mjs --action query_district_list
# 2. 查职场代码
node .../conference-room.mjs --action query_workplace_list --districtCode "13"
# 3. 查可用会议室
node .../conference-room.mjs --action query_meeting_list --date "2026-06-14" --startTime 1400 --endTime 1500 --districtCode "13" --workplaceCode "xxx"
```

---

## 12. joyme-mail — 邮件收发

**When to use:** 用户想搜索、阅读、回复、发送、转发邮件。

**Command:**
```bash
python3 /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyme-mail/scripts/mail.py <command> [options]
```

**Commands:**

| Command | 功能 | 关键参数 |
|---------|------|----------|
| `search` | 搜索邮件 | `--sender`, `--subject`, `--after`, `--before`, `--unread`, `--folder` |
| `detail` | 邮件详情 | `--item-id` |
| `send` | 发新邮件 | `--to`, `--subject`, `--body`, `--cc`, `--attachments` |
| `reply` | 回复邮件 | `--item-id`, `--body`, `--reply-all` |
| `forward` | 转发邮件 | `--item-id`, `--to`, `--body` |
| `folders` | 列出文件夹 | `--parent` |
| `daily-stats` | 按天统计 | `--after`, `--before`, `--days` |
| `pending-important` | 重要未回复 | `--days` |
| `person-unhandled` | 某人未处理 | `--sender`, `--days` |
| `lookup-recipient` | 查收件人 | `--condition`/`--name`/`--erp` |
| `batch-mark-read` | 批量已读 | `--item-ids` |
| `batch-delete` | 批量删除 | `--item-ids` |
| `flag` / `unflag` | 旗标 | `--item-id` |

**Example:**
```bash
python3 .../mail.py search --subject "周报" --after "2026-06-10" --format json
python3 .../mail.py send --to "zhangsan@jd.com" --subject "测试" --body "你好"
python3 .../mail.py detail --item-id "AAMkAGxx..." --format json
```

**Note:** 邮件使用SOAP/EWS协议，认证token自动通过HiOffice获取。加 `--format json` 输出JSON，默认输出markdown表格。

---

## 13. joyme-joyfinance — 小财神（经营数据查询）

**When to use:** 用户想查询GMV、营收、利润、同比环比等经营数据。

**Command:**
```bash
python3 /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyme-joyfinance/scripts/joyfinance.py --keyword "查询问题" [--json]
```

**Options:**

| Flag | Description | Default |
|------|-------------|---------|
| `--keyword` | 自然语言查询（如"25年5月GMV"） | required |
| `--json` | 输出JSON格式 | false |
| `--timeout` | 超时秒数 | 300 |

**Example:**
```bash
python3 .../joyfinance.py --keyword "25年5月GMV是多少" --json
python3 .../joyfinance.py --keyword "我负责品类的同比增速"
```

**Note:** 需要SSO_TOKEN环境变量。权限范围是用户名下SKU或所负责部门。灰度中，并非所有账号都开通。

---

## 14. joyme-oa — OA审批流程

**When to use:** 用户想查看审批流程进度、待审批列表、执行审批操作。

**Scripts:**

| Script | 功能 |
|--------|------|
| `query_process_progress.py` | 查询流程进度/我的申请 |
| `approve.py` | 审批通过/驳回 |
| `quick_approve.py` | 按分类快捷审批 |
| `approved_list.py` | 已审批列表 |

**Command:**
```bash
python3 /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyme-oa/scripts/query_process_progress.py [options]
```

**query_process_progress.py 选项:**
- `--my-applications` — 查询我发起的流程
- `--keyword KEYWORD` — 按标题/单号搜索流程
- `--process-instance-id ID` — 按实例ID查询
- `--query-type TYPE` — current_step/current_approver/history_approvers
- `--start-date`/`--end-date` — 日期范围

**approve.py 选项:**
- `--list-tasks` — 列出待审批任务
- `--approve --ids ID1,ID2,...` — 批量通过
- `--reject --id ID --comment COMMENT` — 驳回

**quick_approve.py 选项:**
- `--list-categories` — 列出流程分类
- `--approve --process-keys KEY1,KEY2,...` — 按分类快捷通过

**Example:**
```bash
python3 .../query_process_progress.py --my-applications --start-date 2026-06-01
python3 .../approve.py --list-tasks
python3 .../approve.py --approve --ids "task123,task456" --comment "同意"
```

**Note:** 需要SSO_TOKEN或ERP_COOKIE环境变量。

---

## 15. joyme-sheets — JoySpace电子表格

**When to use:** 用户想读取/写入JoySpace表格数据。

**Command:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyme-sheets/scripts/sheets.mjs --action <action> [params]
```

**Actions:**

| Action | 功能 | 参数 |
|--------|------|------|
| `get_worksheets` | 获取工作表列表 | `--pageId` |
| `create_worksheet` | 创建工作表 | `--pageId`, `--title` |
| `get_range_data` | 读取区域数据 | `--pageId`, `--sheetId`, `--range` (如A1:C10) |
| `update_range_data` | 写入区域数据 | `--pageId`, `--sheetId`, `--range`, `--values` (JSON二维数组) |
| `delete_range_data` | 清空区域 | `--pageId`, `--sheetId`, `--range` |
| `find_range_data` | 搜索内容 | `--pageId`, `--sheetId`, `--keyword` |
| `create_rows` | 插入行 | `--pageId`, `--sheetId`, `--startRow`, `--count` |

**Example:**
```bash
node .../sheets.mjs --action get_range_data --pageId "abc123" --sheetId "sheet1" --range "A1:D10"
node .../sheets.mjs --action update_range_data --pageId "abc123" --sheetId "sheet1" --range "A1:B2" --values '[["姓名","分数"],["张三","95"]]'
```

---

## 16. joyme-tags — 文档标签管理

**When to use:** 用户想给JoySpace文档打标签、管理标签体系。

**Command:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyme-tags/scripts/tags.mjs --action <action> [params]
```

**Actions:**

| Action | 功能 | 参数 |
|--------|------|------|
| `list_tags` | 列出所有标签 | 无 |
| `create_tag` | 创建标签 | `--name` |
| `delete_tag` | 删除标签 | `--tagId` |
| `rename_tag` | 重命名标签 | `--tagId`, `--name` |
| `get_page_tag_list` | 获取文档标签 | `--pageId` |
| `add_page_tag_ref` | 给文档打标签 | `--pageId`, `--tagId` |
| `remove_page_tag_ref` | 移除文档标签 | `--pageId`, `--tagId` |

**Example:**
```bash
node .../tags.mjs --action list_tags
node .../tags.mjs --action add_page_tag_ref --pageId "abc123" --tagId "5"
```

---

## 17. joyme-file-upload — 文件上传（京东云存储）

**When to use:** 用户想把本地文件上传到京东云获取URL，用于分享或嵌入文档。

**Command:**
```bash
node /Users/yangyilong.elon/.dongcc/npm-global/lib/node_modules/@jd/joyspace-kit/skills/joyme-file-upload/scripts/file-upload.mjs --action upload --filePath /path/to/file
```

**Output:** `{ "fileUrl": "https://..." }`

**Example:**
```bash
node .../file-upload.mjs --action upload --filePath ./report.pdf
node .../file-upload.mjs --action upload --filePath ./demo.mp4
```

**Note:** 支持最大500MB文件，自动分片上传。支持秒传（相同MD5的文件）。

---

## 18. joyme-joychat 扩展功能 — 消息摘要

joychat.mjs 的 message_summary action 可对聊天消息进行AI总结：

| Action | 功能 | 参数 |
|--------|------|------|
| `message_summary` | AI总结聊天消息 | `--pin`+`--app` 或 `--groupId`, `--startTime`, `--endTime`, `--unread` |

**Example:**
```bash
node .../joychat.mjs --action message_summary --groupId "group_xxx" --startTime "2026-06-13 00:00:00"
node .../joychat.mjs --action message_summary --pin "zhangsan" --app "ee" --unread true
```

<!-- joyme-tools end -->

---

## 19. JoyClaw 文档处理 Skills

以下 skills 来自 JoyClaw（`~/.joyclaw/`），可直接使用。**使用前先读取对应 SKILL.md 了解详细用法和注意事项。**

### docx — Word 文档创建/编辑/分析
- **When to use**: 用户要创建、编辑、分析 .docx 文件（tracked changes、comments、格式保留、文本提取）
- **详细文档**: 使用前必读 `~/.joyclaw/builtinSkills/docx/SKILL.md`
- **关键工具**:
  - 文本提取: `pandoc --track-changes=all file.docx -o output.md`
  - 解包XML: `python3 ~/.joyclaw/builtinSkills/docx/ooxml/scripts/unpack.py <file.docx> <output_dir>`
  - 创建新文档: 使用 docx-js (参见 SKILL.md 中的 docx-js.md)
- **依赖**: pandoc, Node.js

### pptx — PPT 演示文稿创建/编辑
- **When to use**: 用户要创建、编辑 .pptx 文件
- **详细文档**: 使用前必读 `~/.joyclaw/builtinSkills/pptx/SKILL.md`
- **关键工具**:
  - 文本提取: `python3 -m markitdown file.pptx`
  - 解包XML: `python3 ~/.joyclaw/builtinSkills/pptx/ooxml/scripts/unpack.py <file.pptx> <output_dir>`
  - 创建新PPT: 使用 html2pptx 工作流 (参见 SKILL.md)
- **依赖**: pandoc, Node.js

### xlsx — Excel 表格处理
- **When to use**: 分析/处理 .xlsx/.csv 文件、公式重算
- **详细文档**: 使用前必读 `~/.joyclaw/builtinSkills/xlsx/SKILL.md`
- **关键工具**:
  - 公式重算: `python3 ~/.joyclaw/builtinSkills/xlsx/recalc.py`
  - 数据处理: 使用 Python openpyxl
- **依赖**: Python (openpyxl)

### pdf — PDF 处理
- **When to use**: PDF 文本提取、合并、拆分、表格提取、表单填写
- **详细文档**: 使用前必读 `~/.joyclaw/builtinSkills/pdf/SKILL.md`
- **依赖**: Python (pypdf, pdfplumber)
- **快速用法**:
```python
from pypdf import PdfReader
reader = PdfReader("doc.pdf")
text = "".join(p.extract_text() for p in reader.pages)
```

### markdown-to-docx — Markdown 转 Word
- **When to use**: 将 .md 文件转为 .docx 格式
- **详细文档**: `~/.joyclaw/remoteSkills/SKILL_LOCAL_YAmrKX77c/SKILL.md`
- **命令**:
```bash
python3 ~/.joyclaw/remoteSkills/SKILL_LOCAL_YAmrKX77c/scripts/convert.py
```
- **依赖**: pandoc

---

## 20. JoyClaw 智能体调用 — JoyAgent

- **When to use**: 用户要调用 JoyAgent 平台智能体、执行工作流、查询智能体信息
- **详细文档**: `~/.joyclaw/remoteSkills/joyagent-caller/SKILL.md`
- **命令**:
```bash
# 查询智能体信息
python3 ~/.joyclaw/remoteSkills/joyagent-caller/scripts/joyagent_invoke.py --agent-id "<id>" --token "<token>" --erp "<erp>" get-agent-info

# 智能体问答
python3 ~/.joyclaw/remoteSkills/joyagent-caller/scripts/joyagent_invoke.py --agent-id "<id>" --token "<token>" --erp "<erp>" ask --keyword "问题"

# 执行工作流
python3 ~/.joyclaw/remoteSkills/joyagent-caller/scripts/joyagent_invoke.py --agent-id "<id>" --token "<token>" --erp "<erp>" run-workflow --keyword "输入"
```
- **环境**: 线上 `--env prod`(默认), 预发 `--env pre`
- **依赖**: Python (requests)

---

## 21. JoyClaw 日报周报生成

- **When to use**: 用户说"写日报""生成周报""生成日报""整理周报"
- **详细文档**: `~/.joyclaw/remoteSkills/SKILL_LOCAL_B4DjWcZ1J/SKILL.md`
- **一键生成+发送**:
```bash
node ~/.joyclaw/remoteSkills/SKILL_LOCAL_B4DjWcZ1J/scripts/send_report_live.mjs
# --dry-run 只生成不发送
```
- **手动生成报告**:
```bash
python3 ~/.joyclaw/remoteSkills/SKILL_LOCAL_B4DjWcZ1J/scripts/unified_report_real.py <daily|weekly> <days_ago>
```
- **需要环境变量**: `REAL_MESSAGES_JSON`, `REAL_DOCS_JSON`, `REAL_TODOS_JSON`（由脚本自动通过工具收集）

---

## 22. JoyClaw 方法论 Skills

使用时**先读取对应 SKILL.md**，按其中定义的流程和模板执行。

| 场景 | 触发词 | SKILL.md 路径 |
|------|--------|--------------|
| PRD 评审/拷打 | "评审PRD""挑PRD问题""grill需求" | `~/.joyclaw/remoteSkills/SKILL_LOCAL_eJvNF0hMe/SKILL.md` |
| PRD 构建 | "写PRD""构建PRD" | `~/.joyclaw/remoteSkills/SKILL_LOCAL_nWFmD2d4b/SKILL.md` |
| 汇报材料框架 | "写汇报""汇报材料""三段论""战略屋" | `~/.joyclaw/remoteSkills/SKILL_LOCAL_EhC2575yM/SKILL.md` |
| 工作周报(个人模板) | "写周报""生成周报""帮我写本周汇报" | `~/.joyclaw/remoteSkills/SKILL_LOCAL_LHALn1DE6/SKILL.md` |
| 内容研究写作 | 写文章、博客、技术文档(带引用) | `~/.joyclaw/workspace-me-claw-f027wfjv9x/skills/CLAW_SKILL_100015721/SKILL.md` |
| 总结方法论 | "总结""TLDR""摘要" | `~/.joyclaw/workspace-me-claw-f027wfjv9x/skills/CLAW_SKILL_100015847/SKILL.md` |
| OA流程(扩展) | "OA审批效率""流程催办""加签" | `~/.joyclaw/workspace-me-claw-f027wfjv9x/skills/CLAW_SKILL_100017521/SKILL.md` |
