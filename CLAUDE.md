# Symfony Professional Starter - Claude Code Guidelines

This document defines the coding standards, architectural principles, and review criteria for this Symfony project. When reviewing code or implementing features, follow these guidelines to ensure consistency and quality.

## Project Overview

This is a **Symfony Professional Starter** template that provides best practices for building enterprise-grade Symfony applications with modern development workflows.

### Technology Stack

- **Language**: PHP 8.4 (strict types, readonly properties)
- **Framework**: Symfony 8
- **Database**: MySQL 8.0 / PostgreSQL
- **CI/CD**: GitHub Actions
- **Code Quality**: PHP CS Fixer, PHPStan, PHPUnit
- **Development Tools**: Docker, Composer, Husky

## Code Quality Standards

### PHP Coding Standards

All code must pass **PHP CS Fixer** with `@Symfony` and `@Symfony:risky` rules:

```bash
composer check-style  # Must pass
composer format       # Auto-fix issues
```

**Key Requirements**:
- `declare(strict_types=1)` at the top of every PHP file
- Use `readonly` properties for immutable value objects
- Follow PSR-12 coding standard
- Use short array syntax `[]`
- No trailing whitespace
- Proper line breaks and indentation

### Static Analysis

All code must pass **PHPStan** at Level 5:

```bash
composer analyze  # Must pass with 0 errors
```

Focus on:
- Type safety for all parameters and return types
- No undefined variables or methods
- Proper null safety handling
- Generic types for collections

## Architectural Principles

### Domain-Driven Design (DDD)

When applicable, organize the codebase into **bounded contexts** based on business domains. Each context should have:

- **Domain Layer**: Entities, value objects, aggregates, domain services
- **Application Layer**: Command handlers, query handlers, use cases
- **Infrastructure Layer**: Repository implementations, external service integrations

**Example DDD Structure**:
```php
namespace App\User\Domain\Entity;

use App\User\Domain\ValueObject\UserId;
use App\User\Domain\ValueObject\Email;

final class User
{
    private readonly UserId $id;
    private readonly Email $email;
    private bool $isActive;

    public function __construct(UserId $id, Email $email)
    {
        $this->id = $id;
        $this->email = $email;
        $this->isActive = true;
    }

    public function deactivate(): void
    {
        if (!$this->isActive) {
            throw new \DomainException('User is already inactive');
        }
        $this->isActive = false;
    }
}
```

### Key Architectural Patterns

#### 1. Service Layer Pattern
Separate business logic from controllers:

```php
// Service class
class UserService
{
    public function __construct(
        private UserRepository $repository,
        private EventDispatcherInterface $dispatcher
    ) {}

    public function createUser(CreateUserCommand $command): User
    {
        $user = new User(
            UserId::generate(),
            new Email($command->email)
        );

        $this->repository->save($user);
        $this->dispatcher->dispatch(new UserCreatedEvent($user->getId()));

        return $user;
    }
}
```

#### 2. Repository Pattern
Abstract data access logic:

```php
interface UserRepositoryInterface
{
    public function save(User $user): void;
    public function findById(UserId $id): ?User;
    public function findByEmail(Email $email): ?User;
}
```

#### 3. Event-Driven Architecture
Use Symfony Messenger for async processing:

```php
// Message handler
class SendWelcomeEmailHandler implements MessageHandlerInterface
{
    public function __invoke(SendWelcomeEmail $message): void
    {
        $user = $this->userRepository->find($message->getUserId());
        $this->mailer->sendWelcomeEmail($user);
    }
}
```

### Performance Optimization

1. **Avoid N+1 Queries**: Use Doctrine batching or DQL
2. **Use Covering Indexes**: Index all columns needed for queries
3. **Cache Aggressively**: Use Symfony Cache for read-heavy operations
4. **Bulk Operations**: Batch inserts/updates when possible
5. **Lazy Loading**: Configure Doctrine lazy associations appropriately

## Security Best Practices

### Critical Security Rules

1. **Never commit secrets** - Use `.env.local` for local overrides
2. **Strong APP_SECRET** - Generate with `openssl rand -hex 32`
3. **Input validation** - Always validate user input with Symfony Validator
4. **Output encoding** - Use Twig auto-escaping for XSS prevention
5. **SQL injection** - Always use parameterized queries (Doctrine)
6. **CSRF protection** - Enable Symfony CSRF tokens on forms
7. **Authentication/Authorization** - Use Symfony Security component

### Security Review Checklist

- [ ] No hardcoded credentials or API keys
- [ ] All user input is validated
- [ ] Database queries use parameterized statements
- [ ] Sensitive data is encrypted at rest
- [ ] Proper error handling (no stack traces to users)
- [ ] Rate limiting on public endpoints
- [ ] CORS configured properly
- [ ] Dependencies are up-to-date

## Performance Standards

### Performance Targets

Set appropriate targets based on your application requirements:
- **API Response**: < 200ms p50, < 1s p95 (typical web apps)
- **Database Queries**: < 100ms for single queries
- **Background Jobs**: Process within SLA requirements

### Optimization Guidelines

1. **Avoid N+1 Queries**: Use Doctrine batching or DQL
2. **Use Covering Indexes**: Index all columns needed for queries
3. **Cache Aggressively**: Use Symfony Cache for read-heavy operations
4. **Bulk Operations**: Batch inserts/updates when appropriate
5. **Lazy Loading**: Configure Doctrine lazy associations appropriately

### Performance Review Checklist

