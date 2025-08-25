# 🚀 Roadmap thực hiện dự án Fastify Todo Backend

## Phase 1: Setup Project Foundation (1-2 ngày)

### Step 1: Khởi tạo project và cài đặt dependencies
```bash
# Tạo thư mục dự án
mkdir todo-backend
cd todo-backend

# Khởi tạo npm project
npm init -y

# Cài đặt dependencies chính
npm install fastify@^5.5.0 @fastify/cors@^11.1.0 fastify-plugin@^5.0.1
npm install knex@^3.1.0 mssql@^11.0.1
npm install bcrypt@^6.0.0 jsonwebtoken@^9.0.2
npm install joi@^18.0.0 ajv@^8.17.1 ajv-formats@^3.0.1
npm install dotenv@^17.2.1

# Cài đặt dev dependencies
npm install nodemon@^3.0.0 --save-dev
```

### Step 2: Cấu hình package.json scripts
```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "migrate": "knex migrate:latest",
    "migrate:rollback": "knex migrate:rollback",
    "migrate:make": "knex migrate:make"
  },
  "type": "commonjs"
}
```

### Step 3: Tạo cấu trúc thư mục
```bash
mkdir src
mkdir src/config
mkdir src/plugins  
mkdir src/routers
mkdir src/schemas
mkdir src/services
mkdir src/utils
mkdir migrations
```

### Step 4: Setup file môi trường
```bash
# Tạo file .env
touch .env
```

```env
# .env
NODE_ENV=development
PORT=3000
JWT_SECRET=your-super-secret-jwt-key-here
DB_HOST=localhost
DB_PORT=1433
DB_NAME=todo_db
DB_USER=sa
DB_PASSWORD=your_password
```

## Phase 2: Database Setup (1 ngày)

### Step 5: Cấu hình Knex.js
```javascript
// knexfile.js
require('dotenv').config()

module.exports = {
  development: {
    client: 'mssql',
    connection: {
      server: process.env.DB_HOST,
      port: parseInt(process.env.DB_PORT),
      database: process.env.DB_NAME,
      user: process.env.DB_USER,
      password: process.env.DB_PASSWORD,
      options: {
        encrypt: true,
        trustServerCertificate: true
      }
    },
    migrations: {
      directory: './migrations'
    }
  },
  
  production: {
    client: 'mssql',
    connection: {
      server: process.env.DB_HOST,
      port: parseInt(process.env.DB_PORT),
      database: process.env.DB_NAME,
      user: process.env.DB_USER,
      password: process.env.DB_PASSWORD,
      options: {
        encrypt: true,
        trustServerCertificate: false
      }
    },
    migrations: {
      directory: './migrations'
    }
  }
}
```

### Step 6: Tạo database connection
```javascript
// src/config/db.js
const knex = require('knex')
const config = require('../../knexfile')[process.env.NODE_ENV || 'development']

const db = knex(config)

// Test connection
db.raw('SELECT 1+1 as result')
  .then(() => {
    console.log('✅ Database connected successfully')
  })
  .catch(err => {
    console.log('❌ Database connection failed:', err.message)
    process.exit(1)
  })

module.exports = db
```

### Step 7: Tạo migrations cho database schema
```bash
# Tạo migration cho bảng users
npm run migrate:make create_users_table

# Tạo migration cho bảng todos  
npm run migrate:make create_todos_table
```

```javascript
// migrations/001_create_users_table.js
exports.up = function(knex) {
  return knex.schema.createTable('users', function(table) {
    table.increments('id').primary()
    table.string('username', 50).notNullable().unique()
    table.string('email', 100).notNullable().unique()
    table.string('password', 255).notNullable()
    table.timestamp('created_at').defaultTo(knex.fn.now())
    table.timestamp('updated_at').defaultTo(knex.fn.now())
    
    // Indexes
    table.index(['email'])
    table.index(['username'])
  })
}

exports.down = function(knex) {
  return knex.schema.dropTable('users')
}
```

```javascript
// migrations/002_create_todos_table.js
exports.up = function(knex) {
  return knex.schema.createTable('todos', function(table) {
    table.increments('id').primary()
    table.integer('user_id').unsigned().notNullable()
    table.string('title', 200).notNullable()
    table.text('description').nullable()
    table.boolean('completed').defaultTo(false)
    table.enum('priority', ['low', 'medium', 'high']).defaultTo('medium')
    table.timestamp('created_at').defaultTo(knex.fn.now())
    table.timestamp('updated_at').defaultTo(knex.fn.now())
    
    // Foreign key constraint
    table.foreign('user_id').references('id').inTable('users').onDelete('CASCADE')
    
    // Indexes
    table.index(['user_id'])
    table.index(['completed'])
    table.index(['created_at'])
  })
}

exports.down = function(knex) {
  return knex.schema.dropTable('todos')
}
```

```bash
# Chạy migrations
npm run migrate
```

## Phase 3: Core Utilities & Plugins (1 ngày)

