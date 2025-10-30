# Laravel 9 REST APIs

A production-ready Laravel 9 REST API boilerplate with multi-tenancy, services & repository patterns, resources, events/jobs, model states, and AWS S3 integration.

This repository provides a modular foundation for building multi-tenant REST APIs using Laravel. It includes examples for Offers, Products, Deductibles, Domains, Tenancy bootstrappers, background jobs, observers, factories and seeders.

---

## Table of contents

- Overview
- Key features & architecture highlights
- Getting started (local development)
- Environment variables (important entries)
- Database, factories, seeders & testing data
- Notable files & where to look
- Example API endpoints & sample requests
- Running queues, jobs & storage notes
- What’s good / design decisions
- Suggested improvements and missing items
- Contributing
- License
- Maintainer

---

## Overview

This project is a Laravel 9 REST API project structured to support multi-tenant applications (using stancl/tenancy patterns). It separates business logic into Services and Repositories, uses API Resources for consistent JSON responses, and employs events/jobs for side-effects like image uploads.

The codebase contains:
- Controllers for API endpoints (app/Http/Controllers/api/v1)
- Service layer (app/CoreLogic/Services)
- Repositories (app/CoreLogic/Repositories)
- Models, Observers and Model States (app/Models, Observers/, App\Models\States)
- Tenancy bootstrappers and features for tenant-specific S3 buckets (TenancyBootstrappers/, TenancyFeatures/)
- Database factories and seeders for quick test data generation (database/...)
- Routes in routes/api.php

---

## Key features & architecture highlights

- Laravel 9 based REST API
- Multi-tenancy support (stancl/tenancy)
- Service + Repository pattern to isolate business logic and data access
- API Resources (consistent JSON formatting)
- Events and Jobs (async tasks like UploadProductImagesJob)
- Observers for model lifecycle logic (e.g., UserObserver)
- Model state management (Spatie/Model-States style pattern present)
- AWS S3 integration with tenant-aware bucket management
- Database factories and seeders to quickly spin-up sample data
- Uuids on models (HasUuids) and SoftDeletes where appropriate
- Enums used for typed values (DeductibleCategoryEnum, OfferTypeEnum, etc.)

---

## Getting started (local development)

Prerequisites
- PHP 8.1+ (match the composer.json requirement)
- Composer
- MySQL / PostgreSQL or other supported DB
- Node (if you use any frontend tooling)
- Redis (optional, for queues)
- AWS account or local S3 emulator (like localstack) if testing S3
- Optional: Postman or HTTP client for API testing

Basic setup
1. Clone the repository:
   git clone https://github.com/codewithnagesh/Laravel-9-REST-APIs.git
2. Install PHP dependencies:
   composer install
3. Copy env file and generate app key:
   cp .env.example .env
   php artisan key:generate
4. Configure .env (database, queue, S3 and tenancy entries) — see next section for important env entries.
5. Run migrations and seeders:
   php artisan migrate
   php artisan db:seed
   Or, to seed tenant-specific data: (follow project-specific seeder instructions if available)
6. Start the dev server:
   php artisan serve

If your app uses queues for image uploads:
- Start a queue worker:
  php artisan queue:work --tries=3

---

## Important environment variables (examples)

Make sure these are present and configured in .env:

- APP_NAME, APP_ENV, APP_KEY, APP_URL
- DB_CONNECTION, DB_HOST, DB_PORT, DB_DATABASE, DB_USERNAME, DB_PASSWORD
- SANCTUM state (if using token auth)
- QUEUE_CONNECTION (database/redis)
- S3 configuration for AWS:
  - AWS_ACCESS_KEY_ID
  - AWS_SECRET_ACCESS_KEY
  - AWS_DEFAULT_REGION
  - AWS_BUCKET
  - AWS_ENDPOINT (for localstack/testing)
- TENANCY related keys (depend on how stancl/tenancy is configured):
  - TENANCY central domain, tenant model setup, etc.

Note: This repo contains bootstrappers and features that change the active S3 bucket per tenant, so proper S3 credentials and bucket naming strategy are important.

---

## Database, factories & seeding

- Factories exist for Offer, Customer, Deductible, Product and other models (database/factories).
- Seeders such as UnitTypeSeeder and TenantDatabaseSeeder create baseline data.
- Use factories for testing and local development:
  - php artisan db:seed --class=UnitTypeSeeder
  - php artisan db:seed --class=TenantDatabaseSeeder

