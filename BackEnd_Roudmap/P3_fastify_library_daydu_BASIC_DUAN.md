# Fastify Backend Essentials - Các phần cơ bản cho dự án thực tế

## 1. FASTIFY CORE - Những phần quan trọng nhất

### 1.1 Server Instance & Basic Setup
**Dùng để**: Khởi tạo server, config cơ bản
```javascript
const fastify = require('fastify')({
  logger: true, // Bắt buộc cho production - log mọi request/response
  trustProxy: true, // Khi deploy sau reverse proxy (nginx, cloudflare)
  bodyLimit: 1048576 * 10, // 10MB - giới hạn kích thước request body
});

// Graceful shutdown - quan trọng cho production
const gracefulShutdown = async () => {
  try {
    await fastify.close();
    console.log('Server closed gracefully');
    process.exit(0);
  } catch (err) {
    console.error('Error during shutdown:', err);
    process.exit(1);
  }
};

process.on('SIGTERM', gracefulShutdown);
process.on('SIGINT', gracefulShutdown);
```

### 1.2 Routes - HTTP Methods
**Dùng để**: Định nghĩa các endpoint API
```javascript
// GET - Lấy dữ liệu
fastify.get('/api/users', async (request, reply) => {
  const { page = 1, limit = 10 } = request.query;
  const users = await userService.getUsers(page, limit);
  return users;
});

// GET với params - Lấy 1 record cụ thể
fastify.get('/api/users/:id', async (request, reply) => {
  const { id } = request.params;
  const user = await userService.getUserById(id);
  
  if (!user) {
    return reply.code(404).send({ error: 'User not found' });
  }
  
  return user;
});

// POST - Tạo mới
fastify.post('/api/users', async (request, reply) => {
  const userData = request.body;
  const newUser = await userService.createUser(userData);
  
  reply.code(201).send(newUser);
});

// PUT - Cập nhật toàn bộ
fastify.put('/api/users/:id', async (request, reply) => {
  const { id } = request.params;
  const updateData = request.body;
  
  const updatedUser = await userService.updateUser(id, updateData);
  return updatedUser;
});

// PATCH - Cập nhật một phần
fastify.patch('/api/users/:id', async (request, reply) => {
  const { id } = request.params;
  const partialData = request.body;
  
  const updatedUser = await userService.partialUpdate(id, partialData);
  return updatedUser;
});

// DELETE - Xóa
fastify.delete('/api/users/:id', async (request, reply) => {
  const { id } = request.params;
  await userService.deleteUser(id);
  
  reply.code(204).send(); // 204 No Content
});
```

### 1.3 Hooks System - Middleware của Fastify
**Dùng để**: Xử lý logic chung cho nhiều routes (auth, logging, validation)

```javascript
// 1. onRequest - Chạy đầu tiên khi có request
fastify.addHook('onRequest', async (request, reply) => {
  // Log tất cả requests
  console.log(`${request.method} ${request.url} - IP: ${request.ip}`);
  
  // Rate limiting check ở đây
  const clientIP = request.ip;
  if (await rateLimiter.isBlocked(clientIP)) {
    reply.code(429).send({ error: 'Too many requests' });
  }
});

// 2. preHandler - Chạy trước handler chính
// Dùng cho: Authentication, Authorization, Validation
fastify.addHook('preHandler', async (request, reply) => {
  // Bỏ qua auth cho public routes
  const publicRoutes = ['/api/auth/login', '/api/auth/register', '/api/health'];
  if (publicRoutes.includes(request.url)) {
    return;
  }

  // Check JWT token
  const token = request.headers.authorization?.replace('Bearer ', '');
  if (!token) {
    reply.code(401).send({ error: 'Token required' });
    return;
  }

  try {
    const user = await jwtService.verifyToken(token);
    request.user = user; // Gắn user info vào request
  } catch (error) {
    reply.code(401).send({ error: 'Invalid token' });
  }
});

// 3. onSend - Xử lý response trước khi gửi về client
fastify.addHook('onSend', async (request, reply, payload) => {
  // Add security headers
  reply.header('X-Frame-Options', 'DENY');
  reply.header('X-Content-Type-Options', 'nosniff');
  
  // Remove sensitive data
  if (typeof payload === 'string') {
    const data = JSON.parse(payload);
    if (data.password) {
      delete data.password;
      return JSON.stringify(data);
    }
  }
  
  return payload;
});

// 4. onError - Xử lý lỗi
fastify.setErrorHandler(async (error, request, reply) => {
  // Log error
  console.error('Error:', error);
  
  // Handle different types of errors
  if (error.validation) {
    reply.code(400).send({
      error: 'Validation Error',
      details: error.validation
    });
  } else if (error.statusCode) {
    reply.code(error.statusCode).send({ error: error.message });
  } else {
    // Unknown error - don't expose details in production
    reply.code(500).send({ 
      error: process.env.NODE_ENV === 'production' 
        ? 'Internal Server Error' 
        : error.message 
    });
  }
});
```

