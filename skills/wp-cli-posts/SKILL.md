---
name: wp-cli-posts
description: >
  Use this skill to create, update, or manage WordPress posts, pages, and custom post types
  via WP-CLI over SSH. Triggers whenever the user wants to publish, edit, update, or manage
  content on a WordPress site — including standard posts, pages, and custom post types (CPTs).
  Also covers media uploads and multisite (subfolder or subdomain) scenarios. Always use this
  skill when the user mentions WP CLI, WordPress content management, updating a WordPress page
  or post, uploading images to WordPress, or managing WordPress from the command line, even if
  they don't explicitly say "WP CLI" or "skill".
---

# WP CLI Posts & Pages Skill

Manage WordPress content (posts, pages, custom post types) using WP-CLI over SSH.

---

## Step 0: Verify SSH Connectivity

Before running any WP-CLI command, confirm SSH access is configured and working.

Run a connectivity test:

```bash
wp @<alias> cli version
```

If this fails, guide the user to set up `~/.wp-cli/config.yml`:

```yaml
@production:
  ssh: user@your-server.com
  path: /var/www/html
```

For AWS Elastic Beanstalk or key-based auth, use:

```yaml
@production:
  ssh: ec2-user@your-eb-hostname.amazonaws.com --ssh-keys=/path/to/key.pem
  path: /var/www/html
```

Or via `~/.ssh/config` (cleaner):

```text
Host wp-production
  HostName your-server.com
  User ec2-user
  IdentityFile ~/.ssh/your-key.pem
```

Then reference in `config.yml`:

```yaml
@production:
  ssh: wp-production
  path: /var/www/html
```

**Do not proceed until SSH connectivity is confirmed.**

---

## Step 1: Gather Context

Before doing anything, collect the necessary context based on the task:

### For UPDATE tasks

- Ask the user for the **page/post URL(s)** to be updated — always, without exception.
- Derive the `--url` flag (for multisite) from the provided URL. See [Multisite Handling](#multisite-handling) below.
- If the post type **cannot be clearly inferred from the URL**, ask explicitly — do NOT guess.

### For CREATE tasks

- Ask what **post type** this should be (post, page, or a custom post type name) if not clearly stated — do NOT assume.
- If this is a **multisite** setup, ask which sub-site the content should be created on if not stated.

---

## Step 2: Fetch the Post ID

### By URL

```bash
wp @<alias> post url-to-id <url> [--url=<subsite-url>]
```

Or by listing posts filtered by slug:

```bash
wp @<alias> post list --post_type=<type> --name=<slug> --fields=ID,post_title,post_status --format=json [--url=<subsite-url>]
```

If you get no results with `post`, try `page` or ask the user for the post type.

---

## Step 3: Fetch Existing Content (Update Tasks Only)

Always retrieve the existing content **before** making any changes:

```bash
wp @<alias> post get <ID> --fields=post_title,post_content,post_excerpt,post_status --format=json [--url=<subsite-url>]
```

For meta/custom fields (e.g. ACF):

```bash
wp @<alias> post meta get <ID> <meta_key> --format=json [--url=<subsite-url>]
```

Apply the requested changes **locally** first (in your working context), and make only the **minimum changes requested** — no reformatting, no unsolicited additions, no structural changes.

---

## Step 4: Handle Media Uploads (If Needed)

If the update involves new images or files, upload media **before** patching the post.

**Upload from local path:**

```bash
wp @<alias> media import /path/to/file.jpg --porcelain [--url=<subsite-url>]
```

**Upload from a URL:**

```bash
wp @<alias> media import https://example.com/image.jpg --porcelain [--url=<subsite-url>]
```

**Attach to a post and set as featured image:**

```bash
wp @<alias> media import image.jpg --post_id=<ID> --featured_image --porcelain [--url=<subsite-url>]
```

After import with `--porcelain`, WP-CLI outputs the plain **attachment ID**. To get the attachment URL (to use in post content), run:

```bash
wp @<alias> post get <ATTACHMENT_ID> --field=guid [--url=<subsite-url>]
```

---

## Step 5: Apply the Update or Create the Post

### Update an existing post

```bash
wp @<alias> post update <ID> \
  --post_title="Updated Title" \
  --post_content="Updated content here" \
  [--url=<subsite-url>]
```

### Update a meta/custom field

```bash
wp @<alias> post meta update <ID> <meta_key> "<new_value>" [--url=<subsite-url>]
```

### Create a new post/page/CPT

```bash
wp @<alias> post create \
  --post_type=<type> \
  --post_title="Title" \
  --post_content="Content" \
  --post_status=publish \
  --porcelain \
  [--url=<subsite-url>]
```

> ⚠️ Make the **smallest requested change only**. Do not alter formatting, structure, or any content that wasn't explicitly requested.

---

## Step 6: Verify the Published Output

After updating or creating, fetch the live URL using `web_fetch` (or take a screenshot if available) to visually verify the changes:

- Confirm the update is reflected correctly.
- Check for layout issues, broken HTML, or missing media.
- If discrepancies are found, report them clearly and ask the user how to proceed before making further changes.

---

## Multisite Handling

Always check the provided URL to determine if this is a multisite setup.

| Setup type          | Example URL                      | `--url` flag                     |
| ------------------- | -------------------------------- | -------------------------------- |
| Single site         | `https://example.com/about`      | Not needed                       |
| Subfolder multisite | `https://example.com/shop/about` | `--url=https://example.com/shop` |
| Subdomain multisite | `https://shop.example.com/about` | `--url=https://shop.example.com` |

For **create tasks**, if the site is a multisite and the target sub-site isn't mentioned, ask explicitly before proceeding.

---

## Custom Post Types (CPTs)

WP-CLI treats CPTs identically to posts/pages — just pass `--post_type=<cpt_slug>`.

**For updates:** If the post type cannot be inferred from the URL (most slugs don't expose this), ask the user directly:

> "What post type is this? (e.g., `post`, `page`, `product`, `event`, etc.)"

**For creates:** If the user hasn't specified a post type, ask:

> "What post type should this be created as?"

Do not assume `post` or `page` as a fallback for CPTs.

**List all registered CPTs on the site** (useful for reference):

```bash
wp @<alias> post-type list --fields=name,label --format=json [--url=<subsite-url>]
```

---

## Quick Reference: Useful Commands

| Task               | Command                                                                                  |
| ------------------ | ---------------------------------------------------------------------------------------- |
| Get post by URL    | `wp post url-to-id <url>`                                                                |
| Get post content   | `wp post get <ID> --fields=post_content --format=json`                                   |
| List posts by type | `wp post list --post_type=<type> --fields=ID,post_title --format=json`                   |
| Update post        | `wp post update <ID> --post_title="..." --post_content="..."`                            |
| Create post        | `wp post create --post_type=<type> --post_title="..." --post_status=publish --porcelain` |
| Import media       | `wp media import <file-or-url> --porcelain`                                              |
| Set featured image | `wp media import <file> --post_id=<ID> --featured_image --porcelain`                     |
| Get meta field     | `wp post meta get <ID> <key> --format=json`                                              |
| Update meta field  | `wp post meta update <ID> <key> "<value>"`                                               |
| List sub-sites     | `wp site list --fields=blog_id,url --format=json`                                        |
| List CPTs          | `wp post-type list --fields=name,label --format=json`                                    |
