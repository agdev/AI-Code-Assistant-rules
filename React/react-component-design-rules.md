# Comprehensive React & TypeScript Best Practices Guide

This guide combines the previously created React/TypeScript best practices with the detailed component design rules from the provided documentation.

## Core Design Principles

### Component Architecture
- Use functional components with TypeScript interfaces
- Define components using the function keyword
- Extract reusable logic into custom hooks
- Implement proper component composition
- Use React.memo() strategically for performance
- Implement proper cleanup in useEffect hooks
- Implement proper focus management for accessibility

### Hook Placement Rules
**Rule: Always call hooks at the top level of your React function components.**

- ✅ **DO** call all hooks (e.g., `useState`, `useEffect`, custom hooks) unconditionally at the top of the component, before any return, conditional logic, or loops.
- ❌ **DON'T** call hooks inside loops, conditions, or after a return statement.

**Correct Usage:**
```tsx
function MyComponent() {
  const [value, setValue] = useState('');
  useEffect(() => { /* ... */ }, []);
  // ...defensive checks and returns here
}
```
**Incorrect Usage:**
```tsx
function MyComponent({ show }) {
  if (show) {
    // ❌ This is invalid!
    useEffect(() => { /* ... */ }, []);
  }
  // ...
}
```

- All hooks must be called on every render, in the same order.
- Defensive or early returns (such as for missing route params) must come after all hooks are called.
- This prevents runtime errors and ensures predictable component behavior.

### File Organization

- **Components**: Use PascalCase for component files (`UserProfile.tsx`)
- **Hooks**: Use camelCase with `use` prefix (`useAuth.ts`)
- **Services**: Use camelCase with descriptive names (`apiService.ts`)
- **Types/Interfaces**: Group in dedicated files or alongside related components

## Component Props Design Rules

### 1. Minimize Prop Dependencies

**Rule: Components should only receive props they directly use.**

- ✅ **DO** pass only the specific data and callbacks a component needs
- ❌ **DON'T** pass entire objects or functions when only parts are needed
- ❌ **DON'T** pass props "just in case" they might be needed in the future

```tsx
// GOOD: Only passing what's needed
<AudienceDefinition
  isAnalyzing={isAnalyzing}
  chatHistory={chatHistory}
  hasValidAnalysis={hasValidAnalysis}
  handleSendMessage={handleSendMessage}
  analysis={analysis}
/>

// BAD: Passing unused props
<AudienceDefinition
  isAnalyzing={isAnalyzing}
  chatHistory={chatHistory}
  hasValidAnalysis={hasValidAnalysis}
  handleSendMessage={handleSendMessage}
  analysis={analysis}
  analyzeProject={analyzeProject} // Unused prop
/>
```

### 2. Never Pass Hooks as Props

**Rule: Hooks should be used within components, not passed as props.**

- ✅ **DO** use hooks within the component that needs them
- ✅ **DO** pass the results of hooks as props when needed by child components
- ❌ **DON'T** pass hook functions (like `useState`, `useEffect`, etc.) as props
- ❌ **DON'T** pass custom hooks as props

```tsx
// GOOD: Using hooks within component
function ParentComponent() {
  const [value, setValue] = useState('');
  return <ChildComponent value={value} onChange={setValue} />;
}

// BAD: Passing hooks as props
function ParentComponent() {
  const [value, setValue] = useState('');
  return <ChildComponent state={[value, setValue]} />;
}
```

### 3. Stabilize Function References

**Rule: Stabilize function references to prevent unnecessary re-renders.**

- ✅ **DO** use `useCallback` for functions passed as props
- ✅ **DO** include all dependencies in the dependency array
- ❌ **DON'T** create new function references on each render for props

```tsx
// GOOD: Stabilized function reference
const handleSendMessage = useCallback(async (message: string) => {
  // Implementation
}, [dependency1, dependency2]);

// BAD: New function reference on each render
const handleSendMessage = async (message: string) => {
  // Implementation
};
```

### 4. Separate Props by Responsibility

**Rule: Group props by their responsibility and create separate objects when appropriate.**

- ✅ **DO** create separate props objects for different component types
- ✅ **DO** use TypeScript interfaces to define prop shapes
- ❌ **DON'T** use a one-size-fits-all approach for all components