---

## Notable files & where to look

- routes/api.php — main API routes and middleware (auth:sanctum, tenant initialization)
- app/Http/Controllers/api/v1 — Controllers for resources (Domain, Offer, Product, Deductible, etc.)
- app/CoreLogic/Services — Service layer (OfferService, ProductService, DeductibleService)
- app/CoreLogic/Repositories — Data access layer
- app/Resources — API Resources (OfferResource, ProductResource)
- TenancyBootstrappers/ & TenancyFeatures — tenant-specific S3 bucket & bootstrapping logic
- database/factories & database/seeders — test data generation
- Observers/UserObserver.php — user hashing and tenant-owner sync

---

## Example API endpoints & sample requests

(Adjust base URL and tokens to your environment)

- Register (example controller present: RegisterController)
  POST /api/v1/register
  Body: { "name": "Alice", "email": "a@a.com", "password": "secret" }

- Offers
  GET /api/v1/offers
  POST /api/v1/offers
  GET /api/v1/offers/{offer}
  PUT /api/v1/offers/{offer}
  DELETE /api/v1/offers/{offer}

- Products
  GET /api/v1/products
  POST /api/v1/products
  GET /api/v1/products/{product}

- Deductibles
  GET /api/v1/deductibles
  POST /api/v1/deductibles

Sample curl (login/register/token flows depend on implementation, here is a simple GET example):
curl -H "Accept: application/json" -H "Authorization: Bearer <TOKEN>" "http://localhost:8000/api/v1/offers"

For detailed route listing:
- Inspect routes/api.php and run:
  php artisan route:list --path=api

---

## Running queues & S3 / tenant bucket notes

- The repo uses jobs like UploadProductImagesJob which expect queued processing (start queue worker to process).
- Tenancy features set S3 bucket per tenant (see TenancyBootstrappers/TenantBucketBootstrapper and TenancyFeatures/AWSS3BucketFeature). Test S3 integration with a real S3 account or a local S3 emulator (localstack).
- When running tenant bootstrappers, config values are swapped during tenant lifecycle — be mindful when debugging file storage.

---

## What’s good / design decisions observed

- Clear separation of concerns: Services handle business logic and Repositories handle persistence.
- Use of API Resources ensures consistent responses across endpoints.
- Multi-tenant design with bootstrappers and per-tenant S3 demonstrates production readiness for SaaS scenarios.
- Use of factories, seeders and observers aids testing and predictable model behavior.
- Event-driven side-effects (events + jobs) for decoupling long-running tasks.
- Use of enums and model states increases correctness and readability.

---

## Suggested improvements / missing items

To make the project even more usable and friendlier to contributors and users:
- Add a top-level README (this file) with clear setup steps and required env variables (this file aims to be that).
- Provide a Postman collection or OpenAPI/Swagger spec for the API endpoints.
- Add example .env.example with required AWS/TENANCY variables documented.
- Provide sample tenant creation & testing guide (how to create tenants, how central vs tenant domains work).
- Add tests (feature + unit) and a short CI pipeline example (GitHub Actions) to run tests and linting.
- Add code style/linter configs and a pre-commit hook (e.g., PHP-CS-Fixer, composer scripts).
- Document authentication flows (how to get tokens, refresh, sanctum usage).
- Provide a CONTRIBUTING.md and ISSUE/PR templates to streamline contributions.
- Add logging/monitoring suggestions and recommended cache/queue redis config for production.
- Optionally include a small demo script or Makefile with common commands (serve, migrate, seed, queue).

---

## Contributing

- Please open issues or PRs for bugs, features or docs.
- Follow repository coding standards; include tests for new features where possible.
- If you'd like, open a PR with the README changes and I can help refine it further and add a Postman collection or API docs.

---

## License

Specify your license here (e.g., MIT). If not set, consider adding a LICENSE file.

---

## Maintainer / Contact

codewithnagesh (GitHub) — use issues or PRs on the repository for discussion.

---

Thank you — I reviewed the main controllers, services, tenancy bootstrappers, factories and seeders when drafting this README. If you want, I can now commit this README.md to a new branch and open a PR, or expand the README with a Postman collection and example environment file.  
