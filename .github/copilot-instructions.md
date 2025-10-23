# GitHub Copilot Instructions for OctoCAT Supply Project

## Project Overview

This is the **OctoCAT Supply Chain Management System** - a TypeScript-based demo application showcasing smart cat products. The project is a monorepo with two main workspaces:

- **API**: Express.js REST API (TypeScript)
- **Frontend**: React 18+ application (TypeScript, Vite, Tailwind CSS)

**Repository**: https://github.com/seandorsett/universe  
**Branch**: main  
**Purpose**: Demonstration application for GitHub Copilot capabilities

---

## Architecture & Technology Stack

### Backend (API)
- **Runtime**: Node.js 18+
- **Framework**: Express.js 4.21+
- **Language**: TypeScript 5.7+
- **Documentation**: Swagger/OpenAPI 3.0
- **Testing**: Vitest with Supertest
- **Data Storage**: In-memory arrays (no database)
- **CORS**: Configured for localhost and GitHub Codespaces

### Frontend
- **Framework**: React 18.3+
- **Build Tool**: Vite 6.2+
- **Language**: TypeScript 5.7+
- **Styling**: Tailwind CSS 3.3+
- **State Management**: React Query 3.39+
- **Routing**: React Router DOM 7.4+
- **UI Components**: Custom components with Tailwind

### DevOps
- **Package Manager**: npm workspaces
- **Development**: Concurrent dev servers (API: 3000, Frontend: 5137)
- **Containerization**: Docker support available

---

## Project Structure

```
universe/
??? api/           # Backend Express API
?   ??? src/
?   ?   ??? index.ts             # Main API entry point
?   ?   ??? models/     # TypeScript entity models
?   ?   ??? routes/      # Express route handlers (CRUD)
?   ?   ??? seedData.ts      # In-memory seed data
?   ??? package.json
?   ??? tsconfig.json
??? frontend/            # React frontend
?   ??? src/
?   ?   ??? components/        # React components
?   ?   ??? context/             # React Context providers
?   ?   ??? api/   # API configuration
?   ?   ??? main.tsx   # Frontend entry point
?   ??? public/      # Static assets
?   ??? package.json
?   ??? vite.config.ts
??? docs/    # Documentation
?   ??? architecture.md
?   ??? build.md
?   ??? tao.md             # Fictional observability framework
?   ??? design/       # Design mockups
??? .github/
?   ??? prompts/              # Custom Copilot prompts
?   ??? workflows/       # GitHub Actions
??? package.json     # Root workspace config
??? README.md
```

---

## Data Model & Entities

The system manages these entities (all stored in-memory):

1. **Headquarters** - Corporate HQ locations
2. **Branch** - Retail branch locations (references Headquarters)
3. **Supplier** - Product suppliers
4. **Product** - Smart cat products (references Supplier)
5. **Order** - Branch orders (references Branch)
6. **OrderDetail** - Order line items (references Order & Product)
7. **Delivery** - Supplier deliveries (references Supplier)
8. **OrderDetailDelivery** - Junction table (references OrderDetail & Delivery)

### Entity Relationships (ERD)
```
Headquarters (1) ??< (N) Branch
Branch (1) ??< (N) Order
Order (1) ??< (N) OrderDetail
OrderDetail (N) >?? (1) Product
Product (N) >?? (1) Supplier
Supplier (1) ??< (N) Delivery
OrderDetail (N) ??< (N) Delivery (via OrderDetailDelivery)
```

---

## Coding Standards & Conventions

### TypeScript Guidelines
- **Always use TypeScript** - No plain JavaScript files
- **Strict mode enabled** - Follow strict type checking
- **Interface over Type** - Use interfaces for object shapes when possible
- **Explicit return types** - Always specify return types for functions
- **No `any`** - Avoid using `any` type; use `unknown` or proper types

### API Development
- **RESTful conventions** - Follow REST principles for all endpoints
- **Swagger documentation** - All API endpoints must have Swagger/JSDoc comments
- **HTTP status codes**:
  - `200 OK` - Successful GET, PUT
  - `201 Created` - Successful POST
  - `204 No Content` - Successful DELETE
  - `404 Not Found` - Resource not found
  - `500 Internal Server Error` - Server errors
- **Error handling** - Always handle errors gracefully with appropriate status codes
- **CORS configuration** - Maintain CORS settings for localhost and Codespaces

### Frontend Development
- **Functional components** - Use React functional components with hooks
- **TypeScript props** - Always define proper TypeScript interfaces for props
- **Tailwind CSS** - Use Tailwind utility classes for styling
- **Dark mode support** - Respect dark mode context using `useTheme()` hook
- **Accessibility** - Include proper ARIA labels and semantic HTML
- **React Query** - Use React Query for data fetching and caching
- **Loading states** - Show loading indicators for async operations
- **Error boundaries** - Handle errors gracefully in UI

### Testing Standards
- **Framework**: Vitest for both API and Frontend
- **API testing**: Use Supertest for route testing
- **Test structure**: Describe blocks for grouping, clear test names
- **Coverage**: Aim for comprehensive CRUD operation coverage
- **Reset functions**: Use reset functions in tests to restore seed data
- **Assertions**: Use descriptive expect statements

### File Naming
- **Components**: PascalCase (e.g., `ProductCard.tsx`)
- **Utilities**: camelCase (e.g., `apiConfig.ts`)
- **Routes**: lowercase (e.g., `product.ts`)
- **Models**: PascalCase (e.g., `Product.ts`)
- **Tests**: Same name as file + `.test.ts` (e.g., `product.test.ts`)

---

## Development Workflow

### Starting the Application
```bash
# Install dependencies
npm install

# Build both workspaces
npm run build

# Run both API and Frontend concurrently
npm run dev

# Or run individually
npm run dev:api   # API on http://localhost:3000
npm run dev:frontend  # Frontend on http://localhost:5137
```