- [ ] No N+1 query problems
- [ ] Appropriate indexes exist
- [ ] Caching strategy defined where needed
- [ ] Database queries analyzed and optimized
- [ ] Query execution time analyzed
- [ ] Memory usage profiled

## Testing Standards

### Test Coverage Requirements

- **Unit Tests**: 80%+ coverage for domain logic
- **Integration Tests**: All repositories and message handlers
- **End-to-End Tests**: Critical workflows (schedule → check → ingest → query)

### Testing Best Practices

```php
// Example: Unit test with Zenstruck Foundry
namespace App\Tests\Unit\Monitor\Domain\Entity;

use App\Monitor\Domain\Entity\Monitor;
use App\Monitor\Domain\ValueObject\MonitorId;
use App\Monitor\Domain\ValueObject\Url;
use PHPUnit\Framework\TestCase;

class MonitorTest extends TestCase
{
    public function test_can_be_disabled(): void
    {
        $monitor = new Monitor(MonitorId::generate(), new Url('https://example.com'));

        $monitor->disable();

        $this->assertFalse($monitor->isActive());
    }

    public function test_cannot_be_disabled_twice(): void
    {
        $monitor = new Monitor(MonitorId::generate(), new Url('https://example.com'));
        $monitor->disable();

        $this->expectException(\DomainException::class);
        $monitor->disable();
    }
}
```

### Test Review Checklist

- [ ] Tests cover happy path
- [ ] Tests cover error cases
- [ ] Tests cover edge cases
- [ ] No test logic duplication (use data providers)
- [ ] Tests are independent (no shared state)
- [ ] Test names are descriptive

## Documentation Standards

### Required Documentation

1. **Inline Comments**: Only for "why", not "what"
2. **PHPDoc**: Required for all public methods
3. **Architecture Decision Records (ADRs)**: For major decisions
4. **README.md**: Update for feature changes
5. **PROJECT_BLUEPRINT.md**: Update for architecture changes

### PHPDoc Example

```php
/**
 * Dispatches monitors that are due for checking.
 *
 * This method finds all monitors whose next_check_at timestamp
 * is in the past and dispatches them to RabbitMQ for processing
 * by worker instances.
 *
 * @return int Number of monitors dispatched
 */
public function dispatchDueMonitors(): int
{
    // Implementation...
}
```

## Code Review Criteria

### Critical Issues (Must Fix)

- Security vulnerabilities (exposed secrets, injection attacks)
- Database queries without pagination (potential OOM)
- Missing validation on user input
- Hardcoded configuration values
- Blocking operations in async handlers
- Missing error handling for external services

### Major Improvements (Should Fix)

- Violation of DDD principles (domain logic in wrong layer)
- N+1 query problems
- Missing test coverage for critical paths
- Unclear naming or code organization
- Missing or incomplete PHPDoc
- Performance bottlenecks

### Minor Suggestions (Nice to Have)

- Code style consistency
- Extract complex logic into well-named methods
- Add more integration tests
- Improve error messages
- Add logging for debugging
- Optimization opportunities

## Review Output Structure

When providing code review feedback, structure it as:

1. **Summary**: 2-3 sentence overall assessment
2. **Strengths**: What's working well (specific examples)
3. **Critical Issues**: Problems that must be addressed
4. **Major Improvements**: Significant enhancements
5. **Minor Suggestions**: Nice-to-have improvements
6. **Learning Resources**: Relevant documentation or patterns
7. **Action Plan**: Prioritized changes with effort estimates

## Example Review Responses

### Positive Review Example

"Excellent implementation of the Monitor entity. The DDD patterns are well-applied with clear separation between domain and application layers. The value objects (MonitorId, Url) properly encapsulate domain concepts. The `disable()` method correctly guards against invalid state transitions with a domain exception. **No changes needed.**"

### Constructive Review Example

"**Summary**: The telemetry ingestion service shows good understanding of performance requirements, but has security and architecture concerns.

**Critical Issues**:
1. **SQL Injection Risk**: The bulk insert concatenates values instead of using parameters. Use parameterized statements.
2. **Missing Error Handling**: No try-catch around Redis operations. Connection failures will crash the worker.

**Major Improvements**:
1. **DDD Violation**: Raw SQL in Telemetry context is correct, but consider extracting to a dedicated `TelemetryRepository` class.
2. **No Batch Size Validation**: Hardcoded 1000 items could exceed MySQL `max_allowed_packet`. Make it configurable.

**Minor Suggestions**:
- Add metrics logging for batch size and processing time
- Consider using a transaction for atomicity

**Action Plan**:
1. Fix SQL injection (5 min)
2. Add error handling (10 min)
3. Extract to repository class (20 min)
4. Add batch size validation (10 min)
Total: ~45 minutes

Overall, good performance-focused approach. Address the critical security issue before merging."

## Resources

- **PROJECT_BLUEPRINT.md**: Full architectural documentation
- **Symfony Best Practices**: https://symfony.com/doc/current/best_practices.html
- **DDD by Example**: https://github.com/CleanArchitecture/DDD-by-examples
- **MySQL Partitioning**: https://dev.mysql.com/doc/refman/8.0/en/partitioning.html
- **Redis Patterns**: https://redis.io/docs/manual/patterns/

---

**Remember**: This is a "System Design Gym" project. The goal is to learn advanced scalability patterns. When reviewing code, balance strict standards with learning opportunities. Explain the "why" behind suggestions to foster understanding.
