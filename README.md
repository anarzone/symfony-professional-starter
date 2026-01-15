# Symfony Professional Starter

A professional Symfony 8 template with best practices for building enterprise-grade applications. Features modern development workflows, automated code quality checks, and interactive code review.

## ✨ Features

- **Symfony 8** with PHP 8.4
- **Strict Type Safety** - `declare(strict_types=1)` enforced everywhere
- **Automated Quality Checks**:
  - PHP CS Fixer with `@Symfony` and `@Symfony:risky` rules
  - PHPStan at Level 5
  - PHPUnit for testing
- **Git Hooks** (via Husky):
  - Pre-commit: Automated style and static analysis checks
  - Pre-push: Interactive code review with Claude CLI
- **Docker Support** - Containerized infrastructure services
- **Semantic Release** - Automated versioning and changelog generation

## 🚀 Quick Start

**Prerequisites:**
- PHP 8.4+
- Composer
- Docker (optional, for infrastructure)
- Claude CLI (for code reviews)

```bash
# 1. Clone the repository
git clone https://github.com/anarzone/symfony-professional-starter.git
cd symfony-professional-starter

# 2. Install dependencies
composer install

# 3. Configure environment
cp .env .env.local
# Edit .env.local with your settings

# 4. Run database migrations
php bin/console doctrine:migrations:migrate

# 5. Start the development server
symfony server:start
```

## 📋 Available Commands

### Quality Checks

```bash
# Run all quality checks
composer check-all

# Run individual checks
composer check-style      # PHP CS Fixer
composer analyze         # PHPStan Level 5
composer test            # PHPUnit

# Auto-fix code style
composer format
```

### Development

```bash
# Database migrations
php bin/console doctrine:migrations:migrate

# Create a new migration
php bin/console doctrine:migrations:diff

# Cache management
php bin/console cache:clear
```

## 🔍 Code Review System

This project includes an **interactive pre-push code review** system that runs automatically before every push.

### How It Works

1. Make your changes and commit them
2. Run `git push`
3. The pre-push hook automatically reviews your changes
4. If critical issues are found, you'll be prompted:
   ```
   Continue with push anyway? (y/N):
   ```
5. Choose to fix issues or proceed with push

### Configuration

Edit `.husky/pre-push-config.yml`:

```yaml
# Enable/disable automatic code review
auto_review: true

# Interactive mode: ask before proceeding with issues
interactive: true

# Maximum files to review per push
max_files: 20
```

### Manual Review

For deeper analysis, use the manual review script:

```bash
./scripts/review-pr.sh <PR_NUMBER>
```

See [CODE_REVIEW_WORKFLOW.md](CODE_REVIEW_WORKFLOW.md) for complete documentation.

## 🏗 Project Structure

```
.
├── bin/                    # Console scripts
├── config/                 # Symfony configuration
├── migrations/             # Database migrations
├── public/                 # Public entry point
├── src/                    # Application source code
│   ├── Controller/         # HTTP controllers
│   ├── Entity/             # Doctrine entities
│   ├── Repository/         # Data access layer
│   ├── Service/            # Business logic
│   └── ...                 # Other domain code
├── templates/              # Twig templates
├── tests/                  # PHPUnit tests
├── .husky/                 # Git hooks
│   ├── pre-commit          # Runs before commit
│   ├── pre-push            # Runs code review before push
│   └── pre-push-config.yml # Code review configuration
├── CLAUDE.md               # Coding standards & review criteria
└── CODE_REVIEW_WORKFLOW.md # Code review documentation
```

## 📚 Coding Standards

This project follows strict coding standards defined in [CLAUDE.md](CLAUDE.md):

- **PHP 8.4** with strict types enabled
- **PSR-12** coding standard (via PHP CS Fixer)
- **Type safety** via PHPStan Level 5
- **SOLID principles** and clean architecture patterns
- **Comprehensive testing** with PHPUnit

## 🐳 Docker Support

Docker is configured for infrastructure services:

```bash
# Start infrastructure services
docker compose up -d

# View logs
docker compose logs -f

# Stop services
docker compose down
```

## 🔄 Git Workflow

### Branch Strategy

- `main` - Production branch
- `feat/*` - Feature branches
- `fix/*` - Bugfix branches
- `docs/*` - Documentation updates

### Commit Convention

This project uses [Conventional Commits](https://www.conventionalcommits.org/):

```bash
feat: add user registration
fix: resolve login issue
docs: update README
refactor: simplify user service
test: add unit tests for user entity
```

### Pre-Commit Hook

Runs automatically before each commit:
- ✅ PHP CS Fixer (code style)
- ✅ PHPStan (static analysis)
- ✅ PHPUnit (tests)

Bypass if needed:
```bash
git commit --no-verify
```

### Pre-Push Hook (Code Review)

Runs automatically before each push:
- 🤖 Claude CLI code review
- 🤔 Interactive prompt for critical issues
- ⚡ Fast local feedback

Bypass if needed:
```bash
git push --no-verify
```

## 📖 Documentation

- **[CLAUDE.md](CLAUDE.md)** - Coding standards and architectural guidelines
- **[CODE_REVIEW_WORKFLOW.md](CODE_REVIEW_WORKFLOW.md)** - Code review system documentation
- **[README.md](README.md)** - This file

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feat/amazing-feature`)
3. Make your changes
4. Commit with conventional commit message
5. Push and create a Pull Request

The pre-push code review will automatically check your changes!

## 📄 License

This project is open-sourced software licensed under the [MIT license](LICENSE).

## 🔗 Resources

- [Symfony Documentation](https://symfony.com/doc)
- [PHP CS Fixer](https://cs.symfony.com)
- [PHPStan](https://phpstan.org)
- [PHPUnit](https://phpunit.de)
- [Docker](https://www.docker.com)
- [Claude Code CLI](https://code.claude.com)
