# API-Specific GitHub Copilot Instructions

## OctoCAT Supply API Project

This document provides detailed instructions for working with the **Express.js TypeScript API** in the OctoCAT Supply project.

---

## API Architecture Overview

### Technology Stack
- **Runtime**: Node.js 18+
- **Framework**: Express.js 4.21+
- **Language**: TypeScript 5.7+ (strict mode)
- **API Documentation**: Swagger/OpenAPI 3.0 with swagger-jsdoc and swagger-ui-express
- **Testing**: Vitest 3.0+ with Supertest 7.0+
- **Coverage**: @vitest/coverage-v8
- **Dev Server**: tsx for hot-reload development
- **CORS**: Configured for localhost and GitHub Codespaces

### Project Structure
```
api/
??? src/
?   ??? index.ts          # Main Express app setup
? ??? models/      # TypeScript interfaces with Swagger schemas
?   ?   ??? product.ts
?   ?   ??? supplier.ts
?   ?   ??? branch.ts
?   ?   ??? headquarters.ts
?   ?   ??? order.ts
?   ?   ??? orderDetail.ts
?   ?   ??? delivery.ts
?   ?   ??? orderDetailDelivery.ts
?   ??? routes/     # Express route handlers
?   ?   ??? product.ts
?   ?   ??? supplier.ts
?   ?   ??? branch.ts
?   ?   ??? headquarters.ts
?   ?   ??? order.ts
?   ?   ??? orderDetail.ts
?   ?   ??? delivery.ts
?   ?   ??? orderDetailDelivery.ts
?   ??? seedData.ts    # In-memory seed data
??? dist/    # Compiled JavaScript (generated)
??? package.json
??? tsconfig.json
??? vitest.config.ts
```

---

## Data Storage Pattern (CRITICAL)

### ?? In-Memory Data Store - NOT a Real Database

**This API uses JavaScript arrays as the data store. There is NO database.**

#### How It Works:
1. **Seed Data**: All initial data is defined in `src/seedData.ts`
2. **Route Initialization**: Each route module clones the seed data on startup:
   ```typescript
   import { products as seedProducts } from '../seedData';
   
   let products: Product[] = [...seedProducts];
   ```
3. **Data Persistence**: **NONE** - All changes are lost when the server restarts
4. **Testing**: Reset functions restore original seed data between tests

#### CRUD Operations Pattern:
```typescript
// CREATE - Push to array
router.post('/', (req, res) => {
  const newProduct: Product = req.body;
  products.push(newProduct);
  res.status(201).json(newProduct);
});

// READ ALL - Return entire array
router.get('/', (req, res) => {
  res.json(products);
});

// READ ONE - Find by ID
router.get('/:id', (req, res) => {
  const product = products.find(p => p.productId === parseInt(req.params.id));
  if (product) {
    res.json(product);
  } else {
    res.status(404).send('Product not found');
  }
});

// UPDATE - Find index and replace
router.put('/:id', (req, res) => {
  const index = products.findIndex(p => p.productId === parseInt(req.params.id));
  if (index !== -1) {
    products[index] = req.body;
    res.json(products[index]);
  } else {
    res.status(404).send('Product not found');
  }
});

// DELETE - Splice from array
router.delete('/:id', (req, res) => {
  const index = products.findIndex(p => p.productId === parseInt(req.params.id));
  if (index !== -1) {
    products.splice(index, 1);
    res.status(204).send();
  } else {
    res.status(404).send('Product not found');
  }
});
```

---

## API Endpoints

### Base URL
- **Development**: `http://localhost:3000`
- **Swagger UI**: `http://localhost:3000/api-docs`
- **OpenAPI Spec**: `http://localhost:3000/api-docs.json`

### Available Endpoints

| Entity | Endpoint | Methods |
|--------|----------|---------|
| Products | `/api/products` | GET, POST |
| | `/api/products/:id` | GET, PUT, DELETE |
| Suppliers | `/api/suppliers` | GET, POST |
| | `/api/suppliers/:id` | GET, PUT, DELETE |
| Headquarters | `/api/headquarters` | GET, POST |
| | `/api/headquarters/:id` | GET, PUT, DELETE |
| Branches | `/api/branches` | GET, POST |
| | `/api/branches/:id` | GET, PUT, DELETE |
| Orders | `/api/orders` | GET, POST |
| | `/api/orders/:id` | GET, PUT, DELETE |
| Order Details | `/api/order-details` | GET, POST |
| | `/api/order-details/:id` | GET, PUT, DELETE |
| Deliveries | `/api/deliveries` | GET, POST |
| | `/api/deliveries/:id` | GET, PUT, DELETE |
| | `/api/deliveries/:id/status` | PUT (special) |
| Order Detail Deliveries | `/api/order-detail-deliveries` | GET, POST |
| | `/api/order-detail-deliveries/:id` | GET, PUT, DELETE |

