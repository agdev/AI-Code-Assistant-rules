# Dependency Management Rules

## Version Compatibility

### React Version Compatibility
- Project uses React version defined in package.json
- All dependencies must be compatible with the project's React version
- Prefer libraries that explicitly support current React features
- Avoid libraries locked to older React versions

### Library Selection Criteria
1. Version Compatibility
   - Must support current React version and TypeScript
   - Must have explicit peer dependency support for our React version
   - Should not require legacy peer dependencies

2. Maintenance Status
   - Regular updates within last 3 months
   - Open issues being addressed
   - Clear documentation and changelog
   - TypeScript support (preferably written in TypeScript)

3. Package Quality
   - High download counts
   - Good test coverage
   - Few open issues relative to usage
   - Clear security policy

### Adding New Dependencies
Before adding a new dependency:
1. Check React version compatibility with package.json
2. Verify peer dependencies
3. Run `npm install` with `--legacy-peer-deps` only as last resort
4. Document any version-specific requirements
5. Test the integration thoroughly

### Updating Dependencies
1. When making Updates   
   - Update packages to latest compatible versions
   - Run full test suite after updates

2. Breaking Changes
   - Document breaking changes in `CHANGELOG.md`
   - Update affected components and tests
   - Verify UI components still match design system

### Removing Dependencies
1. Before Removal
   - Check for internal usage
   - Verify no critical features depend on it
   - Plan migration path if needed

2. After Removal
   - Clean up related configurations
   - Remove from package.json
   - Document in changelog

## Current Stack Requirements
These versions should be checked against package.json for accuracy:
- React: Check package.json 
- TypeScript: Check package.json 
- Node: Check .nvmrc or package.json engines
- Vite: Check package.json 

## Best Practices
1. Use Official Libraries
   - Prefer official React team recommendations
   - Use established UI component libraries
   - Avoid abandoned or unmaintained packages

2. Version Locking
   - Use exact versions for critical dependencies
   - Use caret (^) for minor updates of stable packages
   - Lock versions during critical deployment phases

3. Security
   - Regular security audits
   - Automated vulnerability scanning
   - Quick patching of security issues

4. Performance
   - Monitor bundle size impact
   - Consider tree-shaking support
   - Evaluate performance impact

## Examples

### Good Package Selection
```json
{
  "dependencies": {
    "@radix-ui/react-dialog": "^1.1.6",    // Current project version
    "@testing-library/react": "^14.2.1",    // Compatible with React 18
    "react": "^18.3.1"                      // Project's React version
  }
}
```

### Avoid
```json
{
  "dependencies": {
    "legacy-react-library": "^1.0.0",     // Incompatible with current React
    "unmaintained-package": "^0.1.0",     // Last update 2 years ago
    "react-old-testing": "^7.0.0"         // Requires older React version
  }
}
```

## Enforcement
1. CI/CD Pipeline
   - Version compatibility checks
   - Dependency security scans
   - Bundle size monitoring

2. Code Review
   - Review new dependencies
   - Check version compatibility against package.json
   - Verify necessity of additions

3. Documentation
   - Keep dependency list updated
   - Document version requirements
   - Maintain upgrade guides 