### Step 8: Tạo JWT utilities
```javascript
// src/utils/jwt.js
const jwt = require('jsonwebtoken')

const JWT_SECRET = process.env.JWT_SECRET || 'fallback-secret-key'
const JWT_EXPIRES_IN = process.env.JWT_EXPIRES_IN || '24h'

const jwtUtils = {
  generateToken: (payload) => {
    return jwt.sign(payload, JWT_SECRET, { 
      expiresIn: JWT_EXPIRES_IN,
      issuer: 'todo-app'
    })
  },

  verifyToken: (token) => {
    try {
      return jwt.verify(token, JWT_SECRET, { issuer: 'todo-app' })
    } catch (error) {
      throw new Error('Invalid or expired token')
    }
  },

  decodeToken: (token) => {
    return jwt.decode(token)
  }
}

module.exports = jwtUtils
```

### Step 9: Tạo Authentication Plugin
```javascript
// src/plugins/authPlugins.js
const fastifyPlugin = require('fastify-plugin')
const jwtUtils = require('../utils/jwt')

async function authPlugin(fastify, options) {
  
  // Decorator để authenticate user
  fastify.decorate('authenticate', async function (request, reply) {
    try {
      const authHeader = request.headers.authorization
      
      if (!authHeader || !authHeader.startsWith('Bearer ')) {
        return reply.code(401).send({ 
          error: 'Authorization header required',
          code: 'MISSING_TOKEN' 
        })
      }

      const token = authHeader.slice(7) // Remove 'Bearer ' prefix
      const decoded = jwtUtils.verifyToken(token)
      
      // Gắn user info vào request
      request.user = {
        id: decoded.id,
        email: decoded.email
      }
      
    } catch (error) {
      return reply.code(401).send({ 
        error: 'Invalid or expired token',
        code: 'INVALID_TOKEN'
      })
    }
  })

  // Optional: Check if user is admin
  fastify.decorate('requireAdmin', async function (request, reply) {
    if (!request.user.isAdmin) {
      return reply.code(403).send({ 
        error: 'Admin access required',
        code: 'INSUFFICIENT_PERMISSIONS' 
      })
    }
  })
}

module.exports = fastifyPlugin(authPlugin, {
  name: 'auth-plugin'
})
```

### Step 10: Tạo Validation Plugin
```javascript
// src/plugins/validatePlugins.js
const fastifyPlugin = require('fastify-plugin')

async function validatePlugin(fastify, options) {
  
  fastify.decorate('validateJoi', (schema) => {
    return async (request, reply) => {
      try {
        const { error, value } = schema.validate(request.body, {
          abortEarly: false, // Trả về tất cả lỗi
          stripUnknown: true // Loại bỏ fields không có trong schema
        })
        
        if (error) {
          return reply.code(400).send({
            error: 'Validation failed',
            code: 'VALIDATION_ERROR',
            details: error.details.map(detail => ({
              field: detail.path.join('.'),
              message: detail.message
            }))
          })
        }
        
        request.validatedBody = value
      } catch (err) {
        fastify.log.error(err)
        return reply.code(500).send({ 
          error: 'Validation processing error',
          code: 'VALIDATION_PROCESSING_ERROR'
        })
      }
    }
  })

  // Custom validation cho pagination
  fastify.decorate('validatePagination', async (request, reply) => {
    const { page = 1, limit = 10 } = request.query
    
    const pageNum = parseInt(page)
    const limitNum = parseInt(limit)
    
    if (isNaN(pageNum) || pageNum < 1) {
      return reply.code(400).send({
        error: 'Page must be a positive integer',
        code: 'INVALID_PAGE'
      })
    }
    
    if (isNaN(limitNum) || limitNum < 1 || limitNum > 100) {
      return reply.code(400).send({
        error: 'Limit must be between 1 and 100',
        code: 'INVALID_LIMIT'
      })
    }
    
    request.pagination = {
      page: pageNum,
      limit: limitNum,
      offset: (pageNum - 1) * limitNum
    }
  })
}

module.exports = fastifyPlugin(validatePlugin, {
  name: 'validate-plugin'
})
```

## Phase 4: Schemas Definition (1 ngày)

### Step 11: Tạo Auth Schemas
```javascript
// src/schemas/authSchemas.js
const Joi = require('joi')

// Joi validation schemas
const registerSchema = Joi.object({
  username: Joi.string()
    .min(3)
    .max(30)
    .pattern(/^[a-zA-Z0-9_]+$/)
    .required()
    .messages({
      'string.pattern.base': 'Username can only contain letters, numbers, and underscores'
    }),
  email: Joi.string()
    .email()
    .required(),
  password: Joi.string()
    .min(6)
    .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
    .required()
    .messages({
      'string.pattern.base': 'Password must contain at least one uppercase letter, one lowercase letter, and one number'
    })
})

const loginSchema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().required()
})

const changePasswordSchema = Joi.object({
  currentPassword: Joi.string().required(),
  newPassword: Joi.string()
    .min(6)
    .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
    .required()
})

// Fastify JSON schemas for response validation  
const userResponseSchema = {
  type: 'object',
  properties: {
    id: { type: 'integer' },
    username: { type: 'string' },
    email: { type: 'string', format: 'email' },
    created_at: { type: 'string', format: 'date-time' },
    updated_at: { type: 'string', format: 'date-time' }
  }
}

const loginResponseSchema = {
  type: 'object',
  properties: {
    user: userResponseSchema,
    token: { type: 'string' }
  }
}

const errorSchema = {
  type: 'object',
  properties: {
    error: { type: 'string' },
    code: { type: 'string' },
    details: { 
      type: 'array',
      items: {
        type: 'object',
        properties: {
          field: { type: 'string' },
          message: { type: 'string' }
        }
      }
    }
  }
}

module.exports = {
  registerSchema,
  loginSchema,
  changePasswordSchema,
  userResponseSchema,
  loginResponseSchema,
  errorSchema
}
```

