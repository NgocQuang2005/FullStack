# Fastify Backend Development - Complete Learning Roadmap

## Giai đoạn 1: Nền tảng cốt lõi (Tuần 1-2)

### 1.1 Node.js Fundamentals (3-4 ngày)
**Từ khóa tìm hiểu:**
- Event Loop và Non-blocking I/O
- CommonJS vs ES Modules
- Async/Await và Promises
- HTTP Module cơ bản

**Ví dụ thực hành:**
```javascript
// Tạo HTTP server cơ bản
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ message: 'Hello World' }));
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### 1.2 Fastify Core (4-5 ngày)
**Từ khóa tìm hiểu:**
- Fastify instance và lifecycle
- Routes và HTTP methods
- Hooks system (onRequest, preHandler, onSend, onResponse)
- Plugin system architecture
- Decorators
- Context object (request, reply)

**Ví dụ từng bước:**

**Bước 1 - Server cơ bản:**
```javascript
const fastify = require('fastify')({ logger: true });

// Route đơn giản
fastify.get('/', async (request, reply) => {
  return { hello: 'world' };
});

// Start server
const start = async () => {
  try {
    await fastify.listen({ port: 3000 });
  } catch (err) {
    fastify.log.error(err);
    process.exit(1);
  }
};
start();
```

**Bước 2 - Multiple routes:**
```javascript
// GET với params
fastify.get('/users/:id', async (request, reply) => {
  const { id } = request.params;
  return { userId: id };
});

// POST với body
fastify.post('/users', async (request, reply) => {
  const userData = request.body;
  return { created: userData };
});

// PUT, DELETE tương tự
```

**Bước 3 - Hooks system:**
```javascript
// Global hook
fastify.addHook('onRequest', async (request, reply) => {
  console.log('Request received:', request.method, request.url);
});

// Route-specific hook
fastify.get('/protected', {
  preHandler: async (request, reply) => {
    // Validation logic here
    console.log('Pre-handler executed');
  }
}, async (request, reply) => {
  return { message: 'Protected route' };
});
```

## Giai đoạn 2: Environment & Configuration (Tuần 2)

### 2.1 Environment Variables với dotenv (1-2 ngày)
**Từ khóa tìm hiểu:**
- Environment variables
- Configuration management
- Sensitive data handling
- Different environments (dev, staging, prod)

**Ví dụ thực hành:**
```javascript
// .env file
PORT=3000
DB_HOST=localhost
DB_USER=myuser
DB_PASS=mypassword
JWT_SECRET=mysecretkey

// server.js
require('dotenv').config();

const fastify = require('fastify')({ 
  logger: process.env.NODE_ENV !== 'production' 
});

const PORT = process.env.PORT || 3000;
const DB_CONFIG = {
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASS
};
```

## Giai đoạn 3: Database Layer (Tuần 3-4)

### 3.1 Knex.js - Query Builder (5-7 ngày)
**Từ khóa tìm hiểu:**
- Query builder pattern
- Migrations và Seeds
- Schema definition
- Connection pooling
- Transaction management
- Raw queries vs Builder

**Ví dụ từng bước:**

**Bước 1 - Setup và Connection:**
```javascript
// knexfile.js
module.exports = {
  development: {
    client: 'mssql',
    connection: {
      server: process.env.DB_HOST,
      database: process.env.DB_NAME,
      user: process.env.DB_USER,
      password: process.env.DB_PASS,
      options: {
        encrypt: true,
        trustServerCertificate: true
      }
    },
    migrations: {
      directory: './migrations'
    },
    seeds: {
      directory: './seeds'
    }
  }
};

// database.js
const knex = require('knex');
const config = require('./knexfile');

const db = knex(config.development);
module.exports = db;
```

**Bước 2 - Migrations:**
```javascript
// Migration: create_users_table
exports.up = function(knex) {
  return knex.schema.createTable('users', function(table) {
    table.increments('id').primary();
    table.string('email').unique().notNullable();
    table.string('password').notNullable();
    table.string('name').notNullable();
    table.timestamp('created_at').defaultTo(knex.fn.now());
    table.timestamp('updated_at').defaultTo(knex.fn.now());
  });
};

exports.down = function(knex) {
  return knex.schema.dropTable('users');
};
```

**Bước 3 - CRUD Operations:**
```javascript
// User model/service
class UserService {
  constructor(db) {
    this.db = db;
    this.table = 'users';
  }

