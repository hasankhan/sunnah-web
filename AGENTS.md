# Copilot Instructions for sunnah-web

## Fork

This is a fork. Upstream: https://github.com/sunnah-com/website.git

## Project Overview

This is the frontend for [sunnah.com](https://sunnah.com), a hadith collection website built on the **Yii 2** PHP MVC framework. It serves multilingual hadith content (Arabic, English, Urdu, Bangla, Bosnian, Indonesian) organized by collections, books, and chapters.

## Architecture

### Request Flow

1. Web server points to `public/` → `public/index.php` bootstraps Yii 2
2. URL routing is defined in `application/config/main.php` (extensive pattern-based rules)
3. All frontend logic lives in a single Yii module: `application/modules/front/`
4. The app uses two databases: main hadith DB (`db`) and a search DB (`searchdb`), plus Elasticsearch

### Key Directories

- `application/config/main.php` — Central config: routes, DB connections, caching, modules
- `application/modules/front/controllers/` — 4 controllers: Collection, Index, Narrator, Search
- `application/modules/front/models/` — Domain models (Hadith, Book, Collection, Chapter, Narrator) with language-specific hadith models (EnglishHadith, ArabicHadith, etc.)
- `application/modules/front/views/` — View templates organized by controller
- `application/components/` — CDN caching headers and search engine abstraction
- `public/` — Web root with static assets (CSS, JS, images)

### Configuration

Environment config is in `.env.local` (copy from `.env.local.sample`). This INI file is parsed in `application/config/main.php` and provides DB credentials, cache settings, API keys, and the Yii framework path.

## Development Setup

### Docker (recommended)

```bash
cp .env.local.sample .env.local
docker-compose up --build
```

Access at `http://localhost`.

### Windows (native)

Requires PHP 8.5, IIS, MySQL, and Composer. Run `composer install` in the project root.

## Code Formatting

Use [PHP CS Fixer](https://github.com/PHP-CS-Fixer/PHP-CS-Fixer) for code formatting.

## Conventions

- All user-facing routes are defined as URL rules in `application/config/main.php` — add new pages there
- Hadith display uses a colon-separated URL pattern: `collectionName:hadithNumber` (e.g., `bukhari:1`)
- Language-specific hadith models extend a base pattern — each language has its own model class and likely its own DB table
- The `application/controllers/SController.php` is a shared base controller
- CDN cache behavior is managed via components (`CdnHeaders`, `CdnEdgeCache`, `CdnOriginAndEdgeCache`)
- All content must be Islamically appropriate — no haram references in any user-facing text

## Search

Search uses an abstraction layer (`application/components/search/`) that supports Elasticsearch via `ElasticConnection`. The search controller handles both legacy URL patterns and current search.