### Step 12: Tạo Todo Schemas
```javascript
// src/schemas/todoSchemas.js
const Joi = require('joi')

const createTodoSchema = Joi.object({
  title: Joi.string().min(1).max(200).required(),
  description: Joi.string().max(1000).optional().allow(''),
  priority: Joi.string().valid('low', 'medium', 'high').default('medium')
})

const updateTodoSchema = Joi.object({
  title: Joi.string().min(1).max(200).optional(),
  description: Joi.string().max(1000).optional().allow(''),
  completed: Joi.boolean().optional(),
  priority: Joi.string().valid('low', 'medium', 'high').optional()
})

// Fastify schemas
const todoResponseSchema = {
  type: 'object',
  properties: {
    id: { type: 'integer' },
    user_id: { type: 'integer' },
    title: { type: 'string' },
    description: { type: ['string', 'null'] },
    completed: { type: 'boolean' },
    priority: { type: 'string', enum: ['low', 'medium', 'high'] },
    created_at: { type: 'string', format: 'date-time' },
    updated_at: { type: 'string', format: 'date-time' }
  }
}

const todoListResponseSchema = {
  type: 'object',
  properties: {
    data: {
      type: 'array',
      items: todoResponseSchema
    },
    pagination: {
      type: 'object',
      properties: {
        page: { type: 'integer' },
        limit: { type: 'integer' },
        total: { type: 'integer' },
        totalPages: { type: 'integer' }
      }
    }
  }
}

module.exports = {
  createTodoSchema,
  updateTodoSchema,
  todoResponseSchema,
  todoListResponseSchema
}
```

## Phase 5: Service Layer (2-3 ngày)

### Step 13: Tạo Auth Service
```javascript
// src/services/authService.js
const bcrypt = require('bcrypt')
const db = require('../config/db')
const jwtUtils = require('../utils/jwt')

const SALT_ROUNDS = 12

const authService = {
  register: async (userData) => {
    const { username, email, password } = userData
    
    // Check if user already exists
    const existingUser = await db('users')
      .where('email', email)
      .orWhere('username', username)
      .first()
    
    if (existingUser) {
      if (existingUser.email === email) {
        throw new Error('Email already registered')
      }
      if (existingUser.username === username) {
        throw new Error('Username already taken')
      }
    }

    // Hash password
    const hashedPassword = await bcrypt.hash(password, SALT_ROUNDS)

    // Create user with transaction
    const trx = await db.transaction()
    
    try {
      const [newUser] = await trx('users').insert({
        username,
        email,
        password: hashedPassword,
        created_at: new Date(),
        updated_at: new Date()
      }).returning(['id', 'username', 'email', 'created_at', 'updated_at'])

      await trx.commit()
      
      return newUser

    } catch (error) {
      await trx.rollback()
      throw new Error('Failed to create user')
    }
  },

  login: async (credentials) => {
    const { email, password } = credentials
    
    const user = await db('users').where({ email }).first()
    
    if (!user) {
      throw new Error('Invalid email or password')
    }

    const isValidPassword = await bcrypt.compare(password, user.password)
    
    if (!isValidPassword) {
      throw new Error('Invalid email or password')
    }

    // Generate token
    const token = jwtUtils.generateToken({
      id: user.id,
      email: user.email,
      username: user.username
    })

    // Return user without password
    const { password: _, ...userWithoutPassword } = user
    
    return {
      user: userWithoutPassword,
      token
    }
  },

  getUserProfile: async (userId) => {
    const user = await db('users')
      .select('id', 'username', 'email', 'created_at', 'updated_at')
      .where({ id: userId })
      .first()
      
    if (!user) {
      throw new Error('User not found')
    }
    
    return user
  },

  changePassword: async (userId, { currentPassword, newPassword }) => {
    const user = await db('users').where({ id: userId }).first()
    
    if (!user) {
      throw new Error('User not found')
    }

    const isCurrentPasswordValid = await bcrypt.compare(currentPassword, user.password)
    
    if (!isCurrentPasswordValid) {
      throw new Error('Current password is incorrect')
    }

    const hashedNewPassword = await bcrypt.hash(newPassword, SALT_ROUNDS)
    
    await db('users')
      .where({ id: userId })
      .update({ 
        password: hashedNewPassword,
        updated_at: new Date()
      })
    
    return { message: 'Password changed successfully' }
  }
}

module.exports = authService
```