```tsx
// GOOD: Separate props objects for different components
const stageProps = {
  isAnalyzing,
  chatHistory,
  hasValidAnalysis,
  handleSendMessage,
  analysis
};

const analyzeProps = {
  ...stageProps,
  analyzeProject
};

// Components receive only what they need
<IdeaInput {...stageProps} handleConfirm={handleConfirmIdea} onUpdateIdea={updateIdea} />
<TechnicalPlan {...analyzeProps} handleConfirm={handleConfirmTechnical} />
```

## State Management Best Practices

### Component State
- Use `useState` for simple component state
- Use `useReducer` for complex state logic
- Extract state logic to custom hooks for reusability

### Global State
- Use Context for sharing state between related components
- Consider libraries like Redux Toolkit for complex applications
- Structure state to avoid deep nesting

```typescript
// Good Context pattern
export const UserContext = createContext<UserContextType | undefined>(undefined);

export function UserProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(false);
  
  // Auth functions here
  
  return (
    <UserContext.Provider value={{ user, loading, login, logout }}>
      {children}
    </UserContext.Provider>
  );
}

export function useUser() {
  const context = useContext(UserContext);
  if (context === undefined) {
    throw new Error('useUser must be used within a UserProvider');
  }
  return context;
}
```

### Parent-Child State Updates

**Rule: Handle state updates directly in callback functions, not through side effects or state monitoring.**

- ✅ **DO** update parent state directly in child component callback functions
- ✅ **DO** pass update functions from parent to child that handle specific state changes
- ❌ **DON'T** use useEffect to monitor prop or state changes for updates
- ❌ **DON'T** split related state updates across multiple effects or callbacks

```tsx
// GOOD: Direct updates in callback
interface ChildProps {
  onAction: (data: ActionData) => Promise<void>;
}

const ChildComponent: FC<ChildProps> = ({ onAction }) => {
  const handleAction = async (data: ActionData) => {
    try {
      await onAction(data);
    } catch (error) {
      handleError(error);
    }
  };
  
  return <Button onClick={() => handleAction(someData)} />;
};

// Parent provides callback that handles both action and state update
const ParentComponent: FC = () => {
  const handleAction = async (data: ActionData) => {
    try {
      const result = await performAction(data);
      updateParentState(result);
    } catch (error) {
      handleError(error);
    }
  };

  return <ChildComponent onAction={handleAction} />;
};
```

## Error Handling

**Rule: Implement consistent error handling patterns and handle errors at the appropriate level.**

- ✅ **DO** handle errors at the component level where they can be properly displayed to the user
- ✅ **DO** create reusable error handling utilities for consistent error processing
- ✅ **DO** provide meaningful error messages that help users understand what went wrong
- ❌ **DON'T** swallow errors without informing the user
- ❌ **DON'T** expose raw error objects or technical details to users

```tsx
// GOOD: Proper error handling at component level
const handleSubmit = async (data) => {
  try {
    await submitData(data);
    setSuccess(true);
  } catch (error) {
    // Extract a user-friendly message
    const errorMessage = extractErrorMessage(error);
    setError(errorMessage);
    
    // Log the full error for debugging
    console.error("Error submitting data:", error);
  }
};

// Helper function in a separate utility file
const extractErrorMessage = (error) => {
  // Default fallback message
  let message = 'An unexpected error occurred. Please try again.';
  
  if (typeof error === 'string') {
    return error;
  }
  
  if (error?.message) {
    message = error.message;
    // Additional processing to make the message user-friendly
  }
  
  return message;
};
```

## Async/Await Patterns

**Rule: Implement consistent and proper async/await patterns to handle asynchronous operations.**

- ✅ **DO** mark callback functions as `async` when they contain `await` operations
- ✅ **DO** properly chain promises with `await` to ensure operations complete in the correct order
- ✅ **DO** handle errors in async functions using try/catch blocks
- ❌ **DON'T** ignore returned promises (avoid "floating promises")
- ❌ **DON'T** use nested callbacks when async/await can flatten the code

```tsx
// GOOD: Proper async/await usage in callbacks
const handleSendMessage = async (message: string) => {
  try {
    await api.sendMessage(message, {
      onResponse: async (response) => {  // Mark as async when using await inside
        try {
          // Process response
          const result = await processResponse(response);
          
          // Update state with processed result
          updateState(result);
          
          console.log('Processing complete');
        } catch (error) {
          console.error('Error in response processing:', error);
          handleError(error);
        }
      }
    });
  } catch (error) {
    console.error('Error sending message:', error);
    handleError(error);
  }
};
```

## Service and Component Dependencies

**Rule: Maintain clear separation between services and UI components through proper dependency management.**

