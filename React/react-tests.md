# Comprehensive React Testing Guide

## Key Principles

1. **Mock External Dependencies Completely**
   - Mock all external dependencies, including services, contexts, utilities, browser APIs (`localStorage`, `fetch`), and any module imported by the unit under test
   - Ensure mocks precisely replicate the original module's behavior:
     - Match constructor arguments and method signatures
     - Return values must match expected types (use `mockResolvedValue(undefined)` for `Promise<void>`)
     - Mimic critical internal logic where necessary (e.g., if a service stringifies data before saving)
   - Pay special attention to file system operations, network calls, and browser APIs
   - When mocking singleton services, ensure the mock follows the same pattern as the original

2. **Handle Asynchronous Operations**
   - Use `waitFor` for async operations and state updates after API calls
   - Prefer `findByRole` over `getByRole` for elements that might not be immediately available
   - Always wait for loading states to complete before making assertions
   - Be cautious with progress callbacks and event handlers
   - Be aware that state updates in tests can trigger `useEffect` hooks, potentially causing additional function calls
   - Ensure all promises are handled (awaited or explicitly ignored if appropriate)

3. **Context and Provider Setup**
   - Use `MemoryRouter` instead of `BrowserRouter` for testing router-dependent components
   - Create a reusable `AllTheProviders` wrapper for consistent context setup
   - Ensure all required context providers are present in the correct order
   - Mock context values appropriately and completely

## Real-World Troubleshooting & Lessons Learned

### 1. Deeply Align Mock Context and State
- Ensure your mock context and project data actually trigger the UI states under test, not just superficially matching types or values.
- If a component conditionally renders based on computed or derived state, verify your mock triggers those conditions.

### 2. Debug Output Limitations
- In large or complex DOMs, debug output (e.g. `screen.debug()`, `console.log(document.body.textContent)`) can be truncated or lost.
- Prefer targeted debug prints (e.g. a specific menu or region) and use tools like `logTestingPlaygroundURL()` for visual DOM inspection.

### 3. Async and Portal Rendering Awareness
- Components may render asynchronously or into portals, so standard queries (like `getByText`) may fail if the element is not yet in the DOM or is outside the main tree.
- Use `waitFor` or `findBy*` queries, and consider querying the entire document or portal root if needed.

### 4. Troubleshooting: When Expected Elements Aren't Rendered
- If an expected element is missing, check:
  - The mock data and context shape
  - State updates and async effects
  - Whether the component uses portals or delayed rendering
- Print the entire DOM (`document.body.innerHTML`) after the user action to see what is actually rendered.
- Review the component logic to ensure your test setup matches the real-world scenario.

## Mocking Best Practices

### Service Mocking

```typescript
// Define mock instance outside
const mockServiceInstance = {
  method: vi.fn(),
  anotherMethod: vi.fn()
};

// Mock the service factory/singleton pattern
vi.mock('@/services/example', () => {
  return {
    ExampleService: {
      getInstance: vi.fn(() => mockServiceInstance)
    }
  };
});

// Clear mocks before each test
beforeEach(() => {
  vi.clearAllMocks();
  mockServiceInstance.method.mockImplementation(async (args, onProgress) => {
    if (onProgress) {
      onProgress({ status: 'preparing' });
      onProgress({ status: 'completed' });
    }
    return { success: true };
  });
});
```

### Context Mocking

```typescript
import { useProject } from '@/context/ProjectContext';
import { ProjectContextType, Project } from '@/types';

vi.mock('@/context/ProjectContext');

// In beforeEach:
const mockProject: Project = { /* complete mock project */ };

vi.mocked(useProject).mockReturnValue({
  project: mockProject,
  updateTechnicalPlan: vi.fn(),
  // Mock ALL other properties from ProjectContextType
  setProject: vi.fn(),
  updateStage: vi.fn(),
  // ... all other properties
  isLoading: false,
  allProjects: [mockProject],
} as ProjectContextType);
```

### Important Notes on `vi.mock`

- Define mock instances *outside* the `vi.mock` factory function scope if they need to be accessed in tests
- Use `vi.mocked()` to wrap imported functions/objects for type-safe access to mock properties
- Avoid using `require()` within tests to access mocked modules with path aliases (use direct imports instead)

## Testing Components

### Component Testing Structure