### Step 14: Tạo Todo Service
```javascript
// src/services/todoService.js
const db = require('../config/db')

const todoService = {
  getTodosByUser: async (userId, { page = 1, limit = 10, completed, priority, search }) => {
    const offset = (page - 1) * limit
    
    let query = db('todos').where({ user_id: userId })
    let countQuery = db('todos').where({ user_id: userId })
    
    // Apply filters
    if (typeof completed === 'boolean') {
      query = query.where({ completed })
      countQuery = countQuery.where({ completed })
    }
    
    if (priority) {
      query = query.where({ priority })
      countQuery = countQuery.where({ priority })
    }
    
    if (search) {
      const searchTerm = `%${search}%`
      query = query.where(function() {
        this.where('title', 'like', searchTerm)
            .orWhere('description', 'like', searchTerm)
      })
      countQuery = countQuery.where(function() {
        this.where('title', 'like', searchTerm)
            .orWhere('description', 'like', searchTerm)
      })
    }
    
    // Execute queries
    const [todos, totalResult] = await Promise.all([
      query
        .orderBy('created_at', 'desc')
        .limit(limit)
        .offset(offset),
      countQuery.count('id as count').first()
    ])
    
    const total = parseInt(totalResult.count)
    
    return {
      data: todos,
      pagination: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit),
        hasNext: page * limit < total,
        hasPrev: page > 1
      }
    }
  },

  getTodoById: async (todoId, userId) => {
    const todo = await db('todos')
      .where({ id: todoId, user_id: userId })
      .first()
    
    if (!todo) {
      throw new Error('Todo not found')
    }
    
    return todo
  },

  createTodo: async (userId, todoData) => {
    const trx = await db.transaction()
    
    try {
      const [newTodo] = await trx('todos').insert({
        ...todoData,
        user_id: userId,
        completed: false,
        created_at: new Date(),
        updated_at: new Date()
      }).returning('*')
      
      await trx.commit()
      return newTodo
      
    } catch (error) {
      await trx.rollback()
      throw new Error('Failed to create todo')
    }
  },

  updateTodo: async (todoId, userId, updateData) => {
    const todo = await db('todos')
      .where({ id: todoId, user_id: userId })
      .first()
    
    if (!todo) {
      throw new Error('Todo not found')
    }
    
    const [updatedTodo] = await db('todos')
      .where({ id: todoId })
      .update({
        ...updateData,
        updated_at: new Date()
      })
      .returning('*')
    
    return updatedTodo
  },

  deleteTodo: async (todoId, userId) => {
    const deleted = await db('todos')
      .where({ id: todoId, user_id: userId })
      .del()
    
    if (!deleted) {
      throw new Error('Todo not found')
    }
    
    return { message: 'Todo deleted successfully' }
  },

  toggleTodoStatus: async (todoId, userId) => {
    const todo = await db('todos')
      .where({ id: todoId, user_id: userId })
      .first()
    
    if (!todo) {
      throw new Error('Todo not found')
    }
    
    const [updatedTodo] = await db('todos')
      .where({ id: todoId })
      .update({
        completed: !todo.completed,
        updated_at: new Date()
      })
      .returning('*')
    
    return updatedTodo
  },

  getUserStats: async (userId) => {
    const stats = await db('todos')
      .where({ user_id: userId })
      .select(
        db.raw('COUNT(*) as total'),
        db.raw('COUNT(CASE WHEN completed = 1 THEN 1 END) as completed'),
        db.raw('COUNT(CASE WHEN completed = 0 THEN 1 END) as pending'),
        db.raw('COUNT(CASE WHEN priority = ? THEN 1 END)', ['high']),
        db.raw('COUNT(CASE WHEN priority = ? THEN 1 END)', ['medium']),
        db.raw('COUNT(CASE WHEN priority = ? THEN 1 END)', ['low'])
      )
      .first()
    
    return {
      total: parseInt(stats.total),
      completed: parseInt(stats.completed),
      pending: parseInt(stats.pending),
      priorities: {
        high: parseInt(stats[3]), // COUNT result for high priority
        medium: parseInt(stats[4]), // COUNT result for medium priority  
        low: parseInt(stats[5]) // COUNT result for low priority
      }
    }
  }
}

module.exports = todoService
```

## Phase 6: Router Implementation (2-3 ngày)