  async create(userData) {
    const [id] = await this.db(this.table).insert(userData).returning('id');
    return this.findById(id);
  }

  async findById(id) {
    return await this.db(this.table).where({ id }).first();
  }

  async findByEmail(email) {
    return await this.db(this.table).where({ email }).first();
  }

  async update(id, userData) {
    await this.db(this.table).where({ id }).update(userData);
    return this.findById(id);
  }

  async delete(id) {
    return await this.db(this.table).where({ id }).del();
  }

  async list(page = 1, limit = 10) {
    const offset = (page - 1) * limit;
    return await this.db(this.table)
      .select('*')
      .limit(limit)
      .offset(offset);
  }
}
```

### 3.2 MSSQL Integration (2-3 ngày)
**Từ khóa tìm hiểu:**
- SQL Server connection
- Connection pooling
- Stored procedures
- Bulk operations
- Error handling

**Ví dụ thực hành:**
```javascript
// Direct MSSQL usage (khi cần stored procedures)
const sql = require('mssql');

const config = {
  server: process.env.DB_HOST,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASS,
  options: {
    encrypt: true,
    trustServerCertificate: true
  }
};

class DatabaseService {
  async executeStoredProcedure(procedureName, params = {}) {
    try {
      await sql.connect(config);
      const request = new sql.Request();
      
      // Add parameters
      Object.keys(params).forEach(key => {
        request.input(key, params[key]);
      });
      
      const result = await request.execute(procedureName);
      return result.recordset;
    } catch (err) {
      throw err;
    }
  }
}
```

## Giai đoạn 4: Validation Layer (Tuần 4-5)

### 4.1 Joi Validation (3-4 ngày)
**Từ khóa tìm hiểu:**
- Schema-based validation
- Custom validation rules
- Conditional validation
- Error messages customization
- Async validation

**Ví dụ từng bước:**

**Bước 1 - Basic Schemas:**
```javascript
const Joi = require('joi');

// User registration schema
const userRegistrationSchema = Joi.object({
  email: Joi.string().email().required().messages({
    'string.email': 'Email phải đúng định dạng',
    'any.required': 'Email là bắt buộc'
  }),
  password: Joi.string().min(6).pattern(new RegExp('^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])(?=.*[!@#\$%\^&\*])')).required().messages({
    'string.min': 'Mật khẩu tối thiểu 6 ký tự',
    'string.pattern.base': 'Mật khẩu phải chứa chữ hoa, chữ thường, số và ký tự đặc biệt'
  }),
  name: Joi.string().min(2).max(50).required(),
  age: Joi.number().integer().min(18).max(120).optional()
});

// User update schema (partial)
const userUpdateSchema = Joi.object({
  email: Joi.string().email().optional(),
  name: Joi.string().min(2).max(50).optional(),
  age: Joi.number().integer().min(18).max(120).optional()
}).min(1); // At least one field required
```

**Bước 2 - Advanced Validation:**
```javascript
// Conditional validation
const productSchema = Joi.object({
  type: Joi.string().valid('physical', 'digital').required(),
  name: Joi.string().required(),
  price: Joi.number().positive().required(),
  weight: Joi.when('type', {
    is: 'physical',
    then: Joi.number().positive().required(),
    otherwise: Joi.forbidden()
  }),
  downloadUrl: Joi.when('type', {
    is: 'digital',
    then: Joi.string().uri().required(),
    otherwise: Joi.forbidden()
  })
});

// Custom validation
const customEmailSchema = Joi.string().external(async (value) => {
  const user = await UserService.findByEmail(value);
  if (user) {
    throw new Error('Email đã tồn tại');
  }
  return value;
});
```

**Bước 3 - Integration với Fastify:**
```javascript
// Validation plugin
async function validationPlugin(fastify, options) {
  fastify.decorate('validate', (schema) => {
    return async (request, reply) => {
      try {
        const { error, value } = schema.validate(request.body, { abortEarly: false });
        if (error) {
          return reply.status(400).send({
            error: 'Validation Error',
            details: error.details.map(d => ({
              field: d.path.join('.'),
              message: d.message
            }))
          });
        }
        request.validatedBody = value;
      } catch (err) {
        return reply.status(400).send({ error: err.message });
      }
    };
  });
}