### 1.4 Plugin System - Tách biệt logic
**Dùng để**: Tổ chức code, tái sử dụng, test dễ hơn

```javascript
// plugins/database.js - Database connection
async function databasePlugin(fastify, options) {
  const knex = require('knex')(options.knexConfig);
  
  // Test connection
  try {
    await knex.raw('SELECT 1');
    fastify.log.info('Database connected successfully');
  } catch (error) {
    fastify.log.error('Database connection failed:', error);
    throw error;
  }
  
  fastify.decorate('db', knex);
  
  // Cleanup on server close
  fastify.addHook('onClose', async () => {
    await knex.destroy();
  });
}

// plugins/auth.js - Authentication middleware
async function authPlugin(fastify, options) {
  const jwtService = require('../services/JWTService');
  
  fastify.decorate('authenticate', async (request, reply) => {
    const token = request.headers.authorization?.replace('Bearer ', '');
    
    if (!token) {
      return reply.code(401).send({ error: 'Authentication required' });
    }
    
    try {
      const user = await jwtService.verifyToken(token);
      request.user = user;
    } catch (error) {
      return reply.code(401).send({ error: 'Invalid token' });
    }
  });
  
  fastify.decorate('authorize', (roles = []) => {
    return async (request, reply) => {
      if (!request.user) {
        return reply.code(401).send({ error: 'Authentication required' });
      }
      
      if (roles.length && !roles.includes(request.user.role)) {
        return reply.code(403).send({ error: 'Insufficient permissions' });
      }
    };
  });
}

// Đăng ký plugins
fastify.register(databasePlugin, { knexConfig });
fastify.register(authPlugin);
```

### 1.5 Schema Validation - Fastify built-in
**Dùng để**: Validate input/output, auto-generate docs
```javascript
const userSchema = {
  body: {
    type: 'object',
    required: ['email', 'password', 'name'],
    properties: {
      email: { type: 'string', format: 'email' },
      password: { type: 'string', minLength: 6 },
      name: { type: 'string', minLength: 2, maxLength: 50 }
    }
  },
  response: {
    201: {
      type: 'object',
      properties: {
        id: { type: 'integer' },
        email: { type: 'string' },
        name: { type: 'string' },
        createdAt: { type: 'string' }
      }
    },
    400: {
      type: 'object',
      properties: {
        error: { type: 'string' }
      }
    }
  }
};

fastify.post('/api/users', {
  schema: userSchema,
  preHandler: [fastify.authenticate]
}, async (request, reply) => {
  const user = await userService.createUser(request.body);
  reply.code(201).send(user);
});
```

## 2. DOTENV - Environment Variables