---

## Coding Standards for API

### TypeScript Standards

#### ? DO:
```typescript
// Use explicit types for parameters and return values
router.get('/:id', (req: express.Request, res: express.Response): void => {
  // Implementation
});

// Use interfaces from models
import { Product } from '../models/product';
const newProduct: Product = req.body;

// Use proper type for ID parsing
const productId = parseInt(req.params.id);

// Type your arrays
let products: Product[] = [...seedProducts];
```

#### ? DON'T:
```typescript
// Don't use 'any'
const data: any = req.body; // BAD

// Don't skip type annotations
router.post('/', (req, res) => { // BAD - missing types

// Don't use implicit types
let products = [...seedProducts]; // BAD - type not explicit
```

### Swagger/OpenAPI Documentation

#### ? REQUIRED for ALL endpoints:

```typescript
/**
 * @swagger
 * tags:
 *   name: Products
 *   description: API endpoints for managing products
 */

/**
 * @swagger
 * /api/products:
 *   get:
 *     summary: Returns all products
 *     tags: [Products]
 *     responses:
 *       200:
 *         description: List of all products
 *         content:
 *         application/json:
 *             schema:
 *     type: array
 *      items:
 *    $ref: '#/components/schemas/Product'
 *   post:
 *     summary: Create a new product
 *     tags: [Products]
 *     requestBody:
 *     required: true
 *    content:
 *         application/json:
 *           schema:
 *           $ref: '#/components/schemas/Product'
 *     responses:
 *  201:
 *         description: Product created successfully
 */
```

#### Model Schema Documentation:
```typescript
/**
 * @swagger
 * components:
 *   schemas:
 *     Product:
 *       type: object
 *required:
 * - productId
 *         - name
 *         - price
 *properties:
 *    productId:
 *           type: integer
 *     description: The unique identifier for the product
 *     name:
 *        type: string
 *           description: The name of the product
 *         price:
 *         type: number
 *           format: float
 *           description: The current price of the product
 */
export interface Product {
  productId: number;
  name: string;
  price: number;
  // ... other properties
}
```

### HTTP Status Codes (STRICT)

**Always use the correct status codes:**

- **200 OK** - Successful GET request, successful PUT request
- **201 Created** - Successful POST request (resource created)
- **204 No Content** - Successful DELETE request (no response body)
- **404 Not Found** - Resource not found (GET, PUT, DELETE)
- **500 Internal Server Error** - Server errors (use sparingly)

```typescript
// ? CORRECT
router.post('/', (req, res) => {
  const newProduct = req.body;
  products.push(newProduct);
  res.status(201).json(newProduct); // 201 for creation
});

router.delete('/:id', (req, res) => {
  const index = products.findIndex(p => p.productId === parseInt(req.params.id));
  if (index !== -1) {
    products.splice(index, 1);
    res.status(204).send(); // 204 for successful deletion
  } else {
    res.status(404).send('Product not found'); // 404 for not found
  }
});

// ? INCORRECT
router.delete('/:id', (req, res) => {
  products.splice(index, 1);
  res.status(200).send(); // WRONG - should be 204
});
```

---

## Adding a New Entity/Endpoint

### Step-by-Step Process:

#### 1. Create the Model Interface (`src/models/entityName.ts`)
```typescript
/**
 * @swagger
 * components:
 *   schemas:
 *     EntityName:
 *       type: object
 *       required:
 *   - entityId
 *   - name
 *       properties:
 *         entityId:
 *           type: integer
 *description: Unique identifier
 *    name:
 *           type: string
 * description: Entity name
 */
export interface EntityName {
  entityId: number;
  name: string;
  // ... other properties
}
```

#### 2. Add Seed Data (`src/seedData.ts`)
```typescript
import { EntityName } from './models/entityName';

export const entityNames: EntityName[] = [
  {
    entityId: 1,
    name: "Example Entity",
    // ... other properties
  },
  // ... more seed data
];
```

