# React Navigation Architecture Rules

## Core Principles

### 1. Separation of Concerns
- **Auth State**: Managed exclusively by `AuthContext`
- **Navigation Logic**: Centralized in `NavigationService`
- **Route Protection**: Handled by `RouteWrapper`
- **Feature Navigation**: Implemented by feature-specific components

### 2. State Management
```typescript
// ❌ Don't mix auth state and navigation
if (user && isNewUser) {
  navigate(ROUTES.DASHBOARD);
}

// ✅ Do use NavigationService
navigationService.handleVerificationSuccess(isNewUser, state);
```

### 3. Navigation Service Pattern
- Create dedicated services for different navigation domains
- Each service should:
  1. Accept a `NavigateFunction` and current path
  2. Handle state preservation
  3. Provide type-safe navigation methods
  4. Manage navigation side effects

```typescript
class NavigationService {
  constructor(navigate: NavigateFunction, currentPath: string) {
    this.navigate = navigate;
    this.currentPath = currentPath;
  }

  handleNavigation(params: NavigationParams) {
    // Handle state preservation
    // Manage side effects
    // Execute navigation
  }
}
```

## Implementation Rules

### 1. Route Configuration
- Use type-safe route constants
- Define route metadata centrally
- Specify auth requirements explicitly
```typescript
export const ROUTES = {
  HOME: '/',
  DASHBOARD: '/dashboard'
} as const;

export const requiresAuth = (path: string): boolean => {
  // Centralized auth requirements
};
```

### 2. Protected Routes
- Use `RouteWrapper` for consistent protection
- Only check auth requirements, no other logic
- Preserve navigation state during redirects
```typescript
// RouteWrapper should be simple
if (!loading && needsAuth && !user) {
  navigationService.handleProtectedRoute(false);
  return null;
}
```

### 3. Navigation State
- Preserve state during navigation
- Use type-safe state objects
- Handle return paths properly
```typescript
interface NavigationState {
  returnTo?: string;
  from?: string;
  // Feature-specific state
}
```

### 4. Auth Navigation
- Handle auth state changes in `AuthContext`
- Use `NavigationService` for redirects
- Preserve context during auth flows
- Avoid direct navigation in components

### 5. Feature Navigation
- Create feature-specific navigation services
- Handle local state preservation
- Implement proper loading states
- Use consistent patterns across features

## Anti-patterns to Avoid

### 1. Direct Navigation in Components
```typescript
// ❌ Don't use direct navigation
const handleClick = () => {
  navigate(ROUTES.DASHBOARD);
};

// ✅ Do use navigation service
const handleClick = () => {
  navigationService.navigateToFeature();
};
```

### 2. Mixed Responsibilities
```typescript
// ❌ Don't mix auth and navigation
if (user && needsAuth) {
  setLoading(true);
  navigate(ROUTES.DASHBOARD);
}

// ✅ Do separate concerns
// In AuthContext
const handleAuthChange = (user) => {
  setUser(user);
  navigationService.handleAuthChange(user);
};
```

### 3. Inconsistent State Management
```typescript
// ❌ Don't lose state during navigation
navigate(ROUTES.DASHBOARD);

// ✅ Do preserve state
navigate(ROUTES.DASHBOARD, {
  state: { returnTo, from, ...featureState }
});
```

## Testing Requirements

### 1. Navigation Service Tests
- Test state preservation
- Verify navigation paths
- Check error handling
- Test edge cases

### 2. Integration Tests
- Test complete navigation flows
- Verify state preservation
- Check loading states
- Test error recovery

### 3. Auth Flow Tests
- Test protected route access
- Verify auth redirects
- Check state preservation
- Test edge cases

## Error Handling

### 1. Navigation Errors
- Use error boundaries for navigation failures
- Provide recovery paths
- Preserve state during errors
- Log navigation errors properly

### 2. Loading States
- Show loading indicators during navigation
- Handle transition states properly
- Prevent navigation during loading
- Clean up loading states

## Performance Considerations

### 1. Route Code Splitting
- Use lazy loading for routes
- Implement proper loading states
- Handle chunk loading errors
- Optimize initial load time

### 2. State Management
- Minimize state updates during navigation
- Use proper memoization
- Handle cleanup properly
- Prevent memory leaks

## Documentation Requirements

### 1. Navigation Changes
- Document navigation patterns
- Explain state management
- Detail error handling
- Provide usage examples

### 2. Component Integration
- Document navigation hooks
- Explain state requirements
- Detail prop requirements
- Provide integration examples 