```typescript
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { describe, it, expect, beforeEach, vi } from 'vitest';
import YourComponent from './YourComponent';

// Mock dependencies
vi.mock('@/services/example', () => ({
  // Mock implementation
}));

// Create test wrapper if needed
const AllTheProviders = ({ children }) => (
  <ContextProvider>
    {children}
  </ContextProvider>
);

// Custom render function
const customRender = (ui: React.ReactElement) => {
  return render(ui, { wrapper: AllTheProviders });
};

describe('YourComponent', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('should render successfully', async () => {
    customRender(<YourComponent />);
    
    await waitFor(() => {
      expect(screen.queryByText(/loading/i)).not.toBeInTheDocument();
    });
    
    // Add assertions
  });
});
```

## Testing Router-Dependent Components/Hooks

```typescript
import { renderHook, act } from '@testing-library/react';
import { MemoryRouter, Routes, Route } from 'react-router-dom';
import { useAppNavigation } from '../useAppNavigation';
import { ROUTES } from '@/routes/config';

// Wrapper providing router context
const wrapper = ({ children }) => (
  <MemoryRouter initialEntries={[ROUTES.HOME]}>
    <Routes>
      <Route path={ROUTES.HOME} element={children} />
      <Route path={ROUTES.DASHBOARD} element={children} />
      {/* Add other necessary routes */}
    </Routes>
  </MemoryRouter>
);

describe('useAppNavigation', () => {
  it('navigates to dashboard', async () => {
    const { result } = renderHook(() => useAppNavigation(), { wrapper });
    
    await act(async () => {
      result.current.goToDashboard();
    });
    
    expect(result.current.getCurrentPath()).toBe(ROUTES.DASHBOARD);
  });
});
```

## Effective Assertions

- **Object Equality**: 
  - Use `toMatchObject` instead of `toEqual` for objects with complex types (like `Date`)
  - When testing objects that may change during serialization/deserialization

- **Text Content**: 
  - `expect(element.textContent).toMatch(/pattern/)` can be more reliable than 
  - `expect(element).toHaveTextContent(expect.stringMatching(/pattern/))`

- **State vs. Arguments**: 
  - For functions modifying external state (e.g., `localStorage`), assert the final state rather than function arguments
  - The arguments might not reflect the final intended state due to internal processing

## Common Pitfalls to Avoid

1. **Incomplete Mocking**
   - Not mocking all dependencies
   - Not handling progress callbacks
   - Missing error scenarios
   - Incomplete singleton patterns
   - Not providing all required properties in context mocks

2. **Async Testing Issues**
   - Not waiting for loading states
   - Missing await on async operations
   - Race conditions in tests
   - Improper use of waitFor
   - Not accounting for useEffect side effects

3. **Context-Related Issues**
   - Missing providers in test setup
   - Incorrect provider order
   - Not mocking context values properly

4. **Test Setup Integrity**
   - Data format mismatches between mock setup and actual code
   - Test state leaking between tests
   - Not clearing mocks before each test

## Testing Specific Scenarios

### Testing File Operations
- Mock file system operations
- Don't rely on actual file system
- Test both success and error cases

### Testing Network Requests
- Mock API calls
- Test loading states
- Test error handling
- Verify request payloads

### Testing UI Interactions
- Use user-event over fireEvent when possible
- Test keyboard interactions
- Test accessibility
- Verify state changes

### Testing Forms
- Test validation
- Test submission
- Test error states
- Test loading states

## Debugging Test Issues

1. **When Tests Hang**
   - Check for unresolved promises
   - Verify all async operations are properly mocked
   - Look for infinite loops in component logic
   - Check for missing cleanup in useEffect

2. **When Tests Fail Unexpectedly**
   - Read assertion failures (expected vs. received) carefully
   - Check the implementation of the unit under test AND any utilities it uses
   - Isolate: Run tests individually (`it.only`) or per-file
   - Use console.log sparingly and only when necessary

## Useful Testing Utilities

```typescript
// Wait for multiple conditions
const waitForConditions = async (conditions: (() => Promise<void>)[]) => {
  await Promise.all(conditions.map(condition => waitFor(condition)));
};

// Mock progress handler
const createProgressMock = () => {
  const events: any[] = [];
  return {
    handler: (progress: any) => events.push(progress),
    events
  };
};
```

## Essential Testing Libraries

- @testing-library/react
- @testing-library/user-event
- vitest
- msw (for API mocking)