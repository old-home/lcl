# Project Development Guidelines

## Build/Configuration Instructions

### Frontend (React with React Router)

1. **Installation**:
   ```bash
   cd frontend
   pnpm install
   ```

2. **Development**:
   ```bash
   cd frontend
   pnpm dev
   ```
   This starts the development server with Hot Module Replacement (HMR) at `http://localhost:5173`.

3. **Production Build**:
   ```bash
   cd frontend
   pnpm build
   ```
   The build output will be in the `build` directory.

### Backend (Laravel)

1. **Installation**:
   ```bash
   cd backend
   composer install
   cp .env.example .env
   php artisan key:generate
   ```

2. **Database Setup**:
   ```bash
   php artisan migrate
   php artisan db:seed
   ```

   **Note**: Always use Artisan commands to perform database migrations. Direct SQL execution for migrations is not allowed.

### Docker Setup

The project includes Docker configuration for development and deployment:

1. **Start Docker Environment**:
   ```bash
   docker-compose up -d
   ```

2. **Environment Variables**:
   - `WEB_LISTEN_IP`: IP to bind the web server (default: 0.0.0.0)
   - `WEB_PORT`: External port for web server (default: 80)
   - `WEB_INTERNAL_PORT`: Internal port for web server (default: 80)
   - `UID` and `GID`: User and group IDs for the app container

## Testing Information

### Frontend Testing

The frontend uses Vitest with React Testing Library for testing.

1. **Running Tests**:
   ```bash
   cd frontend
   pnpm run test           # Run all tests
   pnpm run test:coverage  # Run tests with coverage report
   ```

2. **Running Specific Tests**:
   ```bash
   cd frontend
   pnpm run test -- path/to/test/file.test.tsx
   ```

3. **Test Structure**:
   - Tests are located next to the files they test with a `.test.tsx` extension
   - Tests use Vitest's `describe`, `it`, and `expect` functions
   - React Testing Library is used for rendering components and interacting with them

4. **Example Test**:
   ```typescript
   import { describe, it, expect, vi } from 'vitest'
   import { render, screen, fireEvent } from '@testing-library/react'
   import { YourComponent } from './YourComponent'

   describe('YourComponent', () => {
     it('renders correctly', () => {
       // Render the component
       const { getByText } = render(YourComponent())

       // Assert that the expected text is in the document
       expect(getByText('Expected Text')).toBeInTheDocument()
     })
   })
   ```

5. **Mocking**:
   - The `vitest.setup.ts` file contains global mocks for i18n functionality
   - Use `vi.mock()` to mock dependencies in individual test files

### Backend Testing

The backend uses PHPUnit for testing, which is standard for Laravel applications.

1. **Running Tests**:
   ```bash
   cd backend
   php artisan test
   ```

2. **Running Specific Tests**:
   ```bash
   cd backend
   php artisan test tests/Feature/YourTest.php
   ```

3. **Test Structure**:
   - Feature tests are in `tests/Feature/`
   - Unit tests are in `tests/Unit/`
   - Tests extend the `Tests\TestCase` class
   - Test methods are prefixed with `test_`

4. **Example Test**:
   ```php
   <?php

   namespace Tests\Feature;

   use Tests\TestCase;

   class YourTest extends TestCase
   {
       public function test_your_feature(): void
       {
           $response = $this->get('/your-route');
           $response->assertStatus(200);
       }
   }
   ```

5. **Database Testing**:
   - Use the `RefreshDatabase` trait to reset the database between tests
   - Use factories to create test data

## Additional Development Information

### Code Style

1. **EditorConfig**:
   - The project includes an `.editorconfig` file that defines basic code style rules
   - 2-space indentation for most files
   - 4-space indentation for PHP files and docker-compose.yml
   - UTF-8 encoding and LF line endings

2. **Frontend Formatting**:
   - Prettier is used for code formatting
   - Run `pnpm run format` to format all files
   - Run `pnpm run format:check` to check formatting without changing files

3. **Backend Formatting**:
   - Laravel Pint is used for code formatting
   - Run `composer pint` to format PHP files

### Internationalization

The frontend uses i18next for internationalization:
- Language switching is implemented in the `LanguageSwitcher` component
- Translations are accessed using the `useTranslation` hook
- Currently supports English (en) and Japanese (ja)

### Known Issues

1. **Backend Tests**:
   - The default Feature test fails because the root route (`/`) returns a 404 status code
   - Make sure to update tests to match your actual routes or implement the expected routes

### Project Structure

1. **Frontend**:
   - React application with React Router
   - TypeScript for type safety
   - Tailwind CSS for styling
   - Vite for building and development

2. **Backend**:
   - Laravel 12 application
   - PostgreSQL database
   - Redis for caching

3. **Docker**:
   - Nginx for web server
   - PHP-FPM for application server
   - PostgreSQL for database
   - Redis for cache

### Backend Architecture

1. **Routing and Actions**:
   - Controllers should not be used
   - Use single-action classes named with the suffix `Action` (e.g., `CreateUserAction`)
   - All Action classes must implement the `__invoke` magic method
   - Each Action class should handle only one specific task or endpoint