### Cơ bản cần biết:
**Dùng để**: Quản lý config khác nhau giữa dev/staging/production
```javascript
// .env file
NODE_ENV=development
PORT=3000
DB_HOST=localhost
DB_USER=myuser
DB_PASS=mypassword
DB_NAME=mydb
JWT_SECRET=your-super-secret-key
JWT_EXPIRES_IN=24h
BCRYPT_ROUNDS=12

// config/index.js
require('dotenv').config();

const config = {
  env: process.env.NODE_ENV || 'development',
  port: parseInt(process.env.PORT) || 3000,
  
  database: {
    host: process.env.DB_HOST,
    user: process.env.DB_USER,
    password: process.env.DB_PASS,
    database: process.env.DB_NAME,
  },
  
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN || '24h'
  },
  
  bcrypt: {
    rounds: parseInt(process.env.BCRYPT_ROUNDS) || 12
  }
};

module.exports = config;
```

## 3. KNEX.JS - Database Query Builder

### Những phần cần biết cho dự án thực tế:

```javascript
// config/database.js
const knex = require('knex');
const config = require('./config');

const db = knex({
  client: 'mssql', // hoặc 'mysql', 'postgresql'
  connection: {
    server: config.database.host,
    database: config.database.database,
    user: config.database.user,
    password: config.database.password,
    options: {
      encrypt: true, // cho SQL Server
      trustServerCertificate: true
    }
  },
  pool: {
    min: 0,
    max: 10,
    acquireTimeoutMillis: 30000,
    createTimeoutMillis: 30000,
    destroyTimeoutMillis: 5000,
    idleTimeoutMillis: 30000,
    reapIntervalMillis: 1000,
  }
});

// Basic CRUD operations
class BaseRepository {
  constructor(tableName) {
    this.table = tableName;
    this.db = db;
  }

  // CREATE
  async create(data) {
    const [id] = await this.db(this.table).insert(data).returning('id');
    return this.findById(id);
  }

  // READ - Find by ID
  async findById(id) {
    return await this.db(this.table).where({ id }).first();
  }

  // READ - List with pagination
  async findAll(page = 1, limit = 10, filters = {}) {
    let query = this.db(this.table);
    
    // Apply filters
    Object.keys(filters).forEach(key => {
      if (filters[key] !== undefined) {
        query = query.where(key, filters[key]);
      }
    });
    
    const total = await query.clone().count('* as count').first();
    
    const data = await query
      .limit(limit)
      .offset((page - 1) * limit)
      .orderBy('created_at', 'desc');
    
    return {
      data,
      pagination: {
        page,
        limit,
        total: total.count,
        pages: Math.ceil(total.count / limit)
      }
    };
  }

  // UPDATE
  async update(id, data) {
    await this.db(this.table).where({ id }).update({
      ...data,
      updated_at: this.db.fn.now()
    });
    return this.findById(id);
  }

  // DELETE
  async delete(id) {
    return await this.db(this.table).where({ id }).del();
  }

  // Relationships
  async findWithRelations(id, relations = []) {
    let query = this.db(this.table).where(`${this.table}.id`, id);
    
    relations.forEach(relation => {
      query = query.leftJoin(
        relation.table,
        `${this.table}.${relation.foreignKey}`,
        `${relation.table}.id`
      );
    });
    
    return await query.first();
  }
}

// Usage example
class UserRepository extends BaseRepository {
  constructor() {
    super('users');
  }

  async findByEmail(email) {
    return await this.db(this.table).where({ email }).first();
  }

  async findUsersWithTasks(userId) {
    return await this.db('users')
      .leftJoin('tasks', 'users.id', 'tasks.user_id')
      .where('users.id', userId)
      .select('users.*', 'tasks.title as task_title');
  }
}
```

## 4. JOI - Validation Schema

### Cơ bản cần dùng:
**Dùng để**: Validate dữ liệu đầu vào phức tạp hơn built-in của Fastify