- ✅ **DO** keep domain types in a separate `types` package
- ✅ **DO** have both services and components import shared types from the domain types package
- ✅ **DO** maintain a clear dependency hierarchy (UI Components → Domain Types ← Services)
- ❌ **DON'T** import UI components or their types into services
- ❌ **DON'T** define shared types within component files
- ❌ **DON'T** create circular dependencies between services and components

```typescript
// GOOD: Domain types in separate package
// src/types/chat.ts
export interface ChatMessage {
  role: 'user' | 'assistant';
  content: string;
}

// GOOD: Component importing from domain types
// src/components/Chat.tsx
import { ChatMessage } from '@/types/chat';

// GOOD: Service importing from domain types
// src/services/export/exportService.ts
import { ChatMessage } from '@/types/chat';
```

## Package Import Best Practices

**Rule: Always import from package main entry points, not internal paths.**

- ✅ **DO** import from the main package entry point
- ✅ **DO** use officially exported types and interfaces
- ❌ **DON'T** import from internal library paths (e.g., 'package/lib/...')
- ❌ **DON'T** rely on implementation details of packages

```typescript
// GOOD: Import from main entry point
import type { Components } from 'react-markdown';

// BAD: Import from internal path
import type { NormalComponents } from 'react-markdown/lib/components';
```

## Component Variants and Reusability

**Rule: Prefer modifying existing components with variants over creating wrapper components when functionality overlaps significantly.**

- ✅ **DO** use variant props to handle different modes/behaviors of similar components
- ✅ **DO** consolidate shared logic and UI patterns into a single component
- ✅ **DO** use TypeScript discriminated unions for variant-specific props
- ❌ **DON'T** create wrapper components that only change text or minor behaviors

```tsx
// GOOD: Single component with variants
interface BaseEmailFormProps {
  onSubmit: (email: string) => Promise<void>;
  error?: string;
}

interface SignInEmailFormProps extends BaseEmailFormProps {
  variant: 'sign-in';
  // sign-in specific props
}

interface SignUpEmailFormProps extends BaseEmailFormProps {
  variant: 'sign-up';
  // sign-up specific props
}

type EmailFormProps = SignInEmailFormProps | SignUpEmailFormProps;

const EmailForm: FC<EmailFormProps> = ({ variant, onSubmit, error, ...props }) => {
  const buttonText = variant === 'sign-in' ? 'Sign In' : 'Sign Up';
  const headerText = variant === 'sign-in' 
    ? 'Welcome back!' 
    : 'Create your account';

  return (
    <form onSubmit={handleSubmit}>
      <h2>{headerText}</h2>
      {/* Shared form elements */}
      <Button type="submit">{buttonText}</Button>
    </form>
  );
};
```

## TypeScript Implementation

### Type Export and Import Rules

**Rule: Maintain consistent type exports through index files to ensure proper type resolution and prevent import issues.**

- ✅ **DO** export all shared types through the package's index file
- ✅ **DO** use barrel exports (`export * from './file'`) in index files
- ✅ **DO** maintain a clear hierarchy of type exports
- ❌ **DON'T** import types directly from implementation files
- ❌ **DON'T** duplicate type definitions across files

```typescript
// GOOD: Types defined in their own files
// src/types/project.ts
export interface Project {
  id: string;
  name: string;
  // ... other properties
}

// GOOD: Index file exporting all types
// src/types/index.ts
export * from './project';
export * from './chat';
export * from './supabase';

// GOOD: Components importing from index
// src/components/ProjectList.tsx
import { Project } from '@/types';
```

### TypeScript and Type Management Rules

**Rule: Follow strict type management practices to ensure type safety and maintainability.**

- ✅ **DO** use TypeScript's strict mode
- ✅ **DO** define explicit return types for functions and methods
- ✅ **DO** use type inference for simple variable assignments
- ✅ **DO** use discriminated unions for variant handling
- ❌ **DON'T** use `any` type unless absolutely necessary
- ❌ **DON'T** disable TypeScript checks with `@ts-ignore`
- ❌ **DON'T** mix database and application types in components

```typescript
// GOOD: Clear type definitions and hierarchy
import { Project } from '@/types';
import { DatabaseProject } from '@/types/supabase';

// GOOD: Discriminated union for variants
type ButtonVariant = 'primary' | 'secondary' | 'danger';

interface ButtonProps {
  variant: ButtonVariant;
  onClick: () => void;
  disabled?: boolean;
  children?: React.ReactNode;
}

// GOOD: Explicit return type
const Button: React.FC<ButtonProps> = ({ variant, onClick, disabled, children }): JSX.Element => {
  return (
    <button
      className={`btn-${variant}`}
      onClick={onClick}
      disabled={disabled}
    >
      {children}
    </button>
  );
};
```