2. **Example Action**:
   ```php
   <?php

   namespace App\Actions;

   use Illuminate\Http\Request;
   use Illuminate\Http\JsonResponse;

   class GetUserAction
   {
       public function __invoke(Request $request, int $userId): JsonResponse
       {
           // Action logic here
           return response()->json(['user' => $user]);
       }
   }
   ```

3. **Route Definition**:
   ```php
   // routes/api.php
   use App\Actions\GetUserAction;

   Route::get('/users/{userId}', GetUserAction::class);
   ```

4. **Domain-Driven Design**:
   - Do not use Eloquent models directly in the application code
   - Use Entity and Value Object patterns for domain objects
   - All data access should be done through repositories, not directly through models
   - Create an interface for each repository in the `App\Repositories\Interfaces` namespace
   - Implement each repository interface in the `App\Repositories` namespace
   - Register repository bindings in a service provider
   - Inject repositories into actions via constructor dependency injection

5. **Example Repository Interface**:
   ```php
   <?php

   namespace App\Repositories\Interfaces;

   interface AccountRepositoryInterface
   {
       public function create(array $data);
       public function findById(int $id);
       public function update(int $id, array $data);
       public function delete(int $id);
   }
   ```

6. **Example Entity Class**:
   ```php
   <?php

   namespace App\Domain\Entities;

   class Account
   {
       private int $id;
       private string $email;
       private string $password;
       private \DateTime $createdAt;
       private \DateTime $updatedAt;

       public function __construct(
           string $email,
           string $password,
           ?int $id = null,
           ?\DateTime $createdAt = null,
           ?\DateTime $updatedAt = null
       ) {
           $this->email = $email;
           $this->password = $password;
           $this->id = $id ?? 0;
           $this->createdAt = $createdAt ?? new \DateTime();
           $this->updatedAt = $updatedAt ?? new \DateTime();
       }

       public function getId(): int
       {
           return $this->id;
       }

       public function getEmail(): string
       {
           return $this->email;
       }

       public function getPassword(): string
       {
           return $this->password;
       }

       public function getCreatedAt(): \DateTime
       {
           return $this->createdAt;
       }

       public function getUpdatedAt(): \DateTime
       {
           return $this->updatedAt;
       }

       public function toArray(): array
       {
           return [
               'id' => $this->id,
               'email' => $this->email,
               'password' => $this->password,
               'created_at' => $this->createdAt->format('Y-m-d H:i:s'),
               'updated_at' => $this->updatedAt->format('Y-m-d H:i:s'),
           ];
       }

       public static function fromArray(array $data): self
       {
           return new self(
               $data['email'],
               $data['password'],
               $data['id'] ?? null,
               isset($data['created_at']) ? new \DateTime($data['created_at']) : null,
               isset($data['updated_at']) ? new \DateTime($data['updated_at']) : null
           );
       }
   }
   ```

7. **Example Value Object Class**:
   ```php
   <?php

   namespace App\Domain\ValueObjects;

   class Email
   {
       private string $value;

       public function __construct(string $email)
       {
           if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
               throw new \InvalidArgumentException('Invalid email address');
           }
           $this->value = $email;
       }

       public function getValue(): string
       {
           return $this->value;
       }

       public function __toString(): string
       {
           return $this->value;
       }

       public function equals(Email $other): bool
       {
           return $this->value === $other->value;
       }
   }
   ```

8. **Example Repository Implementation**:
   ```php
   <?php

   namespace App\Repositories;

   use App\Domain\Entities\Account;
   use App\Models\Account as AccountModel;
   use App\Repositories\Interfaces\AccountRepositoryInterface;

   class AccountRepository implements AccountRepositoryInterface
   {
       public function create(array $data)
       {
           $model = AccountModel::create($data);
           return Account::fromArray($model->toArray());
       }

       public function findById(int $id)
       {
           $model = AccountModel::findOrFail($id);
           return Account::fromArray($model->toArray());
       }

       public function update(int $id, array $data)
       {
           $model = AccountModel::findOrFail($id);
           $model->update($data);
           return Account::fromArray($model->toArray());
       }

       public function delete(int $id)
       {
           return AccountModel::destroy($id) > 0;
       }
   }
   ```

9. **Example Action with Repository**:
   ```php
   <?php

   namespace App\Actions;

   use App\Repositories\Interfaces\AccountRepositoryInterface;
   use Illuminate\Http\Request;
   use Illuminate\Http\JsonResponse;
   use Illuminate\Support\Facades\Hash;

   class CreateAccountAction
   {
       private $accountRepository;

       public function __construct(AccountRepositoryInterface $accountRepository)
       {
           $this->accountRepository = $accountRepository;
       }

       public function __invoke(Request $request): JsonResponse
       {
           // Validation logic here...

           $account = $this->accountRepository->create([
               'email' => $request->email,
               'password' => Hash::make($request->password),
           ]);

           return response()->json([
               'message' => 'Account created successfully',
               'account' => $account,
           ], 201);
       }
   }
   ```
