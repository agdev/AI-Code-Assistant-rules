---
description: when working on supabase deno environment and edge functions
globs: 
alwaysApply: false
---
# Supabase Edge Functions Guidelines

This document outlines the best practices and guidelines for working with Supabase Edge Functions in our project.

## TypeScript and Deno

1. **Deno Namespace**: Supabase Edge Functions run in the Deno runtime environment, which provides the `Deno` namespace globally. When TypeScript shows errors for the `Deno` namespace:
   - Use `// @ts-expect-error Deno namespace is available in Supabase Edge Functions` comments above lines that use the `Deno` namespace
   - Do not create custom type declaration files (like `deno.d.ts`) as they are not needed in the Supabase environment

2. **Built-in Deno.serve**: Always use the built-in `Deno.serve()` function instead of importing `serve` from external modules:
   ```typescript
   // CORRECT: Use the built-in Deno.serve
   Deno.serve(async (req: Request) => {
     // Handler code
   });

   // INCORRECT: Don't import serve from external modules
   // import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
   // serve(async (req) => { ... });
   ```

4. **Environment Variables**: Access environment variables using `Deno.env.get()`:
   ```typescript
   const API_KEY = Deno.env.get('API_KEY');
   ```

## API Structure

1. **CORS Handling**: Always include CORS headers for browser compatibility:
   ```typescript
   // Handle CORS preflight requests
   if (req.method === 'OPTIONS') {
     return new Response(null, { headers: corsHeaders });
   }
   ```

2. **Response Format**: Always return properly formatted JSON responses with appropriate headers:
   ```typescript
   return new Response(
     JSON.stringify({ data: result }),
     { headers: { ...corsHeaders, 'Content-Type': 'application/json' } }
   );
   ```

3. **Error Handling**: Implement comprehensive error handling with informative error messages:
   ```typescript
   try {
     // API logic
   } catch (error) {
     console.error("Error:", error);
     return new Response(
       JSON.stringify({
         error: error instanceof Error ? error.message : "Unknown error occurred"
       }),
       { 
         status: 500,
         headers: { ...corsHeaders, 'Content-Type': 'application/json' }
       }
     );
   }
   ```

## External Dependencies

1. **Import Specifiers**: Always use proper import specifiers for external dependencies:
   - Use `npm:` prefix for npm packages (e.g., `import express from "npm:express@4.18.2"`)
   - Use `jsr:` prefix for JSR packages
   - Avoid bare specifiers

2. **Version Pinning**: Always specify versions for external dependencies to ensure reproducible builds.

3. **Web APIs**: Prefer Web APIs and Deno's core APIs over external dependencies when possible.

## Type Safety

1. **Type Definitions**: Create and maintain comprehensive type definitions for request and response payloads:
   ```typescript
   export interface RequestPayload {
     // Request properties
   }

   export interface ResponsePayload {
     // Response properties
   }
   ```

2. **Enum Values**: Use enums for values that have a fixed set of options to improve type safety.