### Step 15: Tạo Auth Router
```javascript
// src/routers/authRouter.js
const authService = require('../services/authService')
const { 
  registerSchema, 
  loginSchema, 
  changePasswordSchema,
  userResponseSchema,
  loginResponseSchema,
  errorSchema 
} = require('../schemas/authSchemas')

async function authRouter(fastify, options) {
  
  // POST /api/auth/register
  fastify.post('/register', {
    schema: {
      body: {
        type: 'object',
        required: ['username', 'email', 'password'],
        properties: {
          username: { type: 'string', minLength: 3, maxLength: 30 },
          email: { type: 'string', format: 'email' },
          password: { type: 'string', minLength: 6 }
        }
      },
      response: {
        201: userResponseSchema,
        400: errorSchema,
        500: errorSchema
      }
    },
    preHandler: [fastify.validateJoi(registerSchema)]
  }, async (request, reply) => {
    try {
      const user = await authService.register(request.validatedBody)
      return reply.code(201).send(user)
    } catch (error) {
      fastify.log.error(error)
      return reply.code(400).send({ 
        error: error.message,
        code: 'REGISTRATION_FAILED' 
      })
    }
  })

  // POST /api/auth/login
  fastify.post('/login', {
    schema: {
      body: {
        type: 'object',
        required: ['email', 'password'],
        properties: {
          email: { type: 'string', format: 'email' },
          password: { type: 'string' }
        }
      },
      response: {
        200: loginResponseSchema,
        401: errorSchema,
        500: errorSchema
      }
    },
    preHandler: [fastify.validateJoi(loginSchema)]
  }, async (request, reply) => {
    try {
      const result = await authService.login(request.validatedBody)
      return reply.send(result)
    } catch (error) {
      fastify.log.error(error)
      return reply.code(401).send({ 
        error: error.message,
        code: 'LOGIN_FAILED' 
      })
    }
  })

  // GET /api/auth/profile
  fastify.get('/profile', {
    schema: {
      response: {
        200: userResponseSchema,
        401: errorSchema,
        404: errorSchema
      }
    },
    preHandler: [fastify.authenticate]
  }, async (request, reply) => {
    try {
      const user = await authService.getUserProfile(request.user.id)
      return reply.send(user)
    } catch (error) {
      fastify.log.error(error)
      return reply.code(404).send({ 
        error: error.message,
        code: 'USER_NOT_FOUND' 
      })
    }
  })

  // PUT /api/auth/change-password
  fastify.put('/change-password', {
    schema: {
      body: {
        type: 'object',
        required: ['currentPassword', 'newPassword'],
        properties: {
          currentPassword: { type: 'string' },
          newPassword: { type: 'string', minLength: 6 }
        }
      },
      response: {
        200: {
          type: 'object',
          properties: {
            message: { type: 'string' }
          }
        },
        400: errorSchema,
        401: errorSchema
      }
    },
    preHandler: [
      fastify.authenticate,
      fastify.validateJoi(changePasswordSchema)
    ]
  }, async (request, reply) => {
    try {
      const result = await authService.changePassword(
        request.user.id, 
        request.validatedBody
      )
      return reply.send(result)
    } catch (error) {
      fastify.log.error(error)
      return reply.code(400).send({ 
        error: error.message,
        code: 'PASSWORD_CHANGE_FAILED' 
      })
    }
  })
}

module.exports = authRouter
```