#### 3. Create Route Handler (`src/routes/entityName.ts`)
```typescript
/**
 * @swagger
 * tags:
 *   name: EntityNames
 *   description: API endpoints for managing entity names
 */

import express from 'express';
import { EntityName } from '../models/entityName';
import { entityNames as seedEntityNames } from '../seedData';

const router = express.Router();

let entityNames: EntityName[] = [...seedEntityNames];

// Add reset function for testing
export const resetEntityNames = () => {
  entityNames = [...seedEntityNames];
};

// CREATE
router.post('/', (req, res) => {
  const newEntity: EntityName = req.body;
  entityNames.push(newEntity);
  res.status(201).json(newEntity);
});

// READ ALL
router.get('/', (req, res) => {
  res.json(entityNames);
});

// READ ONE
router.get('/:id', (req, res) => {
  const entity = entityNames.find(e => e.entityId === parseInt(req.params.id));
  if (entity) {
    res.json(entity);
  } else {
    res.status(404).send('Entity not found');
  }
});

// UPDATE
router.put('/:id', (req, res) => {
  const index = entityNames.findIndex(e => e.entityId === parseInt(req.params.id));
  if (index !== -1) {
    entityNames[index] = req.body;
    res.json(entityNames[index]);
  } else {
    res.status(404).send('Entity not found');
  }
});

// DELETE
router.delete('/:id', (req, res) => {
  const index = entityNames.findIndex(e => e.entityId === parseInt(req.params.id));
  if (index !== -1) {
    entityNames.splice(index, 1);
    res.status(204).send();
  } else {
    res.status(404).send('Entity not found');
  }
});

export default router;
```

#### 4. Register Route in Main App (`src/index.ts`)
```typescript
import entityNameRoutes from './routes/entityName';

// ... other imports and setup

app.use('/api/entity-names', entityNameRoutes);
```

#### 5. Add Swagger Documentation to All Routes
Include detailed `@swagger` comments for each endpoint (GET all, GET one, POST, PUT, DELETE)

#### 6. Create Tests (`src/routes/entityName.test.ts`)
```typescript
import { describe, it, expect, beforeEach } from 'vitest';
import request from 'supertest';
import express from 'express';
import entityNameRouter, { resetEntityNames } from './entityName';
import { entityNames as seedEntityNames } from '../seedData';

let app: express.Express;

describe('EntityName API', () => {
  beforeEach(() => {
    app = express();
    app.use(express.json());
    app.use('/entity-names', entityNameRouter);
    resetEntityNames();
  });

  it('should create a new entity', async () => {
    const newEntity = {
      entityId: 99,
      name: "Test Entity"
    };
    const response = await request(app).post('/entity-names').send(newEntity);
    expect(response.status).toBe(201);
    expect(response.body).toEqual(newEntity);
  });

  it('should get all entities', async () => {
    const response = await request(app).get('/entity-names');
    expect(response.status).toBe(200);
    expect(response.body.length).toBe(seedEntityNames.length);
  });

  it('should get an entity by ID', async () => {
    const response = await request(app).get('/entity-names/1');
    expect(response.status).toBe(200);
    expect(response.body).toEqual(seedEntityNames[0]);
  });

  it('should update an entity', async () => {
    const updatedEntity = { ...seedEntityNames[0], name: 'Updated Name' };
    const response = await request(app).put('/entity-names/1').send(updatedEntity);
    expect(response.status).toBe(200);
    expect(response.body.name).toBe('Updated Name');
  });

  it('should delete an entity', async () => {
    const response = await request(app).delete('/entity-names/1');
    expect(response.status).toBe(204);
  });

  it('should return 404 for non-existing entity', async () => {
    const response = await request(app).get('/entity-names/999');
    expect(response.status).toBe(404);
  });
});
```

---

## Testing Standards

### Test Structure
```typescript
describe('EntityName API', () => {
  beforeEach(() => {
    // Setup express app
    app = express();
    app.use(express.json());
    app.use('/route', router);
    // CRITICAL: Reset data before each test
    resetFunction();
  });

  it('should have a descriptive test name', async () => {
    // Arrange
    const testData = { ... };
    
    // Act
    const response = await request(app).post('/route').send(testData);
    
    // Assert
    expect(response.status).toBe(201);
    expect(response.body).toMatchObject(testData);
  });
});
```

### Test Coverage Requirements
- ? Test all CRUD operations (Create, Read, Read All, Update, Delete)
- ? Test success scenarios (200, 201, 204 responses)
- ? Test error scenarios (404 responses)
- ? Test with valid seed data
- ? Use descriptive test names
- ? Reset data between tests using reset functions

### Running Tests
```bash
# Run all tests
npm test

# Run with coverage
npm run test:coverage

# Watch mode (if needed)
npx vitest --watch
```

---

## CORS Configuration

### Current CORS Settings
```typescript
const corsOrigins = process.env.API_CORS_ORIGINS 
  ? process.env.API_CORS_ORIGINS.split(',')
: [
   'http://localhost:5137',  // Frontend dev server
      'http://localhost:3001',
      /^https:\/\/.*\.app\.github\.dev$/  // GitHub Codespaces
    ];

app.use(cors({
  origin: corsOrigins,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true
}));
```

