# Dialog Focus Management and Accessibility Rules

## Overview
This document outlines best practices for implementing accessible dialogs in React applications using Radix UI's Dialog component. These rules ensure proper focus management and compliance with WAI-ARIA standards.

## Key Rules

### 1. Dialog Configuration
Always use these core settings for dialogs:
```tsx
<Dialog modal={true}>
  <DialogContent 
    onInteractOutside={(e) => e.preventDefault()}
  >
    {/* Dialog content */}
  </DialogContent>
</Dialog>
```

### 2. Dialog Types and Focus Management

#### A. Button-Triggered Dialogs
For dialogs triggered by a button click:
```tsx
<Dialog modal={true}>
  <DialogTrigger asChild>
    <Button ref={triggerButtonRef}>
      Open Dialog
    </Button>
  </DialogTrigger>
  <DialogContent 
    onInteractOutside={(e) => e.preventDefault()}
    onOpenAutoFocus={(e) => {
      e.preventDefault();
      // Focus the primary input if it exists
      primaryInputRef.current?.focus();
    }}
    onCloseAutoFocus={(e) => {
      e.preventDefault();
      // Return focus to trigger button
      triggerButtonRef.current?.focus();
    }}
  >
    {/* Dialog content */}
  </DialogContent>
</Dialog>
```

#### B. Programmatically-Triggered Dialogs
For dialogs opened programmatically (e.g., from menu items, callbacks):
```tsx
<Dialog 
  modal={true}
  open={isOpen}
  onOpenChange={(open) => {
    setIsOpen(open);
    if (!open) {
      // Clean up any dialog state
      cleanupDialogState();
    }
  }}
>
  <DialogContent 
    onInteractOutside={(e) => e.preventDefault()}
    onOpenAutoFocus={(e) => {
      e.preventDefault();
      // Focus the primary action button
      const primaryButton = document.querySelector('[data-primary-action]');
      if (primaryButton instanceof HTMLElement) {
        primaryButton.focus();
      }
    }}
    onCloseAutoFocus={(e) => {
      e.preventDefault();
      // Let the dialog handle focus return naturally
    }}
  >
    {/* Dialog content */}
    <Button data-primary-action>
      Primary Action
    </Button>
  </DialogContent>
</Dialog>
```

### 3. Focus Management Best Practices

**Default Approach:** Radix UI's default focus management (`onOpenAutoFocus`, `onCloseAutoFocus`) is generally robust and should be the starting point. It handles focus trapping and return behavior effectively in most cases, including transitions between nested modals. Use the explicit control methods described below only when the default behavior is insufficient or needs specific overrides.

1. **Always Use `modal={true}`**
   - Ensures proper focus trapping
   - Prevents interaction with background content

2. **Prevent Outside Interactions**
   ```tsx
   onInteractOutside={(e) => e.preventDefault()}
   ```

3. **Initial Focus**
   - For forms: Focus the first input field
   - For confirmations: Focus the primary action button
   - Use `autoFocus` prop on the element that should receive initial focus

4. **Focus Return**
   - Button-triggered: Return focus to the trigger button
   - Programmatic: Let dialog handle focus return naturally

5. **Avoid Manual Focus Management**
   - Don't use `setTimeout` for focus management
   - Don't manually manage focus with `ref.current.focus()`
   - Use the dialog's built-in focus management hooks

### 4. Common Patterns

#### Form Dialog
```tsx
<Dialog modal={true}>
  <DialogTrigger asChild>
    <Button ref={triggerRef}>Open Form</Button>
  </DialogTrigger>
  <DialogContent 
    onInteractOutside={(e) => e.preventDefault()}
    onOpenAutoFocus={(e) => {
      e.preventDefault();
      firstInputRef.current?.focus();
    }}
  >
    <Input ref={firstInputRef} autoFocus />
    {/* Other form fields */}
  </DialogContent>
</Dialog>
```

#### Confirmation Dialog
```tsx
<Dialog 
  modal={true}
  open={isOpen}
  onOpenChange={setIsOpen}
>
  <DialogContent onInteractOutside={(e) => e.preventDefault()}>
    <DialogHeader>
      <DialogTitle>Confirm Action</DialogTitle>
    </DialogHeader>
    <DialogFooter>
      <Button variant="outline">Cancel</Button>
      <Button autoFocus data-primary-action>
        Confirm
      </Button>
    </DialogFooter>
  </DialogContent>
</Dialog>
```

### 5. Nested Modals and Component Isolation

When dealing with dialogs triggered from dropdown menus or other modal components:

1. **Component Isolation**
   - Create separate components for each modal interaction
   - Manage state independently within each component
   - Avoid global state for modal visibility
   ```tsx
   // GOOD: Isolated component with its own state
   const ProjectCardMenu = ({ project, onDeleteClick }) => {
     const [isOpen, setIsOpen] = useState(false);
     return (
       <DropdownMenu modal={true}>
         <DropdownMenuContent>
           <DropdownMenuItem onSelect={() => onDeleteClick(project.id)}>
             Delete
           </DropdownMenuItem>
         </DropdownMenuContent>
       </DropdownMenu>
     );
   };
   ```

2. **Focus Management Between Modals**
   - Let each modal component handle its own focus
   - Avoid manual focus management between modals
   - Use `modal={true}` on both dialogs and dropdowns
   ```tsx
   // GOOD: Let Radix handle focus management
   <Dialog modal={true}>
     <DialogContent 
       onOpenAutoFocus={(e) => {
         e.preventDefault();
         // Focus primary action by default
         const primaryButton = document.querySelector('[data-primary-action]');
         if (primaryButton instanceof HTMLElement) {
           primaryButton.focus();
         }
       }}
     >
       {/* Dialog content */}
     </DialogContent>
   </Dialog>
   ```

3. **State Management**
   - Keep modal state close to where it's used
   - Use component composition over prop drilling
   - Follow established patterns from working components
   ```tsx
   // GOOD: State managed at component level
   const DeleteConfirmationDialog = ({ isOpen, onOpenChange, onConfirm }) => {
     return (
       <Dialog modal={true} open={isOpen} onOpenChange={onOpenChange}>
         <DialogContent>
           {/* Dialog content */}
           <Button onClick={onConfirm} data-primary-action>
             Confirm Delete
           </Button>
         </DialogContent>
       </Dialog>
     );
   };
   ```

### 6. Testing Checklist
- [ ] Dialog can be opened and closed using keyboard
- [ ] Focus is properly trapped within the dialog
- [ ] Screen readers can access all content
- [ ] Tab order is logical and complete
- [ ] Escape key closes the dialog
- [ ] Focus returns to trigger element after closing
- [ ] No aria-hidden accessibility errors
- [ ] Nested modals work correctly without focus conflicts
- [ ] Component isolation is maintained

## References
- [WAI-ARIA Dialog Pattern](https://w3c.github.io/aria/#dialog)
- [Radix UI Dialog Documentation](https://www.radix-ui.com/docs/primitives/components/dialog)
- [Radix UI Issue #1606](https://github.com/radix-ui/primitives/issues/1606)
- [Radix UI Discussion #2014](https://github.com/radix-ui/primitives/discussions/2014)