### Step 16: Tạo Todo Router
```javascript
// src/routers/todosRouter.js
const todoService = require('../services/todoService')
const { 
  createTodoSchema, 
  updateTodoSchema,
  todoResponseSchema,
  todoListResponseSchema
} = require('../schemas/todoSchemas')
const { errorSchema } = require('../schemas/authSchemas')

async function todosRouter(fastify, options) {
  
  // Apply authentication to all routes in this plugin
  fastify.addHook('preHandler', fastify.authenticate)
  
  // GET /api/todos - List todos with filtering and pagination
  fastify.get('/', {
    schema: {
      querystring: {
        type: 'object',
        properties: {
          page: { type: 'integer', minimum: 1, default: 1 },
          limit: { type: 'integer', minimum: 1, maximum: 100, default: 10 },
          completed: { type: 'boolean' },
          priority: { type: 'string', enum: ['low', 'medium', 'high'] },
          search: { type: 'string', maxLength: 100 }
        }
      },
      response: {
        200: todoListResponseSchema,
        400: errorSchema,
        401: errorSchema
      }
    },
    preHandler: [fastify.validatePagination]
  }, async (request, reply) => {
    try {
      const { page, limit } = request.pagination
      const { completed, priority, search } = request.query
      
      const result = await todoService.getTodosByUser(request.user.id, {
        page,
        limit,
        completed,
        priority,
        search
      })
      
      return reply.send(result)
    } catch (error) {
      fastify.log.error(error)
      return reply.code(500).send({ 
        error: 'Failed to fetch todos',
        code: 'FETCH_TODOS_FAILED' 
      })
    }
  })

  // GET /api/todos/:id - Get single todo
  fastify.get('/:id', {
    schema: {
      params: {
        type: 'object',
        properties: {
          id: { type: 'integer' }
        }
      },
      response: {
        200: todoResponseSchema,
        404: errorSchema,
        401: errorSchema
      }
    }
  }, async (request, reply) => {
    try {
      const todo = await todoService.getTodoById(request.params.id, request.user.id)
      return reply.send(todo)
    } catch (error) {
      fastify.log.error(error)
      return reply.code(404).send({ 
        error: error.message,
        code: 'TODO_NOT_FOUND' 
      })
    }
  })

  // POST /api/todos - Create new todo
  fastify.post('/', {
    schema: {
      body: {
        type: 'object',
        required: ['title'],
        properties: {
          title: { type: 'string', minLength: 1, maxLength: 200 },
          description: { type: 'string', maxLength: 1000 },
          priority: { type: 'string', enum: ['low', 'medium', 'high'], default: 'medium' }
        }
      },
      response: {
        201: todoResponseSchema,
        400: errorSchema,
        401: errorSchema
      }
    },
    preHandler: [fastify.validateJoi(createTodoSchema)]
  }, async (request, reply) => {
    try {
      const todo = await todoService.createTodo(request.user.id, request.validatedBody)
      return reply.code(201).send(todo)
    } catch (error) {
      fastify.log.error(error)
      return reply.code(400).send({ 
        error: error.message,
        code: 'TODO_CREATION_FAILED' 
      })
    }
  })

  // PUT /api/todos/:id - Update todo
  fastify.put('/:id', {
    schema: {
      params: {
        type: 'object',
        properties: {
          id: { type: 'integer' }
        }
      },
      body: {
        type: 'object',
        properties: {
          title: { type: 'string', minLength: 1, maxLength: 200 },
          description: { type: 'string', maxLength: 1000 },
          completed: { type: 'boolean' },
          priority: { type: 'string', enum: ['low', 'medium', 'high'] }
        }
      },
      response: {
        200: todoResponseSchema,
        400: errorSchema,
        404: errorSchema,
        401: errorSchema
      }
    },
    preHandler: [fastify.validateJoi(updateTodoSchema)]
  }, async (request, reply) => {
    try {
      const todo = await todoService.updateTodo(
        request.params.id, 
        request.user.id, 
        request.validatedBody
      )
      return reply.send(todo)
    } catch (error) {
      fastify.log.error(error)
      const statusCode = error.message === 'Todo not found' ? 404 : 400
      return reply.code(statusCode).send({ 
        error: error.message,
        code: error.message === 'Todo not found' ? 'TODO_NOT_FOUND' : 'TODO_UPDATE_FAILED'
      })
    }
  })

  // PATCH /api/todos/:id/toggle - Toggle todo completion status
  fastify.patch('/:id/toggle', {
    schema: {
      params: {
        type: 'object',
        properties: {
          id: { type: 'integer' }
        }
      },
      response: {
        200: todoResponseSchema,
        404: errorSchema,
        401: errorSchema
      }
    }
  }, async (request, reply) => {
    try {
      const todo = await todoService.toggleTodoStatus(request.params.id, request.user.id)
      return reply.send(todo)
    } catch (error) {
      fastify.log.error(error)
      return reply.code(404).send({ 
        error: error.message,
        code: 'TODO_NOT_FOUND' 
      })
    }
  })

  // DELETE /api/todos/:id - Delete todo
  fastify.delete('/:id', {
    schema: {
      params: {
        type: 'object',
        properties: {
          id: { type: 'integer' }
        }
      },
      response: {
        200: {
          type: 'object',
          properties: {
            message: { type: 'string' }
          }
        },
        404: errorSchema,
        401: errorSchema
      }
    }
  }, async (request, reply) => {
    try {
      const result = await todoService.deleteTodo(request.params.id, request.user.id)
      return reply.send(result)
    } catch (error) {
      fastify.log.error(error)
      return reply.code(404).send({ 
        error: error.message,
        code: 'TODO_NOT_FOUND' 
      })
    }
  })

  // GET /api/todos/stats - Get user todo statistics
  fastify.get('/stats', {
    schema: {
      response: {
        200: {
          type: 'object',
          properties: {
            total: { type: 'integer' },
            completed: { type: 'integer' },
            pending: { type: 'integer' },
            priorities: {
              type: 'object',
              properties: {
                high: { type: 'integer' },
                medium: { type: 'integer' },
                low: { type: 'integer' }
              }
            }
          }
        },
        401: errorSchema
      }
    }
  }, async (request, reply) => {
    try {
      const stats = await todoService.getUserStats(request.user.id)
      return reply.send(stats)
    } catch (error) {
      fastify.log.error(error)
      return reply.code(500).send({ 
        error: 'Failed to fetch statistics',
        code: 'STATS_FETCH_FAILED' 
      })
    }
  })
}

module.exports = todosRouter
```

### Step 17: Tạo User Router (Optional)
```javascript
// src/routers/userRouter.js
const authService = require('../services/authService')
const { userResponseSchema, errorSchema } = require('../schemas/authSchemas')

async function userRouter(fastify, options) {
  
  // Apply authentication to all user routes
  fastify.addHook('preHandler', fastify.authenticate)
  
  // GET /api/users/me - Get current user profile (alias for auth/profile)
  fastify.get('/me', {
    schema: {
      response: {
        200: userResponseSchema,
        401: errorSchema,
        404: errorSchema
      }
    }
  }, async (request, reply) => {
    try {
      const user = await authService.getUserProfile(request.user.id)
      return reply.send(user)
    } catch (error) {
      fastify.log.error(error)
      return reply.code(404).send({ 
        error: error.message,
        code: 'USER_NOT_FOUND' 
      })
    }
  })

  // PUT /api/users/me - Update user profile
  fastify.put('/me', {
    schema: {
      body: {
        type: 'object',
        properties: {
          username: { type: 'string', minLength: 3, maxLength: 30 }
        }
      },
      response: {
        200: userResponseSchema,
        400: errorSchema,
        401: errorSchema
      }
    }
  }, async (request, reply) => {
    try {
      // This would require implementing updateUserProfile in authService
      // For now, return current user data
      const user = await authService.getUserProfile(request.user.id)
      return reply.send(user)
    } catch (error) {
      fastify.log.error(error)
      return reply.code(400).send({ 
        error: error.message,
        code: 'PROFILE_UPDATE_FAILED' 
      })
    }
  })
}

module.exports = userRouter
```

