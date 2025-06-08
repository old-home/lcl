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

Now writing...