```javascript
const Joi = require('joi');

// User schemas
const userSchemas = {
  register: Joi.object({
    email: Joi.string().email().required().messages({
      'string.email': 'Email không đúng định dạng',
      'any.required': 'Email là bắt buộc'
    }),
    password: Joi.string()
      .min(6)
      .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
      .required()
      .messages({
        'string.min': 'Mật khẩu tối thiểu 6 ký tự',
        'string.pattern.base': 'Mật khẩu phải có chữ hoa, thường và số'
      }),
    name: Joi.string().min(2).max(50).required(),
    phone: Joi.string().pattern(/^[0-9]{10,11}$/).optional()
  }),

  login: Joi.object({
    email: Joi.string().email().required(),
    password: Joi.string().required()
  }),

  update: Joi.object({
    name: Joi.string().min(2).max(50),
    phone: Joi.string().pattern(/^[0-9]{10,11}$/)
  }).min(1) // Ít nhất 1 field
};

// Task schemas
const taskSchemas = {
  create: Joi.object({
    title: Joi.string().min(1).max(200).required(),
    description: Joi.string().max(1000).optional(),
    priority: Joi.string().valid('low', 'medium', 'high').default('medium'),
    dueDate: Joi.date().greater('now').optional(),
    categoryId: Joi.number().integer().positive().required()
  }),

  update: Joi.object({
    title: Joi.string().min(1).max(200),
    description: Joi.string().max(1000),
    priority: Joi.string().valid('low', 'medium', 'high'),
    status: Joi.string().valid('pending', 'in_progress', 'completed'),
    dueDate: Joi.date().greater('now').allow(null)
  }).min(1)
};

// Validation middleware
const validate = (schema) => {
  return async (request, reply) => {
    try {
      const { error, value } = schema.validate(request.body, { 
        abortEarly: false,
        stripUnknown: true 
      });
      
      if (error) {
        return reply.code(400).send({
          error: 'Validation failed',
          details: error.details.map(d => ({
            field: d.path.join('.'),
            message: d.message
          }))
        });
      }
      
      request.validatedBody = value;
    } catch (err) {
      return reply.code(400).send({ error: err.message });
    }
  };
};

// Usage
fastify.post('/api/users', {
  preHandler: [validate(userSchemas.register)]
}, async (request, reply) => {
  const userData = request.validatedBody;
  const user = await userService.createUser(userData);
  reply.code(201).send(user);
});
```

## 5. BCRYPT - Password Security

### Cần biết:
**Dùng để**: Hash password an toàn, so sánh password

```javascript
const bcrypt = require('bcrypt');
const config = require('../config');

class PasswordService {
  static async hash(plainPassword) {
    try {
      const salt = await bcrypt.genSalt(config.bcrypt.rounds);
      return await bcrypt.hash(plainPassword, salt);
    } catch (error) {
      throw new Error('Failed to hash password');
    }
  }

  static async compare(plainPassword, hashedPassword) {
    try {
      return await bcrypt.compare(plainPassword, hashedPassword);
    } catch (error) {
      throw new Error('Failed to compare password');
    }
  }
}

// Usage trong service
class AuthService {
  static async register(userData) {
    // Hash password trước khi lưu
    const hashedPassword = await PasswordService.hash(userData.password);
    
    const user = await userRepository.create({
      ...userData,
      password: hashedPassword
    });
    
    // Remove password from response
    delete user.password;
    return user;
  }

  static async login(email, password) {
    const user = await userRepository.findByEmail(email);
    if (!user) {
      throw new Error('Invalid credentials');
    }

    const isValidPassword = await PasswordService.compare(password, user.password);
    if (!isValidPassword) {
      throw new Error('Invalid credentials');
    }

    delete user.password;
    return user;
  }
}
```

## 6. JSONWEBTOKEN - Authentication

### Cơ bản cần biết:
**Dùng để**: Tạo và verify JWT tokens cho authentication