## Phase 7: Main Server Setup (1 ngày)

### Step 18: Tạo Main Server File
```javascript
// server.js
require('dotenv').config()

const fastify = require('fastify')({ 
  logger: {
    level: process.env.LOG_LEVEL || 'info',
    prettyPrint: process.env.NODE_ENV !== 'production'
  }
})

// Global error handler
fastify.setErrorHandler(async (error, request, reply) => {
  request.log.error(error)
  
  // Handle validation errors
  if (error.validation) {
    return reply.code(400).send({
      error: 'Validation failed',
      code: 'VALIDATION_ERROR',
      details: error.validation
    })
  }
  
  // Handle Fastify errors
  if (error.statusCode) {
    return reply.code(error.statusCode).send({
      error: error.message,
      code: 'REQUEST_ERROR'
    })
  }
  
  // Handle database errors
  if (error.code?.startsWith('ER_') || error.code === 'ECONNREFUSED') {
    return reply.code(503).send({
      error: 'Database connection failed',
      code: 'DATABASE_ERROR'
    })
  }
  
  // Default error response
  return reply.code(500).send({
    error: 'Internal server error',
    code: 'INTERNAL_ERROR'
  })
})

// Handle 404 routes
fastify.setNotFoundHandler(async (request, reply) => {
  return reply.code(404).send({
    error: `Route ${request.method} ${request.url} not found`,
    code: 'ROUTE_NOT_FOUND'
  })
})

// Register CORS
fastify.register(require('@fastify/cors'), {
  origin: [
    'http://localhost:3000',
    'http://localhost:3001',
    // Add your frontend URLs here
  ],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS']
})

// Register custom plugins
fastify.register(require('./src/plugins/authPlugins'))
fastify.register(require('./src/plugins/validatePlugins'))

// Health check endpoint
fastify.get('/health', async (request, reply) => {
  return { 
    status: 'OK', 
    timestamp: new Date().toISOString(),
    uptime: process.uptime()
  }
})

// Register API routes
fastify.register(require('./src/routers/authRouter'), { prefix: '/api/auth' })
fastify.register(require('./src/routers/todosRouter'), { prefix: '/api/todos' })
fastify.register(require('./src/routers/userRouter'), { prefix: '/api/users' })

// Start server
const start = async () => {
  try {
    const port = process.env.PORT || 3000
    const host = process.env.HOST || '0.0.0.0'
    
    await fastify.listen({ port, host })
    
    console.log(`
🚀 Server running on http://${host}:${port}
📝 API Documentation: http://${host}:${port}/documentation
🔍 Health Check: http://${host}:${port}/health
    `)
    
  } catch (err) {
    fastify.log.error(err)
    process.exit(1)
  }
}

// Graceful shutdown
process.on('SIGINT', async () => {
  console.log('\n🛑 Shutting down server...')
  try {
    await fastify.close()
    console.log('✅ Server shutdown complete')
    process.exit(0)
  } catch (err) {
    console.error('❌ Error during shutdown:', err)
    process.exit(1)
  }
})

start()
```

## Phase 8: Testing & Documentation (2-3 ngày)

### Step 19: Tạo Test Setup
```bash
# Cài đặt testing dependencies
npm install tap supertest --save-dev
```

```javascript
// test/helper.js
const { build } = require('fastify')
const fp = require('fastify-plugin')

// Mock database for testing
const buildApp = (t) => {
  const app = require('fastify')({ logger: false })
  
  // Register plugins
  app.register(require('../src/plugins/authPlugins'))
  app.register(require('../src/plugins/validatePlugins'))
  
  // Register routes
  app.register(require('../src/routers/authRouter'), { prefix: '/api/auth' })
  app.register(require('../src/routers/todosRouter'), { prefix: '/api/todos' })
  
  // Clean up after test
  t.teardown(() => app.close())
  
  return app
}

module.exports = { build: buildApp }
```

```javascript
// test/auth.test.js
const { test } = require('tap')
const { build } = require('./helper')

test('POST /api/auth/register - should register new user', async (t) => {
  const app = build(t)
  
  const response = await app.inject({
    method: 'POST',
    url: '/api/auth/register',
    payload: {
      username: 'testuser',
      email: 'test@example.com',
      password: 'Password123'
    }
  })
  
  t.equal(response.statusCode, 201)
  t.ok(response.json().id)
  t.equal(response.json().email, 'test@example.com')
  t.notOk(response.json().password) // Password should not be returned
})

test('POST /api/auth/login - should login user', async (t) => {
  const app = build(t)
  
  // First register a user
  await app.inject({
    method: 'POST',
    url: '/api/auth/register',
    payload: {
      username: 'testuser',
      email: 'test@example.com',
      password: 'Password123'
    }
  })
  
  // Then login
  const response = await app.inject({
    method: 'POST',
    url: '/api/auth/login',
    payload: {
      email: 'test@example.com',
      password: 'Password123'
    }
  })
  
  t.equal(response.statusCode, 200)
  t.ok(response.json().token)
  t.ok(response.json().user)
  t.equal(response.json().user.email, 'test@example.com')
})
```

