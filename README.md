<!-- post\README.md -->

# Nihil Post Database Service

<!-- ![Nihil Logo](link-to-logo.png) -->

The **Post Database Service** is the persistence layer for the [Nihil platform](https://github.com/Ange230700/nihil_db_post).  
It provides schema definitions, seeding logic, test utilities, and a generated Prisma client for managing **posts** and their relationships (authors and shares).

This package (`nihildbpost`) is designed to be reusable and published to npm for consumption by other Nihil services.

---

## Table of Contents
- [Demo](#demo)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Running the Project](#running-the-project)
- [Project Structure](#project-structure)
- [Entities & ERD](#entities--erd)
- [Testing](#testing)
- [Deployment](#deployment)
- [Environment Variables](#environment-variables)
- [Contributing](#contributing)
- [License](#license)
- [Contact](#contact)

---

## Demo

ER Diagram of the Post Service schema:

```mermaid
erDiagram
    post {
        UUID id PK
        UUID user_id FK
        TEXT content
        STRING media_url
        DATETIME created_at
        DATETIME updated_at
        BOOLEAN is_deleted
        UUID original_post_id FK "for shares/retweets"
    }
    post }|..|| user : "posted by"
    post ||--|{ post : "shares"
````

---

## Tech Stack

* **Language**: TypeScript (ESM, strict mode)
* **ORM**: [Prisma](https://www.prisma.io/)
* **Database**: MySQL
* **Security**: [argon2](https://www.npmjs.com/package/argon2) (for user-password parity with User Service)
* **Testing**: Jest + ts-jest
* **Tooling**: Nx, Docker, Husky, Commitlint, Lint-staged, tsx

---

## Getting Started

### Prerequisites

* Node.js (>= 20.x)
* MySQL database instance
* Docker (optional, for containerized builds)

### Installation

Clone the repository and install dependencies:

```bash
git clone https://github.com/Ange230700/nihil_db_post
cd nihil/post
npm install
```

---

## Running the Project

### Build

```bash
npm run build
```

### Generate Prisma Client

```bash
npm run prisma:generate
```

### Apply Schema to Database

```bash
npm run prisma:db:push
```

### Seed Database

```bash
npm run prisma:db:seed
```

---

## Project Structure

```
post/
├── prisma/
│   ├── schema.prisma       # Prisma schema
│   ├── seed.ts             # Seeder logic
│   ├── main.ts             # Seeding entrypoint
│   ├── lib/client.ts       # Prisma client wrapper
│   ├── helpers/            # DB cleanup scripts
│   └── generated/          # Prisma client output
├── docs/
│   ├── erd.mmd             # ER diagram (Mermaid)
│   └── erd.md              # Schema documentation
├── tsconfig.json
├── project.json
├── package.json
└── Dockerfile
```

---

## Entities & ERD

* **Post**

  * `id`: UUID (PK)
  * `userId`: author (FK → User service)
  * `content`: text body of the post
  * `mediaUrl`: optional link to media
  * `isDeleted`: soft-delete flag
  * `originalPostId`: self-reference for shares/retweets
  * `createdAt`, `updatedAt`

**Relationships:**

* Each post belongs to a **user** (foreign service).
* A post can reference one original post.
* A post can have many shares.

---

## Testing

Tests are powered by Jest and Prisma.
The schema is reset before tests, ensuring consistent runs.

```bash
npm run test
```

Example (`prisma/seed.spec.ts`):

* Seeds posts
* Asserts seeded count matches `NUM_POSTS`
* Cleans up after

---

## Deployment

A multi-stage Dockerfile is included:

```dockerfile
# build stage
FROM node:24-alpine AS build
WORKDIR /app
COPY package*.json nx.json ./
COPY shared ./shared
COPY post ./post
RUN npm ci
RUN npm run prisma:generate --workspace=post && npm run build:all

# run stage
FROM node:24-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY --from=build /app ./
CMD ["sh", "-lc", "npm run prisma:db:push --workspace=post && npm run prisma:db:seed --workspace=post && tail -f /dev/null"]
```

Build and run:

```bash
docker build -t nihil-post-db .
docker run --env-file .env nihil-post-db
```

---

## Environment Variables

Create a `.env` or use `.env.sample`:

```env
POST_DATABASE_URL="mysql://user:password@localhost:3306/post_db"
SKIP_CLEANUP=<true|false>
NODE_ENV=<dev|test|prod>
```

Optional:

* `NUM_POSTS`: Number of posts to seed (default: 20)
* `USER_IDS`: Comma-separated list of user IDs to assign as authors (falls back to synthetic IDs)

---

## Contributing

Contributions are welcome!

1. Fork the repository
2. Create a branch (`git checkout -b feat/my-feature`)
3. Commit (`git commit -m "✨ add my feature"`)
4. Push (`git push origin feat/my-feature`)
5. Create a Pull Request

Git hooks enforce linting and test passes before commits/pushes.

---

## License

MIT License

---

## Contact

**Ange KOUAKOU**
[GitHub](https://github.com/your-username) • [Email](mailto:your.email@example.com)
