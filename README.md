# NVD Mirror

[中文文档](README_zh.md)

A GitHub Actions-powered tool that periodically downloads and mirrors the [NVD (National Vulnerability Database)](https://nvd.nist.gov/) used by [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/). The database is packaged as a compressed archive and published as a GitHub Release, providing a ready-to-use mirror for anyone who wants to skip the slow NVD download process.

## Why?

OWASP Dependency-Check needs to download the full NVD database before scanning dependencies. This process can be slow and is rate-limited by the NVD API. This project solves the problem by providing a pre-built, regularly updated database that you can download and use directly.

## How It Works

1. A GitHub Actions workflow runs **daily at 00:00 UTC** (also supports manual trigger).
2. It uses the `dependency-check-maven` plugin (`update-only` goal) to download/update the full NVD database.
3. The database is cached across runs for incremental updates.
4. The result is compressed into `nvd-database.tar.gz` (plus a `.sha256` checksum) and uploaded in place to the GitHub Release `nvd-data-latest`, so the download URL is always valid.

## Download

Download the latest database archive from the [Releases page](https://github.com/diktamen/nvd-mirror/releases/tag/nvd-data-latest).

## Usage

1. Download and extract the archive:

   ```bash
   wget https://github.com/diktamen/nvd-mirror/releases/download/nvd-data-latest/nvd-database.tar.gz
   tar -xzf nvd-database.tar.gz
   ```

2. Point your `dependency-check-maven` configuration to the extracted data directory:

   **Maven CLI:**
   ```bash
   mvn org.owasp:dependency-check-maven:check \
     -DdataDirectory=./dc-data
   ```

   **dependency-check CLI** (the CLI distribution keeps its database in `lib/data/11.0`):
   ```bash
   mkdir -p dependency-check/lib/data/11.0
   curl -fsSL https://github.com/diktamen/nvd-mirror/releases/download/nvd-data-latest/nvd-database.tar.gz \
     | tar -xz --strip-components=1 -C dependency-check/lib/data/11.0
   dependency-check/bin/dependency-check.sh --updateonly --nvdApiKey "$NVD_API_KEY"   # incremental update only
   ```

   **pom.xml:**
   ```xml
   <plugin>
     <groupId>org.owasp</groupId>
     <artifactId>dependency-check-maven</artifactId>
     <version>13.0.0</version>
     <configuration>
       <dataDirectory>/path/to/dc-data</dataDirectory>
     </configuration>
   </plugin>
   ```

## Tech Stack

- **Java 17** (Temurin)
- **Maven** + OWASP Dependency-Check Maven Plugin 13.0.0
- **GitHub Actions** (scheduled CI pipeline)
- **GitHub Releases** (artifact distribution)

## Configuration

| Setting | Value | Description |
|---------|-------|-------------|
| Schedule | `0 0 * * *` (daily 00:00 UTC) | Daily incremental update; the workflow re-enables itself so the 60-day inactivity rule never switches it off |
| NVD API Delay | 500ms | Rate limit compliance |
| Data Directory | `dc-data/` | NVD database storage location |
| Release Tag | `nvd-data-latest` | Always points to the latest build |
| Timeout | 720 minutes | Maximum workflow duration |

### Required Secrets

- `NVD_API_KEY` — Your [NVD API key](https://nvd.nist.gov/developers/request-an-api-key) for authenticated access with higher rate limits.
- `GITHUB_TOKEN` — Automatically provided by GitHub Actions.

## Forking

To set up your own mirror:

1. Fork this repository.
2. Add your `NVD_API_KEY` to the repository secrets (**Settings > Secrets and variables > Actions**).
3. Enable GitHub Actions for the forked repository.
4. The workflow will run on schedule, or you can trigger it manually from the Actions tab.

## License

This project is provided as-is for convenience. The NVD data is provided by NIST and subject to the [NVD terms of use](https://nvd.nist.gov/faq).