```javascript
const jwt = require('jsonwebtoken');
const config = require('../config');

class JWTService {
  static generateToken(payload) {
    return jwt.sign(
      payload,
      config.jwt.secret,
      {
        expiresIn: config.jwt.expiresIn,
        issuer: 'your-app-name',
        audience: 'your-app-users'
      }
    );
  }

  static verifyToken(token) {
    try {
      return jwt.verify(token, config.jwt.secret);
    } catch (error) {
      if (error.name === 'TokenExpiredError') {
        throw new Error('Token expired');
      }
      if (error.name === 'JsonWebTokenError') {
        throw new Error('Invalid token');
      }
      throw new Error('Token verification failed');
    }
  }

  static generateRefreshToken(userId) {
    return jwt.sign(
      { userId, type: 'refresh' },
      config.jwt.secret,
      { expiresIn: '7d' }
    );
  }
}

// Auth routes
fastify.post('/api/auth/login', {
  schema: {
    body: {
      type: 'object',
      required: ['email', 'password'],
      properties: {
        email: { type: 'string', format: 'email' },
        password: { type: 'string' }
      }
    }
  }
}, async (request, reply) => {
  const { email, password } = request.body;

  try {
    const user = await AuthService.login(email, password);
    const token = JWTService.generateToken({ 
      id: user.id, 
      email: user.email 
    });

    return {
      user,
      token,
      expiresIn: config.jwt.expiresIn
    };
  } catch (error) {
    reply.code(401).send({ error: error.message });
  }
});
```

## 7. @FASTIFY/CORS - Cross-Origin Resource Sharing

### Cần biết:
**Dùng để**: Cho phép frontend call API từ domain khác

```javascript
const cors = require('@fastify/cors');

// Basic setup
fastify.register(cors, {
  origin: (origin, callback) => {
    // Cho phép tất cả origins trong development
    if (process.env.NODE_ENV === 'development') {
      callback(null, true);
      return;
    }

    // Production: chỉ cho phép domains cụ thể
    const allowedOrigins = [
      'https://yourdomain.com',
      'https://app.yourdomain.com'
    ];

    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'), false);
    }
  },
  credentials: true, // Cho phép cookies
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS']
});
```

## 8. Complete Example - Tích hợp tất cả

```javascript
// server.js - Main application file
const fastify = require('fastify')({ logger: true });
const config = require('./config');

async function buildApp() {
  // 1. Register CORS
  await fastify.register(require('@fastify/cors'), {
    origin: true,
    credentials: true
  });

  // 2. Register database plugin
  await fastify.register(require('./plugins/database'), {
    knexConfig: config.database
  });

  // 3. Register auth plugin
  await fastify.register(require('./plugins/auth'));

  // 4. Register validation plugin
  await fastify.register(require('./plugins/validation'));

  // 5. Register routes
  await fastify.register(require('./routes/auth'), { prefix: '/api/auth' });
  await fastify.register(require('./routes/users'), { prefix: '/api/users' });
  await fastify.register(require('./routes/tasks'), { prefix: '/api/tasks' });

  // 6. Health check
  fastify.get('/health', async (request, reply) => {
    return { status: 'OK', timestamp: new Date().toISOString() };
  });

  return fastify;
}

// Start server
async function start() {
  try {
    const app = await buildApp();
    await app.listen({ 
      port: config.port, 
      host: '0.0.0.0' 
    });
    console.log(`Server running on port ${config.port}`);
  } catch (err) {
    console.error('Error starting server:', err);
    process.exit(1);
  }
}

if (require.main === module) {
  start();
}

module.exports = { buildApp };
```

## Tóm tắt - Những gì QUAN TRỌNG NHẤT cho dự án:

### Fastify:
1. **Server setup** với logger và error handling
2. **Routes** với đầy đủ HTTP methods
3. **Hooks** cho middleware (auth, validation, logging)
4. **Plugin system** để tổ chức code

### Thư viện:
1. **dotenv**: Config môi trường
2. **Knex**: Database operations (CRUD + migrations)
3. **Joi**: Validation phức tạp
4. **bcrypt**: Hash password
5. **jsonwebtoken**: Authentication
6. **@fastify/cors**: Frontend integration