### Generic Type Usage Rules

**Rule: Use generics effectively to create reusable, type-safe components and utilities.**

- ✅ **DO** use generics for reusable components
- ✅ **DO** constrain generic types appropriately
- ✅ **DO** provide clear generic parameter names
- ❌ **DON'T** overuse generics when simple types suffice
- ❌ **DON'T** use generics without constraints

```typescript
// GOOD: Clear generic constraints
interface ListProps<T extends { id: string }> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
}

// GOOD: Generic component with constraints
function List<T extends { id: string }>({ items, renderItem }: ListProps<T>) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}
```

## Performance Optimization

- Use `React.memo` for preventing unnecessary renders
- Memoize expensive calculations with `useMemo`
- Memoize callback functions with `useCallback`
- Avoid inline function definitions in render

```typescript
// Good performance practice
const MemoizedComponent = React.memo(({ items, onItemClick }) => {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id} onClick={() => onItemClick(item.id)}>
          {item.name}
        </li>
      ))}
    </ul>
  );
});

function ParentComponent() {
  const [items, setItems] = useState([]);
  
  // Memoized callback
  const handleItemClick = useCallback((id) => {
    console.log('Item clicked:', id);
  }, []);
  
  // Memoized expensive calculation
  const sortedItems = useMemo(() => {
    return [...items].sort((a, b) => a.name.localeCompare(b.name));
  }, [items]);
  
  return <MemoizedComponent items={sortedItems} onItemClick={handleItemClick} />;
}
```

## Styling Best Practices

- Prefer Tailwind CSS for utility-first styling
- Use consistent spacing and color variables
- Follow responsive design principles (mobile-first)

```tsx
// Good Tailwind usage
function Button({ children, primary }: ButtonProps) {
  return (
    <button 
      className={`
        px-4 py-2 rounded font-medium
        ${primary 
          ? 'bg-blue-500 text-white hover:bg-blue-600' 
          : 'bg-gray-200 text-gray-800 hover:bg-gray-300'}
      `}
    >
      {children}
    </button>
  );
}
```

## Accessibility Best Practices

- Use semantic HTML elements
- Implement proper keyboard navigation
- Add appropriate ARIA attributes
- Ensure sufficient color contrast
- Manage focus properly for interactive elements

```tsx
// Good accessibility practice
function SearchBox({ onSearch }: SearchBoxProps) {
  return (
    <div role="search">
      <label htmlFor="search-input">Search:</label>
      <input 
        id="search-input"
        type="search" 
        aria-label="Search for items"
      />
      <button 
        aria-label="Submit search"
        onClick={onSearch}
      >
        Search
      </button>
    </div>
  );
}
```

## Naming Conventions

- Components: PascalCase (`UserProfile`)
- Hooks: camelCase with `use` prefix (`useFormValidation`)
- Event handlers: camelCase with `handle` prefix (`handleSubmit`)
- Boolean variables: camelCase with is/has/should (`isLoading`, `hasError`)

## Testing Best Practices

- Write unit tests for components and hooks
- Test user interactions with React Testing Library
- Mock external dependencies and API calls

```tsx
// Good testing practice
test('renders user information correctly', () => {
  const user = { name: 'John Doe', email: 'john@example.com' };
  render(<UserProfile user={user} />);
  
  expect(screen.getByText('John Doe')).toBeInTheDocument();
  expect(screen.getByText('john@example.com')).toBeInTheDocument();
});
```

## Benefits of Following These Rules

1. **Reduced Re-renders**: Components only re-render when props they actually use change
2. **Better Type Safety**: TypeScript can more accurately check prop usage
3. **Improved Performance**: Fewer unnecessary renders and computations
4. **Enhanced Maintainability**: Clearer component responsibilities and dependencies
5. **Easier Debugging**: Clearer component boundaries and responsibilities
6. **Better User Experience**: Consistent error handling provides clear feedback to users
7. **Predictable Async Behavior**: Properly structured async code prevents race conditions and timing issues

## Additional Resources

- [React Documentation](https://reactjs.org/docs/getting-started.html)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [React TypeScript Cheatsheets](https://react-typescript-cheatsheet.netlify.app/)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)