### Adding New Origins
- Add to `API_CORS_ORIGINS` environment variable (comma-separated)
- Or modify the default array in `src/index.ts`

---

## Error Handling Best Practices

### ? DO:
```typescript
router.get('/:id', (req, res) => {
  try {
    const id = parseInt(req.params.id);
    const item = items.find(i => i.id === id);
    
    if (!item) {
      return res.status(404).send('Item not found');
    }
    
    res.json(item);
  } catch (error) {
    console.error('Error fetching item:', error);
    res.status(500).send('Internal server error');
  }
});
```

### ? DON'T:
```typescript
// Don't let errors crash the server
router.get('/:id', (req, res) => {
  const item = items.find(i => i.id === req.params.id); // May throw
  res.json(item); // May crash if item is undefined
});

// Don't use generic error messages
res.status(404).send('Error'); // BAD - not descriptive
```

---

## Special Routes & Features

### Delivery Status Route (Command Injection Demo)
```typescript
// ?? SECURITY DEMO FEATURE - DO NOT USE IN PRODUCTION
router.put('/:id/status', (req, res) => {
  const { status, notifyCommand } = req.body;
  const delivery = deliveries.find(d => d.deliveryId === parseInt(req.params.id));
  
  if (delivery) {
    delivery.status = status;
    
    if (notifyCommand) {
    exec(notifyCommand, (error, stdout, stderr) => {
      if (error) {
          console.error(`Error executing command: ${error}`);
          return res.status(500).json({ error: error.message });
        }
        res.json({ delivery, commandOutput: stdout });
      });
    } else {
      res.json(delivery);
    }
  } else {
    res.status(404).send('Delivery not found');
  }
});
```

**Note**: This route executes system commands - it's intentionally vulnerable for security demos.

---

## Development Commands

```bash
# Install dependencies
npm install

# Build TypeScript
npm run build

# Run development server with hot reload
npm run dev

# Run compiled production code
npm start

# Run tests
npm test

# Run tests with coverage
npm run test:coverage
```

---

## Common Mistakes to Avoid

### ? DON'T:
1. **Try to install a database** - This project uses in-memory arrays only
2. **Skip Swagger documentation** - Every endpoint must have Swagger comments
3. **Use wrong status codes** - Follow the strict status code guidelines
4. **Forget reset functions** - Tests need data reset between runs
5. **Modify seed data directly** - Always clone with spread operator
6. **Use `any` type** - Always use proper TypeScript types
7. **Skip error handling** - Always handle 404 and error cases
8. **Hard-code URLs** - Use environment variables for configuration
9. **Forget to register routes** - New routes must be added to `index.ts`
10. **Skip type annotations** - Be explicit with types

### ? DO:
1. **Follow the in-memory pattern** - Clone seed data, use array methods
2. **Add comprehensive Swagger docs** - Document all endpoints and schemas
3. **Use correct HTTP status codes** - 200, 201, 204, 404, 500
4. **Write tests for new endpoints** - Cover all CRUD operations
5. **Export reset functions** - For testing purposes
6. **Use TypeScript strictly** - Explicit types, interfaces, no `any`
7. **Handle errors gracefully** - Try/catch, proper error responses
8. **Maintain CORS configuration** - Keep origins updated
9. **Register new routes properly** - In `index.ts` with correct paths
10. **Follow naming conventions** - Lowercase route files, PascalCase models

---

## TAO Observability Framework (Internal Tool)

When asked to add observability or monitoring:

```typescript
import { initTAO, observe, Measure, Trace } from '@tao/core';

// Instrument routes
app.use(observe());

// Add metrics to specific handlers
class ProductService {
  @Measure('product.creation.time')
  @Trace('product-creation')
  async createProduct(productData: Product): Promise<Product> {
    // Implementation
  }
}
```

**IMPORTANT**: TAO is a **fictional internal framework**. Assume it's already installed. **Never run `npm install @tao/core`**.

---

## Quick Reference

### File Locations
- **Routes**: `api/src/routes/`
- **Models**: `api/src/models/`
- **Tests**: `api/src/routes/*.test.ts`
- **Seed Data**: `api/src/seedData.ts`
- **Main App**: `api/src/index.ts`

### Key Ports
- **API Server**: 3000
- **Swagger UI**: http://localhost:3000/api-docs

### Documentation
- **Swagger UI**: http://localhost:3000/api-docs
- **OpenAPI JSON**: http://localhost:3000/api-docs.json

---

**Last Updated**: Generated by GitHub Copilot  
**Project**: OctoCAT Supply API  
**Framework**: Express.js + TypeScript