fastify.register(validationPlugin);

// Usage in routes
fastify.post('/users', {
  preHandler: fastify.validate(userRegistrationSchema)
}, async (request, reply) => {
  const userData = request.validatedBody;
  // userData is now validated and sanitized
  const user = await userService.create(userData);
  return user;
});
```

### 4.2 AJV Validation (2-3 ngày)
**Từ khóa tìm hiểu:**
- JSON Schema validation
- Performance optimization
- Custom keywords
- Format validation
- Schema compilation

**Ví dụ thực hành:**
```javascript
const Ajv = require('ajv');
const addFormats = require('ajv-formats');

const ajv = new Ajv({ allErrors: true });
addFormats(ajv);

// JSON Schema definition
const userSchema = {
  type: 'object',
  properties: {
    email: { 
      type: 'string', 
      format: 'email',
      errorMessage: 'Email không đúng định dạng'
    },
    password: { 
      type: 'string', 
      minLength: 6,
      pattern: '^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])(?=.*[!@#\\$%\\^&\\*])',
      errorMessage: 'Mật khẩu phải chứa chữ hoa, thường, số và ký tự đặc biệt'
    },
    name: { 
      type: 'string', 
      minLength: 2, 
      maxLength: 50 
    }
  },
  required: ['email', 'password', 'name'],
  additionalProperties: false
};

const validateUser = ajv.compile(userSchema);

// Fastify schema validation
fastify.post('/users', {
  schema: {
    body: userSchema,
    response: {
      200: {
        type: 'object',
        properties: {
          id: { type: 'number' },
          email: { type: 'string' },
          name: { type: 'string' }
        }
      }
    }
  }
}, async (request, reply) => {
  const user = await userService.create(request.body);
  return user;
});
```

## Giai đoạn 5: Security Layer (Tuần 5-6)

### 5.1 Password Hashing với bcrypt (2-3 ngày)
**Từ khóa tìm hiểu:**
- Password hashing vs encryption
- Salt rounds và security
- Async vs sync hashing
- Timing attacks prevention

**Ví dụ thực hành:**
```javascript
const bcrypt = require('bcrypt');

class PasswordService {
  constructor() {
    this.saltRounds = 12; // Higher = more secure but slower
  }

  async hashPassword(plainPassword) {
    try {
      const salt = await bcrypt.genSalt(this.saltRounds);
      const hash = await bcrypt.hash(plainPassword, salt);
      return hash;
    } catch (error) {
      throw new Error('Password hashing failed');
    }
  }

  async comparePassword(plainPassword, hashedPassword) {
    try {
      return await bcrypt.compare(plainPassword, hashedPassword);
    } catch (error) {
      throw new Error('Password comparison failed');
    }
  }
}

// Integration with User Service
class UserService {
  constructor(db) {
    this.db = db;
    this.passwordService = new PasswordService();
  }

  async register(userData) {
    const hashedPassword = await this.passwordService.hashPassword(userData.password);
    const user = await this.create({
      ...userData,
      password: hashedPassword
    });
    
    // Remove password from response
    delete user.password;
    return user;
  }

  async authenticate(email, password) {
    const user = await this.findByEmail(email);
    if (!user) {
      throw new Error('User not found');
    }

    const isValidPassword = await this.passwordService.comparePassword(password, user.password);
    if (!isValidPassword) {
      throw new Error('Invalid password');
    }

    delete user.password;
    return user;
  }
}
```

### 5.2 JWT Authentication (3-4 ngày)
**Từ khóa tìm hiểu:**
- JWT structure (header, payload, signature)
- Access vs Refresh tokens
- Token expiration strategies
- Stateless authentication
- Security best practices

**Ví dụ từng bước:**

**Bước 1 - JWT Service:**
```javascript
const jwt = require('jsonwebtoken');

class JWTService {
  constructor() {
    this.accessTokenSecret = process.env.JWT_ACCESS_SECRET;
    this.refreshTokenSecret = process.env.JWT_REFRESH_SECRET;
    this.accessTokenExpiry = '15m';
    this.refreshTokenExpiry = '7d';
  }

  generateAccessToken(payload) {
    return jwt.sign(payload, this.accessTokenSecret, {
      expiresIn: this.accessTokenExpiry,
      issuer: 'myapp',
      audience: 'myapp-users'
    });
  }

