# Amazon S3 Tables Hands-on Lab with AWS Analytics Services

Bilingual (English / Vietnamese) Hugo documentation site for the AWS Workshop covering Amazon S3 Tables, Apache Iceberg, and AWS Analytics services.

## Workshop Overview

This workshop guides participants through building a data lake analytics environment using Amazon S3 Tables with Apache Iceberg. Participants learn to:

- Create and manage S3 Table Buckets and Namespaces
- Load CSV and VCF data into Iceberg tables via EMR Serverless
- Perform CRUD operations (Merge, Update, Delete) on Iceberg tables
- Use Time Travel queries (snapshots, rollback)
- Query tables with Amazon Athena and EMR Serverless
- Visualize data using Amazon QuickSight
- Manage S3 Tables maintenance jobs (compaction, snapshot management)
- Leverage Agentic AI with Amazon Q CLI and MCP Server

## Content Structure

```
content/
  _index.md / _index.vi.md          # Home page
  0-introduction/                    # Apache Iceberg & S3 Tables concepts
  1-getting-started/                 # Account setup (instructor-led / self-paced)
  2-prerequisites/                   # Table Bucket, Namespace, EMR Workspace
  4-lab-2/                           # [Lab-1] Queries Using Clinical Data
  4-lab-1/                           # [Lab-2] Queries using Public Data
  4-lab-3/                           # [Lab-3] Queries using VCF Data
  5-management-task/                 # S3 Tables Maintenance Jobs
  6-agentic-ai/                      # Amazon Q CLI + MCP Server
  7-conclusion/                      # Next steps
```

## Tech Stack

| Component | Version / Details |
|-----------|-------------------|
| Hugo | 0.134.3+ extended |
| Theme | hugo-theme-learn (Git submodule) |
| Languages | English (primary), Vietnamese |
| Deployment | GitHub Pages via GitHub Actions |
| Source | [AWS Workshop Studio](https://catalog.us-east-1.prod.workshops.aws/workshops/61e4a7a9-5776-46d2-a671-497df9b22a13/en-US) |

## Quick Start

```bash
# Clone with submodules
git clone --recurse-submodules <repo-url>
cd S3TableAnalytics_Workshop

# Run locally (with drafts)
hugo server -D

# Production build
hugo --gc --minify
```

## Development

- English pages: `content/**/_index.md`
- Vietnamese pages: `content/**/_index.vi.md`
- Images: `static/images/workshop/`
- Config: `config/_default/hugo.toml`

See [AGENTS.md](AGENTS.md) for detailed authoring rules and conventions.

## License

Workshop content sourced from AWS Workshop Studio. All rights reserved by Amazon Web Services, Inc.