### Step 20: Environment Configuration
```bash
# Tạo file .env.example
cp .env .env.example
```

```env
# .env.example
NODE_ENV=development
PORT=3000
HOST=0.0.0.0
LOG_LEVEL=info

# JWT Configuration
JWT_SECRET=your-super-secret-jwt-key-change-this-in-production
JWT_EXPIRES_IN=24h

# Database Configuration
DB_HOST=localhost
DB_PORT=1433
DB_NAME=todo_db
DB_USER=sa
DB_PASSWORD=your_secure_password

# CORS Origins (comma-separated)
CORS_ORIGIN=http://localhost:3000,http://localhost:3001
```

### Step 21: Production Deployment Preparation
```javascript
// ecosystem.config.js (for PM2)
module.exports = {
  apps: [{
    name: 'todo-backend',
    script: 'server.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'development'
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 8000
    },
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    log_file: './logs/combined.log',
    time: true
  }]
}
```

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Create logs directory
RUN mkdir -p logs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js

# Start application
CMD ["node", "server.js"]
```

## Phase 9: Advanced Features (Optional - 2-3 ngày)

### Step 22: API Documentation với Swagger
```bash
npm install @fastify/swagger @fastify/swagger-ui
```

```javascript
// Add to server.js after CORS registration
await fastify.register(require('@fastify/swagger'), {
  openapi: {
    openapi: '3.0.0',
    info: {
      title: 'Todo Backend API',
      description: 'A simple todo management API',
      version: '1.0.0'
    },
    servers: [
      { url: 'http://localhost:3000', description: 'Development server' }
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT'
        }
      }
    }
  }
})

await fastify.register(require('@fastify/swagger-ui'), {
  routePrefix: '/documentation',
  uiConfig: {
    docExpansion: 'full',
    deepLinking: false
  }
})
```

### Step 23: Rate Limiting
```bash
npm install @fastify/rate-limit
```

```javascript
// Add to server.js
await fastify.register(require('@fastify/rate-limit'), {
  max: 100, // 100 requests
  timeWindow: '1 minute',
  redis: process.env.REDIS_URL, // Optional: use Redis for distributed rate limiting
})
```

### Step 24: Request Logging
```javascript
// Add to server.js after fastify creation
fastify.addHook('onRequest', async (request, reply) => {
  request.startTime = Date.now()
})

fastify.addHook('onResponse', async (request, reply) => {
  const responseTime = Date.now() - request.startTime
  request.log.info({
    method: request.method,
    url: request.url,
    statusCode: reply.statusCode,
    responseTime: `${responseTime}ms`,
    userAgent: request.headers['user-agent']
  })
})
```

## Phase 10: Deployment & Production (1-2 ngày)

### Step 25: Production Checklist
```bash
# Create production scripts
npm run migrate            # Run database migrations
npm run start             # Start production server

# Environment validation
node -e "console.log('Environment check:', {
  NODE_ENV: process.env.NODE_ENV,
  PORT: process.env.PORT,
  DB_HOST: process.env.DB_HOST,
  JWT_SECRET: process.env.JWT_SECRET ? '***' : 'MISSING'
})"
```

### Step 26: Monitoring & Logging
```javascript
// src/utils/logger.js
const pino = require('pino')

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => {
      return { level: label }
    }
  },
  timestamp: pino.stdTimeFunctions.isoTime,
  redact: ['password', 'authorization'] // Hide sensitive data in logs
})

module.exports = logger
```

## 🎯 Summary Timeline

| Phase | Duration | Tasks |
|-------|----------|-------|
| 1 | 1-2 days | Project setup, dependencies |
| 2 | 1 day | Database setup, migrations |
| 3 | 1 day | Core utilities & plugins |
| 4 | 1 day | Schema definitions |
| 5 | 2-3 days | Service layer implementation |
| 6 | 2-3 days | Router implementation |
| 7 | 1 day | Main server setup |
| 8 | 2-3 days | Testing & documentation |
| 9 | 2-3 days | Advanced features (optional) |
| 10 | 1-2 days | Production deployment |

**Total: 12-20 days** (depending on optional features)

## 🚀 Quick Start Commands

```bash
# Development
npm run dev

# Database operations
npm run migrate
npm run migrate:rollback

# Production
npm start

# Testing
npm test

# View API docs
# http://localhost:3000/documentation
```

## 📋 Learning Outcomes

Sau khi hoàn thành roadmap này, bạn sẽ nắm vững:

1. **Fastify Core**: Plugin system, lifecycle hooks, decorators
2. **Authentication**: JWT, bcrypt, middleware implementation
3. **Database**: Knex.js, migrations, query optimization
4. **Validation**: Schema-based validation, error handling
5. **API Design**: RESTful APIs, proper status codes
6. **Security**: CORS, rate limiting, input validation
7. **Production**: Deployment, monitoring, logging