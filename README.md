# WordPress Skills

A collection of Claude AI skills for managing WordPress sites via WP-CLI over SSH.

These skills are designed to be installed in [Claude.ai](https://claude.ai) and give Claude structured, opinionated workflows for common WordPress content management tasks — without needing a plugin, REST API integration, or admin panel access.

---

## Skills

### [`skills/wp-cli-posts`](./skills/wp-cli-posts/)

Create, update, and manage WordPress posts, pages, and custom post types using WP-CLI over SSH.

**What it does:**
- Creates and updates posts, pages, and custom post types (CPTs)
- Uploads media via WP-CLI and injects attachment URLs into post content
- Fetches existing content before applying changes (read → patch → verify)
- Verifies the published URL after every update for discrepancies
- Supports WP Multisite — both subfolder and subdomain setups
- Guides SSH/WP-CLI configuration if not already set up

**Guardrails built in:**
- Always asks for the page URL before any update
- Never guesses the post type — asks explicitly if it can't be inferred
- Makes the minimum requested change only — no unsolicited edits
- Asks which sub-site to target for multisite create tasks

---

## Installation

1. Download the `.skill` file for the skill you want from the [Releases](../../releases) page (or directly from the `skills/` directory).
2. In Claude.ai, go to **Settings → Skills**.
3. Upload the `.skill` file.
4. Claude will now automatically use the skill when relevant.

---

## Prerequisites

Each skill assumes the following are in place on your local machine:

- **WP-CLI** installed locally (or accessible on the remote server)
- **SSH access** to the remote WordPress host
- A `~/.wp-cli/config.yml` with a named alias for the remote server

Minimal `config.yml` example:
```yaml
@production:
  ssh: user@your-server.com
  path: /var/www/html
```

For AWS Elastic Beanstalk with a key pair:
```yaml
@production:
  ssh: ec2-user@your-eb-hostname.amazonaws.com --ssh-keys=~/.ssh/your-key.pem
  path: /var/www/html
```

If SSH isn't configured, the skill will detect this and walk you through the setup before proceeding.

---

## Contributing

Skills follow a standard structure:

```
skills/
└── skill-name/
    ├── SKILL.md          # Required — skill instructions and metadata
    └── references/       # Optional — additional reference docs
```

To add a new skill, open a PR with your `SKILL.md` under an appropriate subdirectory in `skills/`. Please include a description of what the skill does and when it should trigger in the YAML frontmatter.

---

## Maintained by

[UEM Group Websites](https://github.com/UEM-Group-Websites)