  generateRefreshToken(payload) {
    return jwt.sign(payload, this.refreshTokenSecret, {
      expiresIn: this.refreshTokenExpiry,
      issuer: 'myapp',
      audience: 'myapp-users'
    });
  }

  verifyAccessToken(token) {
    try {
      return jwt.verify(token, this.accessTokenSecret);
    } catch (error) {
      throw new Error('Invalid access token');
    }
  }

  verifyRefreshToken(token) {
    try {
      return jwt.verify(token, this.refreshTokenSecret);
    } catch (error) {
      throw new Error('Invalid refresh token');
    }
  }

  generateTokenPair(user) {
    const payload = {
      id: user.id,
      email: user.email,
      name: user.name
    };

    return {
      accessToken: this.generateAccessToken(payload),
      refreshToken: this.generateRefreshToken({ id: user.id })
    };
  }
}
```

**Bước 2 - Authentication Plugin:**
```javascript
async function authPlugin(fastify, options) {
  const jwtService = new JWTService();

  fastify.decorate('authenticate', async (request, reply) => {
    try {
      const authHeader = request.headers.authorization;
      if (!authHeader || !authHeader.startsWith('Bearer ')) {
        return reply.status(401).send({ error: 'Missing or invalid authorization header' });
      }

      const token = authHeader.substring(7);
      const decoded = jwtService.verifyAccessToken(token);
      
      // Add user info to request
      request.user = decoded;
    } catch (error) {
      return reply.status(401).send({ error: 'Invalid token' });
    }
  });

  fastify.decorate('jwtService', jwtService);
}

fastify.register(authPlugin);
```

**Bước 3 - Auth Routes:**
```javascript
// Login route
fastify.post('/auth/login', {
  schema: {
    body: {
      type: 'object',
      properties: {
        email: { type: 'string', format: 'email' },
        password: { type: 'string', minLength: 1 }
      },
      required: ['email', 'password']
    }
  }
}, async (request, reply) => {
  const { email, password } = request.body;

  try {
    const user = await userService.authenticate(email, password);
    const tokens = fastify.jwtService.generateTokenPair(user);
    
    return {
      user,
      ...tokens
    };
  } catch (error) {
    return reply.status(401).send({ error: error.message });
  }
});

// Protected route
fastify.get('/users/profile', {
  preHandler: fastify.authenticate
}, async (request, reply) => {
  const userId = request.user.id;
  const user = await userService.findById(userId);
  return user;
});