### Testing
```bash
# Run all tests
npm test

# API tests only
npm run test:api

# Frontend tests only
npm run test:frontend

# Coverage report
npm run test:coverage --workspace=api
```

### API Endpoints Base URLs
- **Local**: `http://localhost:3000`
- **Swagger UI**: `http://localhost:3000/api-docs`
- **OpenAPI JSON**: `http://localhost:3000/api-docs.json`

### Frontend URLs
- **Local**: `http://localhost:5137`
- **Codespaces**: Auto-configured via environment variables

---

## Important Implementation Details

### In-Memory Data Store Pattern
- **NO REAL DATABASE** - All data is stored in JavaScript arrays
- **Data initialization**: Each route module clones seed data on startup
  ```typescript
  let products: Product[] = [...seedProducts];
  ```
- **Data persistence**: None - all changes lost on restart
- **CRUD operations**: Use native JavaScript array methods
  - Create: `array.push(item)`
  - Read: `array.find()` or return whole array
  - Update: `array.findIndex()` then replace
  - Delete: `array.splice(index, 1)`
- **Testing**: Reset functions restore original seed data

### Special Route Features
- **Delivery route** has a special `/api/deliveries/:id/status` endpoint that can execute system commands (?? security demo feature)

### Authentication
- **Mock authentication** - Login is simulated, not real
- **Admin users**: Email ending with `@github.com`
- **No password validation** - Any password accepted (demo only)

### Theme System
- **Dark/Light mode** - Use `useTheme()` hook in components
- **Transition classes**: Add `transition-colors duration-300` for smooth theme changes
- **Context**: `ThemeContext` provides `darkMode` boolean and `toggleTheme()` function

---

## Custom Copilot Features Reference

### Internal Observability Framework (TAO)
- **Fictional library** - TAO (TypeScript API Observability) is NOT a real package
- **Purpose**: Demonstrates custom instructions for internal tools
- **Documentation**: See `docs/tao.md`
- **Usage**: When asked to add observability, reference TAO patterns
- **Important**: Never try to install TAO - assume it's already available

### Custom Prompt Files
Available in `.github/prompts/`:
- `demo-cart-page.prompt.md` - Cart implementation workflow
- `demo-unit-test-coverage.prompt.md` - Test generation workflow
- `plan.prompt.md` - Planning mode
- Other custom prompts for specific workflows

---

## Common Tasks & Patterns

### Adding a New API Endpoint
1. Define model interface in `api/src/models/`
2. Add seed data in `api/src/seedData.ts`
3. Create route file in `api/src/routes/`
4. Add Swagger documentation comments
5. Implement CRUD operations using array methods
6. Register route in `api/src/index.ts`
7. Add tests in `api/src/routes/*.test.ts`

### Adding a New React Component
1. Create component in `frontend/src/components/`
2. Define TypeScript interfaces for props
3. Use Tailwind CSS for styling
4. Respect dark mode with `useTheme()` hook
5. Add proper accessibility attributes
6. Handle loading and error states

### Creating Tests
1. Use Vitest's `describe` and `it` structure
2. Import necessary modules and mocks
3. Set up test data and reset functions
4. Test all CRUD operations
5. Verify status codes and response data
6. Include error scenarios (404, etc.)

---

## Known Limitations & Considerations

?? **Data Persistence**: All data is lost on server restart  
?? **Scalability**: Single instance only, no horizontal scaling  
?? **Security**: Mock authentication for demo purposes only  
?? **Command Injection**: Delivery status endpoint accepts commands (security demo feature)  
?? **No Database**: No SQL/NoSQL database - pure in-memory storage  
?? **Cart Functionality**: Currently not fully implemented (TODO feature)  

---

## When Working on This Project

### DO:
? Use TypeScript for all new code  
? Add Swagger documentation to new API endpoints  
? Follow existing patterns for CRUD operations  
? Write tests for new features  
? Maintain dark mode support in UI components  
? Use React Query for data fetching  
? Handle loading and error states  
? Follow RESTful API conventions  
? Use proper HTTP status codes  

### DON'T:
? Try to add a real database without explicit instruction  
? Remove or modify seed data without updating all references  
? Use inline styles instead of Tailwind classes  
? Ignore TypeScript errors  
? Skip Swagger documentation for API endpoints  
? Use class components (use functional components)  
? Hard-code API URLs (use config from `frontend/src/api/config.ts`)  
? Attempt to install TAO observability framework (it's fictional)  

---

## Environment & Configuration

### Environment Variables
- `PORT` - API port (default: 3000)
- `API_CORS_ORIGINS` - Comma-separated CORS origins
- `CODESPACE_NAME` - Auto-set in GitHub Codespaces

### Port Configuration
- **API**: 3000 (or `process.env.PORT`)
- **Frontend**: 5137 (configured in `vite.config.ts`)
- **Codespaces**: Auto-forwarded and configured

---

## Getting Help

### Documentation
- Architecture: `docs/architecture.md`
- Build Instructions: `docs/build.md`
- TAO Framework: `docs/tao.md`
- README: `README.md`

### API Documentation
- Swagger UI: http://localhost:3000/api-docs
- OpenAPI spec: http://localhost:3000/api-docs.json

### Demo Scripts
- See `docs/demo-script.md` for demo scenarios
- Custom prompts in `.github/prompts/`

---

## Version Information

- **Node.js**: >= 18
- **TypeScript**: 5.7+
- **React**: 18.3+
- **Express**: 4.21+
- **Vite**: 6.2+

---

**Last Updated**: Generated by GitHub Copilot  
**Workspace**: C:\temp\universe\  
**Repository**: https://github.com/seandorsett/universe