// Refresh token route
fastify.post('/auth/refresh', async (request, reply) => {
  const { refreshToken } = request.body;
  
  try {
    const decoded = fastify.jwtService.verifyRefreshToken(refreshToken);
    const user = await userService.findById(decoded.id);
    
    if (!user) {
      return reply.status(401).send({ error: 'User not found' });
    }

    const tokens = fastify.jwtService.generateTokenPair(user);
    return tokens;
  } catch (error) {
    return reply.status(401).send({ error: 'Invalid refresh token' });
  }
});
```

## Giai đoạn 6: CORS & Security (Tuần 6)

### 6.1 CORS Configuration (1-2 ngày)
**Từ khóa tìm hiểu:**
- Cross-Origin Resource Sharing
- Preflight requests
- Credentials handling
- Security implications

**Ví dụ thực hành:**
```javascript
await fastify.register(require('@fastify/cors'), {
  origin: (origin, callback) => {
    const hostname = new URL(origin).hostname;
    
    // Allow localhost in development
    if (hostname === 'localhost' && process.env.NODE_ENV === 'development') {
      callback(null, true);
      return;
    }

    // Allow specific domains in production
    const allowedDomains = process.env.ALLOWED_ORIGINS?.split(',') || [];
    if (allowedDomains.includes(hostname)) {
      callback(null, true);
      return;
    }

    callback(new Error('Not allowed by CORS'), false);
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization']
});
```

## Giai đoạn 7: Plugin System & Architecture (Tuần 7)

### 7.1 Fastify Plugins (3-4 ngày)
**Từ khóa tìm hiểu:**
- Plugin encapsulation
- Plugin dependencies
- Context isolation
- Plugin lifecycle

**Ví dụ thực hành:**
```javascript
// Database plugin
async function databasePlugin(fastify, options) {
  const db = require('knex')(options.knexConfig);
  
  fastify.decorate('db', db);
  
  fastify.addHook('onClose', async (instance) => {
    await instance.db.destroy();
  });
}

// Services plugin
async function servicesPlugin(fastify, options) {
  const userService = new UserService(fastify.db);
  const jwtService = new JWTService();
  
  fastify.decorate('userService', userService);
  fastify.decorate('jwtService', jwtService);
}

// Routes plugin
async function userRoutesPlugin(fastify, options) {
  const userService = fastify.userService;
  
  fastify.post('/users', {
    preHandler: [fastify.authenticate, fastify.validate(userSchema)]
  }, async (request, reply) => {
    const user = await userService.create(request.validatedBody);
    return user;
  });
  
  fastify.get('/users/:id', {
    preHandler: fastify.authenticate
  }, async (request, reply) => {
    const user = await userService.findById(request.params.id);
    if (!user) {
      return reply.status(404).send({ error: 'User not found' });
    }
    return user;
  });
}

// Main server setup
async function buildServer() {
  const fastify = require('fastify')({ logger: true });

  // Register plugins in order
  await fastify.register(databasePlugin, { knexConfig });
  await fastify.register(servicesPlugin);
  await fastify.register(authPlugin);
  await fastify.register(validationPlugin);
  await fastify.register(userRoutesPlugin, { prefix: '/api' });

  return fastify;
}
```

## Giai đoạn 8: Complete Integration Project (Tuần 8-9)

### 8.1 Full-Stack Application
**Project: Task Management API**

**Features to implement:**
- User registration/authentication
- CRUD operations for tasks
- Task categories and priorities
- File upload for task attachments
- Search and filtering
- Pagination
- Role-based access control
- API documentation

**Project structure:**
```
project/
├── src/
│   ├── config/
│   │   ├── database.js
│   │   └── config.js
│   ├── migrations/
│   ├── models/
│   │   ├── User.js
│   │   └── Task.js
│   ├── services/
│   │   ├── UserService.js
│   │   ├── TaskService.js
│   │   ├── AuthService.js
│   │   └── FileService.js
│   ├── plugins/
│   │   ├── auth.js
│   │   ├── validation.js
│   │   └── database.js
│   ├── routes/
│   │   ├── auth.js
│   │   ├── users.js
│   │   └── tasks.js
│   ├── schemas/
│   │   ├── userSchemas.js
│   │   └── taskSchemas.js
│   └── server.js
├── tests/
├── .env
├── knexfile.js
└── package.json
```

## Lịch trình học tập chi tiết:

### Tuần 1: Foundation
- **Ngày 1-2**: Node.js fundamentals
- **Ngày 3-5**: Fastify basics (server, routes, hooks)
- **Ngày 6-7**: Environment setup và dotenv

### Tuần 2: Database
- **Ngày 1-3**: Knex.js setup, migrations, basic queries
- **Ngày 4-5**: Advanced Knex (relations, transactions)
- **Ngày 6-7**: MSSQL integration

### Tuần 3: Validation
- **Ngày 1-3**: Joi validation (schemas, custom rules)
- **Ngày 4-5**: AJV validation
- **Ngày 6-7**: Integration với Fastify

### Tuần 4: Security
- **Ngày 1-2**: bcrypt password hashing
- **Ngày 3-5**: JWT authentication
- **Ngày 6-7**: CORS và security headers

### Tuần 5: Architecture
- **Ngày 1-3**: Plugin system
- **Ngày 4-5**: Error handling và logging
- **Ngày 6-7**: Testing setup

### Tuần 6-7: Complete Project
- **Tuần 6**: Build core features
- **Tuần 7**: Advanced features và optimization

## Resources để tham khảo:

### Documentation:
- [Fastify Official Docs](https://www.fastify.io/docs/)
- [Knex.js Documentation](https://knexjs.org/)
- [Joi API Reference](https://joi.dev/api/)
- [AJV Documentation](https://ajv.js.org/)

### Best Practices:
- Error handling patterns
- Security headers
- Rate limiting
- Input sanitization
- SQL injection prevention
- API versioning
- Monitoring và logging

Bạn muốn tôi chi tiết hóa phần nào trước? Hoặc bạn muốn bắt đầu với ví dụ cụ thể nào?