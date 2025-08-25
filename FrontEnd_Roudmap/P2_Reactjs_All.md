# React.js + TypeScript - Khóa học từ cơ bản đến nâng cao

## Mục lục
1. [Giới thiệu và Setup](#1-giới-thiệu-và-setup)
2. [TypeScript Basics cho React](#2-typescript-basics-cho-react)
3. [React Components với TypeScript](#3-react-components-với-typescript)
4. [React Hooks chi tiết](#4-react-hooks-chi-tiết)
5. [State Management](#5-state-management)
6. [API Integration](#6-api-integration)
7. [Routing](#7-routing)
8. [Form Handling](#8-form-handling)
9. [Styling và UI Libraries](#9-styling-và-ui-libraries)
10. [Testing](#10-testing)
11. [Performance Optimization](#11-performance-optimization)
12. [Dự án thực tế](#12-dự-án-thực-tế)

## 1. Giới thiệu và Setup

### 1.1 Tạo project React với TypeScript

```bash
# Sử dụng Vite (khuyên dùng - nhanh hơn)
npm create vite@latest my-react-app -- --template react-ts
cd my-react-app
npm install

# Hoặc sử dụng Create React App
npx create-react-app my-app --template typescript
cd my-app
npm start
```

### 1.2 Cấu trúc thư mục đề xuất

```
src/
├── components/
│   ├── common/
│   └── specific/
├── hooks/
├── pages/
├── services/
├── types/
├── utils/
├── styles/
└── store/
```

### 1.3 Dependencies cần thiết cho dự án thực tế

```bash
# Core dependencies
npm install react react-dom

# TypeScript
npm install -D typescript @types/react @types/react-dom

# Routing
npm install react-router-dom
npm install -D @types/react-router-dom

# API & HTTP Client
npm install axios
npm install react-query # hoặc @tanstack/react-query (phiên bản mới)

# Form handling
npm install react-hook-form
npm install @hookform/resolvers yup # validation

# UI Libraries
npm install @mui/material @emotion/react @emotion/styled
# hoặc
npm install antd
# hoặc
npm install tailwindcss

# State Management
npm install zustand
# hoặc
npm install @reduxjs/toolkit react-redux

# Utilities
npm install lodash
npm install -D @types/lodash
npm install date-fns
npm install uuid
npm install -D @types/uuid

# Development tools
npm install -D eslint @typescript-eslint/eslint-plugin
npm install -D prettier
```

## 2. TypeScript Basics cho React

### 2.1 Định nghĩa Types cho React

```typescript
// types/index.ts
export interface User {
  id: number;
  name: string;
  email: string;
  avatar?: string; // optional
}

export interface Product {
  id: number;
  title: string;
  price: number;
  description: string;
  category: string;
  image: string;
}

export interface ApiResponse<T> {
  data: T;
  message: string;
  status: number;
}

// Props types
export interface ButtonProps {
  children: React.ReactNode;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
}

export interface InputProps {
  value: string;
  onChange: (value: string) => void;
  placeholder?: string;
  type?: 'text' | 'email' | 'password';
}
```

### 2.2 Utility Types hữu ích

```typescript
// Partial - làm tất cả properties thành optional
type PartialUser = Partial<User>;

// Pick - chọn một số properties
type UserBasic = Pick<User, 'id' | 'name'>;

// Omit - loại bỏ một số properties  
type UserWithoutId = Omit<User, 'id'>;

// Record - tạo object type
type UserRoles = Record<string, 'admin' | 'user' | 'guest'>;
```

## 3. React Components với TypeScript

### 3.1 Function Components

```typescript
// components/Button.tsx
import React from 'react';

interface ButtonProps {
  children: React.ReactNode;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  className?: string;
}

const Button: React.FC<ButtonProps> = ({
  children,
  onClick,
  variant = 'primary',
  disabled = false,
  className = ''
}) => {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant} ${className}`}
    >
      {children}
    </button>
  );
};

export default Button;
```

### 3.2 Generic Components

```typescript
// components/List.tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string | number;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map((item) => (
        <li key={keyExtractor(item)}>
          {renderItem(item)}
        </li>
      ))}
    </ul>
  );
}

// Sử dụng
<List
  items={users}
  renderItem={(user) => <span>{user.name}</span>}
  keyExtractor={(user) => user.id}
/>
```

## 4. React Hooks chi tiết

### 4.1 useState Hook

```typescript
import React, { useState } from 'react';

const Counter: React.FC = () => {
  // Basic useState
  const [count, setCount] = useState<number>(0);
  
  // useState with object
  const [user, setUser] = useState<User | null>(null);
  
  // useState with array
  const [items, setItems] = useState<string[]>([]);
  
  // Complex state
  const [form, setForm] = useState<{
    name: string;
    email: string;
    age: number;
  }>({
    name: '',
    email: '',
    age: 0
  });

  const updateForm = (field: keyof typeof form, value: string | number) => {
    setForm(prev => ({
      ...prev,
      [field]: value
    }));
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(prev => prev + 1)}>
        Increment
      </button>
    </div>
  );
};
```

### 4.2 useEffect Hook

```typescript
import React, { useState, useEffect } from 'react';

const UserProfile: React.FC<{ userId: number }> = ({ userId }) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  // Effect chạy khi component mount và khi userId thay đổi
  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        const userData = await response.json();
        setUser(userData);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Unknown error');
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId]);

  // Effect với cleanup
  useEffect(() => {
    const timer = setInterval(() => {
      console.log('Timer tick');
    }, 1000);

    // Cleanup function
    return () => {
      clearInterval(timer);
    };
  }, []);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>User not found</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
};
```

### 4.3 useContext Hook

```typescript
// contexts/AuthContext.tsx
import React, { createContext, useContext, useState } from 'react';

interface AuthContextType {
  user: User | null;
  login: (user: User) => void;
  logout: () => void;
  isAuthenticated: boolean;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);

  const login = (userData: User) => {
    setUser(userData);
    localStorage.setItem('user', JSON.stringify(userData));
  };

  const logout = () => {
    setUser(null);
    localStorage.removeItem('user');
  };

  const value: AuthContextType = {
    user,
    login,
    logout,
    isAuthenticated: !!user
  };

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

### 4.4 useReducer Hook

```typescript
import React, { useReducer } from 'react';

// Định nghĩa types cho reducer
type ActionType = 
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'RESET' }
  | { type: 'SET_VALUE'; payload: number };

interface CounterState {
  count: number;
  history: number[];
}

// Reducer function
const counterReducer = (state: CounterState, action: ActionType): CounterState => {
  switch (action.type) {
    case 'INCREMENT':
      return {
        count: state.count + 1,
        history: [...state.history, state.count + 1]
      };
    case 'DECREMENT':
      return {
        count: state.count - 1,
        history: [...state.history, state.count - 1]
      };
    case 'RESET':
      return {
        count: 0,
        history: [0]
      };
    case 'SET_VALUE':
      return {
        count: action.payload,
        history: [...state.history, action.payload]
      };
    default:
      return state;
  }
};

const Counter: React.FC = () => {
  const [state, dispatch] = useReducer(counterReducer, {
    count: 0,
    history: [0]
  });

  return (
    <div>
      <p>Count: {state.count}</p>
      <p>History: {state.history.join(', ')}</p>
      
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>
        +
      </button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>
        -
      </button>
      <button onClick={() => dispatch({ type: 'RESET' })}>
        Reset
      </button>
    </div>
  );
};
```

### 4.5 Custom Hooks

```typescript
// hooks/useLocalStorage.ts
import { useState, useEffect } from 'react';

function useLocalStorage<T>(
  key: string, 
  initialValue: T
): [T, (value: T | ((val: T) => T)) => void] {
  // Lấy giá trị từ localStorage
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });

  // Hàm để set giá trị
  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(`Error setting localStorage key "${key}":`, error);
    }
  };

  return [storedValue, setValue];
}

// hooks/useApi.ts
import { useState, useEffect } from 'react';
import axios from 'axios';

interface UseApiResult<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
  refetch: () => void;
}

function useApi<T>(url: string): UseApiResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  const fetchData = async () => {
    try {
      setLoading(true);
      setError(null);
      const response = await axios.get<T>(url);
      setData(response.data);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Unknown error');
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchData();
  }, [url]);

  return { data, loading, error, refetch: fetchData };
}

// Sử dụng custom hooks
const UsersList: React.FC = () => {
  const { data: users, loading, error, refetch } = useApi<User[]>('/api/users');
  const [favoriteUsers, setFavoriteUsers] = useLocalStorage<number[]>('favoriteUsers', []);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <button onClick={refetch}>Refresh</button>
      {users?.map(user => (
        <div key={user.id}>
          <h3>{user.name}</h3>
          <button
            onClick={() => setFavoriteUsers(prev => 
              prev.includes(user.id) 
                ? prev.filter(id => id !== user.id)
                : [...prev, user.id]
            )}
          >
            {favoriteUsers.includes(user.id) ? 'Unfavorite' : 'Favorite'}
          </button>
        </div>
      ))}
    </div>
  );
};
```

### 4.6 useMemo và useCallback

```typescript
import React, { useMemo, useCallback, useState } from 'react';

interface ExpensiveComponentProps {
  items: Product[];
  searchTerm: string;
}

const ExpensiveComponent: React.FC<ExpensiveComponentProps> = ({ items, searchTerm }) => {
  const [sortOrder, setSortOrder] = useState<'asc' | 'desc'>('asc');

  // useMemo - cache kết quả tính toán expensive
  const filteredAndSortedItems = useMemo(() => {
    console.log('Filtering and sorting...'); // Chỉ log khi thực sự tính toán lại
    
    const filtered = items.filter(item => 
      item.title.toLowerCase().includes(searchTerm.toLowerCase())
    );
    
    return filtered.sort((a, b) => {
      if (sortOrder === 'asc') {
        return a.price - b.price;
      } else {
        return b.price - a.price;
      }
    });
  }, [items, searchTerm, sortOrder]);

  // useCallback - cache function
  const handleSort = useCallback(() => {
    setSortOrder(prev => prev === 'asc' ? 'desc' : 'asc');
  }, []);

  const handleItemClick = useCallback((itemId: number) => {
    console.log('Item clicked:', itemId);
    // Handle item click logic
  }, []);

  return (
    <div>
      <button onClick={handleSort}>
        Sort by price ({sortOrder})
      </button>
      
      {filteredAndSortedItems.map(item => (
        <ProductItem 
          key={item.id}
          item={item}
          onClick={handleItemClick} // Function được cache
        />
      ))}
    </div>
  );
};

// Component con được optimize với React.memo
interface ProductItemProps {
  item: Product;
  onClick: (id: number) => void;
}

const ProductItem = React.memo<ProductItemProps>(({ item, onClick }) => {
  console.log('ProductItem render:', item.title);
  
  return (
    <div onClick={() => onClick(item.id)}>
      <h3>{item.title}</h3>
      <p>${item.price}</p>
    </div>
  );
});
```

## 5. State Management

### 5.1 Zustand (Đơn giản và hiệu quả)

```typescript
// store/useStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface AppState {
  // User state
  user: User | null;
  setUser: (user: User | null) => void;
  
  // Products state
  products: Product[];
  setProducts: (products: Product[]) => void;
  addProduct: (product: Product) => void;
  
  // Cart state
  cart: { productId: number; quantity: number }[];
  addToCart: (productId: number) => void;
  removeFromCart: (productId: number) => void;
  clearCart: () => void;
  
  // UI state
  isLoading: boolean;
  setLoading: (loading: boolean) => void;
}

export const useStore = create<AppState>()(
  persist(
    (set, get) => ({
      // User state
      user: null,
      setUser: (user) => set({ user }),
      
      // Products state
      products: [],
      setProducts: (products) => set({ products }),
      addProduct: (product) => set((state) => ({
        products: [...state.products, product]
      })),
      
      // Cart state
      cart: [],
      addToCart: (productId) => set((state) => {
        const existingItem = state.cart.find(item => item.productId === productId);
        if (existingItem) {
          return {
            cart: state.cart.map(item =>
              item.productId === productId
                ? { ...item, quantity: item.quantity + 1 }
                : item
            )
          };
        }
        return {
          cart: [...state.cart, { productId, quantity: 1 }]
        };
      }),
      removeFromCart: (productId) => set((state) => ({
        cart: state.cart.filter(item => item.productId !== productId)
      })),
      clearCart: () => set({ cart: [] }),
      
      // UI state
      isLoading: false,
      setLoading: (isLoading) => set({ isLoading })
    }),
    {
      name: 'app-storage', // localStorage key
      partialize: (state) => ({ 
        user: state.user, 
        cart: state.cart 
      }), // Chỉ persist một số state
    }
  )
);

// Sử dụng trong component
const ShoppingCart: React.FC = () => {
  const { cart, products, addToCart, removeFromCart, clearCart } = useStore();
  
  const cartItems = cart.map(cartItem => ({
    ...products.find(p => p.id === cartItem.productId)!,
    quantity: cartItem.quantity
  }));

  const total = cartItems.reduce((sum, item) => 
    sum + (item.price * item.quantity), 0
  );

  return (
    <div>
      <h2>Shopping Cart</h2>
      {cartItems.map(item => (
        <div key={item.id}>
          <span>{item.title} x {item.quantity}</span>
          <span>${item.price * item.quantity}</span>
          <button onClick={() => removeFromCart(item.id)}>Remove</button>
        </div>
      ))}
      <div>Total: ${total}</div>
      <button onClick={clearCart}>Clear Cart</button>
    </div>
  );
};
```

## 6. API Integration

### 6.1 Axios Setup

```typescript
// services/api.ts
import axios, { AxiosRequestConfig, AxiosResponse } from 'axios';

// Tạo axios instance
const api = axios.create({
  baseURL: process.env.REACT_APP_API_BASE_URL || 'https://api.example.com',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('authToken');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Response interceptor
api.interceptors.response.use(
  (response: AxiosResponse) => {
    return response;
  },
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('authToken');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default api;
```

### 6.2 API Services

```typescript
// services/userService.ts
import api from './api';
import { ApiResponse, User } from '../types';

export const userService = {
  // GET all users
  getUsers: async (): Promise<User[]> => {
    const response = await api.get<ApiResponse<User[]>>('/users');
    return response.data.data;
  },

  // GET user by id
  getUserById: async (id: number): Promise<User> => {
    const response = await api.get<ApiResponse<User>>(`/users/${id}`);
    return response.data.data;
  },

  // POST create user
  createUser: async (userData: Omit<User, 'id'>): Promise<User> => {
    const response = await api.post<ApiResponse<User>>('/users', userData);
    return response.data.data;
  },

  // PUT update user
  updateUser: async (id: number, userData: Partial<User>): Promise<User> => {
    const response = await api.put<ApiResponse<User>>(`/users/${id}`, userData);
    return response.data.data;
  },

  // DELETE user
  deleteUser: async (id: number): Promise<void> => {
    await api.delete(`/users/${id}`);
  }
};

// services/productService.ts
export const productService = {
  getProducts: async (params?: {
    category?: string;
    search?: string;
    page?: number;
    limit?: number;
  }): Promise<Product[]> => {
    const response = await api.get<ApiResponse<Product[]>>('/products', { params });
    return response.data.data;
  },

  getProductById: async (id: number): Promise<Product> => {
    const response = await api.get<ApiResponse<Product>>(`/products/${id}`);
    return response.data.data;
  }
};
```

### 6.3 React Query Integration

```typescript
// hooks/useUsers.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { userService } from '../services/userService';
import { User } from '../types';

// GET users
export const useUsers = () => {
  return useQuery({
    queryKey: ['users'],
    queryFn: userService.getUsers,
    staleTime: 5 * 60 * 1000, // 5 minutes
    cacheTime: 10 * 60 * 1000, // 10 minutes
  });
};

// GET user by id
export const useUser = (id: number) => {
  return useQuery({
    queryKey: ['user', id],
    queryFn: () => userService.getUserById(id),
    enabled: !!id, // Chỉ chạy khi có id
  });
};

// CREATE user
export const useCreateUser = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: userService.createUser,
    onSuccess: () => {
      // Invalidate và refetch users list
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
    onError: (error) => {
      console.error('Error creating user:', error);
    }
  });
};

// UPDATE user
export const useUpdateUser = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: ({ id, userData }: { id: number; userData: Partial<User> }) =>
      userService.updateUser(id, userData),
    onSuccess: (updatedUser) => {
      // Update cache
      queryClient.setQueryData(['user', updatedUser.id], updatedUser);
      queryClient.invalidateQueries({ queryKey: ['users'] });
    }
  });
};

// Sử dụng trong component
const UserManagement: React.FC = () => {
  const { data: users, isLoading, error } = useUsers();
  const createUserMutation = useCreateUser();
  const updateUserMutation = useUpdateUser();

  const handleCreateUser = async (userData: Omit<User, 'id'>) => {
    try {
      await createUserMutation.mutateAsync(userData);
      alert('User created successfully!');
    } catch (error) {
      alert('Error creating user');
    }
  };

  if (isLoading) return <div>Loading users...</div>;
  if (error) return <div>Error loading users</div>;

  return (
    <div>
      <h2>Users</h2>
      {users?.map(user => (
        <div key={user.id}>
          <span>{user.name} - {user.email}</span>
          <button 
            onClick={() => updateUserMutation.mutate({
              id: user.id,
              userData: { name: user.name + ' (Updated)' }
            })}
            disabled={updateUserMutation.isPending}
          >
            Update
          </button>
        </div>
      ))}
      
      <button 
        onClick={() => handleCreateUser({
          name: 'New User',
          email: 'new@example.com'
        })}
        disabled={createUserMutation.isPending}
      >
        {createUserMutation.isPending ? 'Creating...' : 'Create User'}
      </button>
    </div>
  );
};
```

## 7. Routing

### 7.1 React Router Setup

```typescript
// App.tsx
import React from 'react';
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { AuthProvider } from './contexts/AuthContext';
import Layout from './components/Layout';
import Home from './pages/Home';
import Products from './pages/Products';
import ProductDetail from './pages/ProductDetail';
import Login from './pages/Login';
import Profile from './pages/Profile';
import ProtectedRoute from './components/ProtectedRoute';

const queryClient = new QueryClient();

const App: React.FC = () => {
  return (
    <QueryClientProvider client={queryClient}>
      <AuthProvider>
        <BrowserRouter>
          <Layout>
            <Routes>
              <Route path="/" element={<Home />} />
              <Route path="/login" element={<Login />} />
              <Route path="/products" element={<Products />} />
              <Route path="/products/:id" element={<ProductDetail />} />
              
              {/* Protected routes */}
              <Route
                path="/profile"
                element={
                  <ProtectedRoute>
                    <Profile />
                  </ProtectedRoute>
                }
              />
              
              {/* Redirect */}
              <Route path="/old-path" element={<Navigate to="/new-path" replace />} />
              
              {/* 404 */}
              <Route path="*" element={<div>Page Not Found</div>} />
            </Routes>
          </Layout>
        </BrowserRouter>
      </AuthProvider>
    </QueryClientProvider>
  );
};

export default App;
```

### 7.2 Protected Route Component

```typescript
// components/ProtectedRoute.tsx
import React from 'react';
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from '../contexts/AuthContext';

interface ProtectedRouteProps {
  children: React.ReactNode;
  requiredRole?: string;
}

const ProtectedRoute: React.FC<ProtectedRouteProps> = ({ 
  children, 
  requiredRole 
}) => {
  const { isAuthenticated, user } = useAuth();
  const location = useLocation();

  if (!isAuthenticated) {
    // Redirect to login page với return url
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  if (requiredRole && user?.role !== requiredRole) {
    return <Navigate to="/unauthorized" replace />;
  }

  return <>{children}</>;
};

export default ProtectedRoute;
```

### 7.3 Navigation Hook

```typescript
// hooks/useNavigation.ts
import { useNavigate, useLocation, useSearchParams } from 'react-router-dom';

export const useAppNavigation = () => {
  const navigate = useNavigate();
  const location = useLocation();
  const [searchParams, setSearchParams] = useSearchParams();

  const goToProduct = (id: number) => {
    navigate(`/products/${id}`);
  };

  const goToProducts = (filters?: { category?: string; search?: string }) => {
    const params = new URLSearchParams();
    if (filters?.category) params.set('category', filters.category);
    if (filters?.search) params.set('search', filters.search);
    
    navigate(`/products?${params.toString()}`);
  };

  const goBack = () => {
    navigate(-1);
  };

  const getCurrentPath = () => location.pathname;
  
  const getSearchParam = (key: string) => searchParams.get(key);
  
  const updateSearchParams = (params: Record<string, string>) => {
    const newParams = new URLSearchParams(searchParams);
    Object.entries(params).forEach(([key, value]) => {
      if (value) {
        newParams.set(key, value);
      } else {
        newParams.delete(key);
      }
    });
    setSearchParams(newParams);
  };

  return {
    goToProduct,
    goToProducts,
    goBack,
    getCurrentPath,
    getSearchParam,
    updateSearchParams
  };
};
```

## 8. Form Handling

### 8.1 React Hook Form với Validation

```typescript
// components/forms/UserForm.tsx
import React from 'react';
import { useForm, Controller } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import * as yup from 'yup';

// Validation schema
const userSchema = yup.object().shape({
  name: yup
    .string()
    .required('Name is required')
    .min(2, 'Name must be at least 2 characters'),
  email: yup
    .string()
    .required('Email is required')
    .email('Invalid email format'),
  age: yup
    .number()
    .required('Age is required')
    .min(18, 'Must be at least 18 years old')
    .max(100, 'Age cannot exceed 100'),
  phone: yup
    .string()
    .matches(/^[0-9+\-\s()]+$/, 'Invalid phone number'),
  website: yup
    .string()
    .url('Must be a valid URL')
    .nullable(),
  address: yup.object().shape({
    street: yup.string().required('Street is required'),
    city: yup.string().required('City is required'),
    zipCode: yup.string().required('Zip code is required')
  })
});

type UserFormData = yup.InferType<typeof userSchema>;

interface UserFormProps {
  initialData?: Partial<UserFormData>;
  onSubmit: (data: UserFormData) => Promise<void>;
  isSubmitting?: boolean;
}

const UserForm: React.FC<UserFormProps> = ({ 
  initialData, 
  onSubmit, 
  isSubmitting = false 
}) => {
  const {
    register,
    handleSubmit,
    control,
    watch,
    setValue,
    reset,
    formState: { errors, isValid, dirtyFields }
  } = useForm<UserFormData>({
    resolver: yupResolver(userSchema),
    defaultValues: {
      name: '',
      email: '',
      age: 18,
      phone: '',
      website: '',
      address: {
        street: '',
        city: '',
        zipCode: ''
      },
      ...initialData
    },
    mode: 'onChange' // Validate on change
  });

  // Watch specific fields
  const watchedAge = watch('age');

  const onFormSubmit = async (data: UserFormData) => {
    try {
      await onSubmit(data);
      reset(); // Reset form after successful submission
    } catch (error) {
      console.error('Form submission error:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit(onFormSubmit)} className="space-y-4">
      {/* Basic Input */}
      <div>
        <label htmlFor="name">Name *</label>
        <input
          id="name"
          {...register('name')}
          className={`form-input ${errors.name ? 'border-red-500' : ''}`}
        />
        {errors.name && (
          <p className="text-red-500 text-sm">{errors.name.message}</p>
        )}
      </div>

      {/* Email Input */}
      <div>
        <label htmlFor="email">Email *</label>
        <input
          id="email"
          type="email"
          {...register('email')}
          className={`form-input ${errors.email ? 'border-red-500' : ''}`}
        />
        {errors.email && (
          <p className="text-red-500 text-sm">{errors.email.message}</p>
        )}
      </div>

      {/* Number Input */}
      <div>
        <label htmlFor="age">Age *</label>
        <input
          id="age"
          type="number"
          {...register('age', { valueAsNumber: true })}
          className={`form-input ${errors.age ? 'border-red-500' : ''}`}
        />
        {errors.age && (
          <p className="text-red-500 text-sm">{errors.age.message}</p>
        )}
        {watchedAge >= 65 && (
          <p className="text-blue-500 text-sm">Senior discount available!</p>
        )}
      </div>

      {/* Controller for custom components */}
      <div>
        <label>Phone</label>
        <Controller
          name="phone"
          control={control}
          render={({ field }) => (
            <input
              {...field}
              placeholder="Enter phone number"
              className={`form-input ${errors.phone ? 'border-red-500' : ''}`}
            />
          )}
        />
        {errors.phone && (
          <p className="text-red-500 text-sm">{errors.phone.message}</p>
        )}
      </div>

      {/* Nested Object Fields */}
      <fieldset className="border p-4">
        <legend>Address</legend>
        
        <div>
          <label htmlFor="address.street">Street *</label>
          <input
            id="address.street"
            {...register('address.street')}
            className={`form-input ${errors.address?.street ? 'border-red-500' : ''}`}
          />
          {errors.address?.street && (
            <p className="text-red-500 text-sm">{errors.address.street.message}</p>
          )}
        </div>

        <div>
          <label htmlFor="address.city">City *</label>
          <input
            id="address.city"
            {...register('address.city')}
            className={`form-input ${errors.address?.city ? 'border-red-500' : ''}`}
          />
          {errors.address?.city && (
            <p className="text-red-500 text-sm">{errors.address.city.message}</p>
          )}
        </div>

        <div>
          <label htmlFor="address.zipCode">Zip Code *</label>
          <input
            id="address.zipCode"
            {...register('address.zipCode')}
            className={`form-input ${errors.address?.zipCode ? 'border-red-500' : ''}`}
          />
          {errors.address?.zipCode && (
            <p className="text-red-500 text-sm">{errors.address.zipCode.message}</p>
          )}
        </div>
      </fieldset>

      {/* Action Buttons */}
      <div className="flex space-x-4">
        <button
          type="submit"
          disabled={!isValid || isSubmitting}
          className="btn btn-primary disabled:opacity-50"
        >
          {isSubmitting ? 'Submitting...' : 'Submit'}
        </button>
        
        <button
          type="button"
          onClick={() => reset()}
          className="btn btn-secondary"
        >
          Reset
        </button>
        
        <button
          type="button"
          onClick={() => setValue('name', 'John Doe')}
          className="btn btn-outline"
        >
          Fill Sample Data
        </button>
      </div>

      {/* Debug Info (Development only) */}
      {process.env.NODE_ENV === 'development' && (
        <div className="mt-4 p-4 bg-gray-100">
          <h3>Form Debug Info:</h3>
          <p>Is Valid: {isValid ? 'Yes' : 'No'}</p>
          <p>Dirty Fields: {Object.keys(dirtyFields).join(', ')}</p>
          <p>Watch Age: {watchedAge}</p>
        </div>
      )}
    </form>
  );
};

export default UserForm;
```

### 8.2 Advanced Form Patterns

```typescript
// hooks/useFormPersistence.ts
import { useEffect } from 'react';
import { UseFormReturn } from 'react-hook-form';

export const useFormPersistence = <T extends Record<string, any>>(
  form: UseFormReturn<T>,
  key: string
) => {
  const { watch, setValue } = form;
  const watchedValues = watch();

  // Save to localStorage when form changes
  useEffect(() => {
    const subscription = watch((value) => {
      localStorage.setItem(key, JSON.stringify(value));
    });
    
    return () => subscription.unsubscribe();
  }, [watch, key]);

  // Load from localStorage on mount
  useEffect(() => {
    const saved = localStorage.getItem(key);
    if (saved) {
      try {
        const parsedData = JSON.parse(saved);
        Object.keys(parsedData).forEach((fieldKey) => {
          setValue(fieldKey as keyof T, parsedData[fieldKey]);
        });
      } catch (error) {
        console.error('Error parsing saved form data:', error);
      }
    }
  }, [setValue, key]);

  const clearSaved = () => {
    localStorage.removeItem(key);
  };

  return { clearSaved };
};

// components/forms/MultiStepForm.tsx
import React, { useState } from 'react';
import { useForm } from 'react-hook-form';

interface MultiStepFormData {
  // Step 1
  personalInfo: {
    name: string;
    email: string;
    phone: string;
  };
  // Step 2
  address: {
    street: string;
    city: string;
    country: string;
  };
  // Step 3
  preferences: {
    newsletter: boolean;
    notifications: boolean;
  };
}

const MultiStepForm: React.FC = () => {
  const [currentStep, setCurrentStep] = useState(1);
  const totalSteps = 3;

  const form = useForm<MultiStepFormData>({
    defaultValues: {
      personalInfo: { name: '', email: '', phone: '' },
      address: { street: '', city: '', country: '' },
      preferences: { newsletter: false, notifications: false }
    }
  });

  const { handleSubmit, trigger, getValues } = form;

  // Persist form data
  useFormPersistence(form, 'multiStepForm');

  const nextStep = async () => {
    let fieldsToValidate: (keyof MultiStepFormData)[] = [];
    
    switch (currentStep) {
      case 1:
        fieldsToValidate = ['personalInfo'];
        break;
      case 2:
        fieldsToValidate = ['address'];
        break;
    }

    const isValid = await trigger(fieldsToValidate);
    if (isValid) {
      setCurrentStep(prev => Math.min(prev + 1, totalSteps));
    }
  };

  const prevStep = () => {
    setCurrentStep(prev => Math.max(prev - 1, 1));
  };

  const onSubmit = (data: MultiStepFormData) => {
    console.log('Final form data:', data);
    // Submit logic here
  };

  const renderStep = () => {
    switch (currentStep) {
      case 1:
        return <PersonalInfoStep form={form} />;
      case 2:
        return <AddressStep form={form} />;
      case 3:
        return <PreferencesStep form={form} />;
      default:
        return null;
    }
  };

  return (
    <div className="max-w-md mx-auto">
      {/* Progress Bar */}
      <div className="mb-8">
        <div className="flex justify-between text-sm text-gray-600">
          <span>Step {currentStep} of {totalSteps}</span>
          <span>{Math.round((currentStep / totalSteps) * 100)}% Complete</span>
        </div>
        <div className="w-full bg-gray-200 rounded-full h-2 mt-2">
          <div 
            className="bg-blue-600 h-2 rounded-full transition-all duration-300"
            style={{ width: `${(currentStep / totalSteps) * 100}%` }}
          />
        </div>
      </div>

      <form onSubmit={handleSubmit(onSubmit)}>
        {renderStep()}

        <div className="flex justify-between mt-8">
          <button
            type="button"
            onClick={prevStep}
            disabled={currentStep === 1}
            className="btn btn-secondary disabled:opacity-50"
          >
            Previous
          </button>

          {currentStep < totalSteps ? (
            <button
              type="button"
              onClick={nextStep}
              className="btn btn-primary"
            >
              Next
            </button>
          ) : (
            <button
              type="submit"
              className="btn btn-primary"
            >
              Submit
            </button>
          )}
        </div>
      </form>
    </div>
  );
};
```

## 9. Styling và UI Libraries

### 9.1 Tailwind CSS Setup

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

```typescript
// tailwind.config.js
module.exports = {
  content: [
    "./src/**/*.{js,jsx,ts,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          500: '#3b82f6',
          600: '#2563eb',
          700: '#1d4ed8',
        }
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif'],
      },
    },
  },
  plugins: [],
}
```

### 9.2 Styled Components

```typescript
// components/ui/Button.tsx
import React from 'react';
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '../../utils/cn';

const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:opacity-50 disabled:pointer-events-none ring-offset-background",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive: "bg-destructive text-destructive-foreground hover:bg-destructive/90",
        outline: "border border-input hover:bg-accent hover:text-accent-foreground",
        secondary: "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        link: "underline-offset-4 hover:underline text-primary",
      },
      size: {
        default: "h-10 py-2 px-4",
        sm: "h-9 px-3 rounded-md",
        lg: "h-11 px-8 rounded-md",
        icon: "h-10 w-10",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean;
}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    return (
      <button
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    );
  }
);

Button.displayName = "Button";

export { Button, buttonVariants };
```

### 9.3 Material-UI Integration

```typescript
// components/ui/MuiComponents.tsx
import React from 'react';
import {
  Card,
  CardContent,
  CardActions,
  Typography,
  Button,
  TextField,
  Grid,
  Box,
  ThemeProvider,
  createTheme
} from '@mui/material';

const theme = createTheme({
  palette: {
    primary: {
      main: '#1976d2',
    },
    secondary: {
      main: '#dc004e',
    },
  },
});

interface ProductCardProps {
  product: Product;
  onAddToCart: (productId: number) => void;
}

export const ProductCard: React.FC<ProductCardProps> = ({ 
  product, 
  onAddToCart 
}) => {
  return (
    <ThemeProvider theme={theme}>
      <Card sx={{ maxWidth: 345, margin: 2 }}>
        <CardContent>
          <Typography gutterBottom variant="h5" component="div">
            {product.title}
          </Typography>
          <Typography variant="body2" color="text.secondary">
            {product.description}
          </Typography>
          <Typography variant="h6" color="primary" sx={{ mt: 2 }}>
            ${product.price}
          </Typography>
        </CardContent>
        <CardActions>
          <Button 
            size="small" 
            variant="contained"
            onClick={() => onAddToCart(product.id)}
          >
            Add to Cart
          </Button>
          <Button size="small" variant="outlined">
            View Details
          </Button>
        </CardActions>
      </Card>
    </ThemeProvider>
  );
};
```

## 10. Testing

### 10.1 Jest và React Testing Library Setup

```bash
npm install -D @testing-library/react @testing-library/jest-dom @testing-library/user-event
npm install -D jest-environment-jsdom
```

```typescript
// src/setupTests.ts
import '@testing-library/jest-dom';

// __tests__/components/Button.test.tsx
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from '../../components/ui/Button';

describe('Button Component', () => {
  test('renders button with text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument();
  });

  test('calls onClick handler when clicked', async () => {
    const handleClick = jest.fn();
    const user = userEvent.setup();
    
    render(<Button onClick={handleClick}>Click me</Button>);
    
    await user.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  test('is disabled when disabled prop is true', () => {
    render(<Button disabled>Disabled Button</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });

  test('applies correct variant classes', () => {
    render(<Button variant="secondary">Secondary Button</Button>);
    const button = screen.getByRole('button');
    expect(button).toHaveClass('bg-secondary');
  });
});
```

### 10.2 Testing Custom Hooks

```typescript
// __tests__/hooks/useLocalStorage.test.ts
import { renderHook, act } from '@testing-library/react';
import { useLocalStorage } from '../../hooks/useLocalStorage';

describe('useLocalStorage', () => {
  beforeEach(() => {
    localStorage.clear();
  });

  test('should return initial value when localStorage is empty', () => {
    const { result } = renderHook(() => useLocalStorage('test-key', 'initial'));
    expect(result.current[0]).toBe('initial');
  });

  test('should update localStorage when value changes', () => {
    const { result } = renderHook(() => useLocalStorage('test-key', 'initial'));
    
    act(() => {
      result.current[1]('updated');
    });

    expect(result.current[0]).toBe('updated');
    expect(localStorage.getItem('test-key')).toBe('"updated"');
  });

  test('should load value from localStorage on init', () => {
    localStorage.setItem('test-key', '"stored-value"');
    
    const { result } = renderHook(() => useLocalStorage('test-key', 'initial'));
    expect(result.current[0]).toBe('stored-value');
  });
});
```

### 10.3 Testing Components with Context

```typescript
// __tests__/components/UserProfile.test.tsx
import React from 'react';
import { render, screen, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { AuthProvider } from '../../contexts/AuthContext';
import UserProfile from '../../components/UserProfile';

// Test utilities
const createTestQueryClient = () => new QueryClient({
  defaultOptions: {
    queries: { retry: false },
    mutations: { retry: false },
  },
});

const AllTheProviders: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const queryClient = createTestQueryClient();
  
  return (
    <QueryClientProvider client={queryClient}>
      <AuthProvider>
        {children}
      </AuthProvider>
    </QueryClientProvider>
  );
};

const renderWithProviders = (ui: React.ReactElement) => {
  return render(ui, { wrapper: AllTheProviders });
};

describe('UserProfile Component', () => {
  test('displays user information when authenticated', async () => {
    const mockUser = {
      id: 1,
      name: 'John Doe',
      email: 'john@example.com'
    };

    // Mock the auth context
    jest.spyOn(require('../../contexts/AuthContext'), 'useAuth')
      .mockReturnValue({
        user: mockUser,
        isAuthenticated: true,
        login: jest.fn(),
        logout: jest.fn()
      });

    renderWithProviders(<UserProfile />);

    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
  });

  test('shows login prompt when not authenticated', () => {
    jest.spyOn(require('../../contexts/AuthContext'), 'useAuth')
      .mockReturnValue({
        user: null,
        isAuthenticated: false,
        login: jest.fn(),
        logout: jest.fn()
      });

    renderWithProviders(<UserProfile />);

    expect(screen.getByText(/please log in/i)).toBeInTheDocument();
  });
});
```

## 11. Performance Optimization

### 11.1 React.memo và Optimization

```typescript
// components/OptimizedComponents.tsx
import React, { memo, useMemo, useCallback } from 'react';

interface ExpensiveListItemProps {
  item: {
    id: number;
    title: string;
    data: number[];
  };
  onUpdate: (id: number, newTitle: string) => void;
  isSelected: boolean;
}

// Memoized component - chỉ re-render khi props thay đổi
const ExpensiveListItem = memo<ExpensiveListItemProps>(({ 
  item, 
  onUpdate, 
  isSelected 
}) => {
  console.log(`Rendering item ${item.id}`);
  
  // Expensive calculation được memoize
  const processedData = useMemo(() => {
    console.log(`Processing data for item ${item.id}`);
    return item.data.reduce((sum, num) => sum + num * 2, 0);
  }, [item.data]);

  // Callback được memoize để tránh re-render con
  const handleClick = useCallback(() => {
    onUpdate(item.id, `Updated ${item.title}`);
  }, [item.id, item.title, onUpdate]);

  return (
    <div className={`p-4 ${isSelected ? 'bg-blue-100' : ''}`}>
      <h3>{item.title}</h3>
      <p>Processed: {processedData}</p>
      <button onClick={handleClick}>Update</button>
    </div>
  );
}, (prevProps, nextProps) => {
  // Custom comparison function
  return (
    prevProps.item.id === nextProps.item.id &&
    prevProps.item.title === nextProps.item.title &&
    prevProps.isSelected === nextProps.isSelected &&
    JSON.stringify(prevProps.item.data) === JSON.stringify(nextProps.item.data)
  );
});

// Parent component với optimization
const OptimizedList: React.FC = () => {
  const [items, setItems] = useState(/* large data */);
  const [selectedId, setSelectedId] = useState<number | null>(null);
  const [filter, setFilter] = useState('');

  // Memoized filtered data
  const filteredItems = useMemo(() => {
    return items.filter(item => 
      item.title.toLowerCase().includes(filter.toLowerCase())
    );
  }, [items, filter]);

  // Memoized callback
  const handleUpdateItem = useCallback((id: number, newTitle: string) => {
    setItems(prev => prev.map(item => 
      item.id === id ? { ...item, title: newTitle } : item
    ));
  }, []);

  return (
    <div>
      <input
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
        placeholder="Filter items..."
      />
      
      {filteredItems.map(item => (
        <ExpensiveListItem
          key={item.id}
          item={item}
          onUpdate={handleUpdateItem}
          isSelected={selectedId === item.id}
        />
      ))}
    </div>
  );
};
```

### 11.2 Code Splitting và Lazy Loading

```typescript
// Lazy loading components
import React, { Suspense, lazy } from 'react';
import { Routes, Route } from 'react-router-dom';
import LoadingSpinner from './components/LoadingSpinner';

// Lazy import components
const Home = lazy(() => import('./pages/Home'));
const Products = lazy(() => import('./pages/Products'));
const ProductDetail = lazy(() => import('./pages/ProductDetail'));
const UserDashboard = lazy(() => 
  import('./pages/UserDashboard').then(module => ({
    default: module.UserDashboard
  }))
);

const App: React.FC = () => {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/products" element={<Products />} />
        <Route path="/products/:id" element={<ProductDetail />} />
        <Route path="/dashboard" element={<UserDashboard />} />
      </Routes>
    </Suspense>
  );
};

// Dynamic imports trong components
const DynamicModal: React.FC = () => {
  const [showModal, setShowModal] = useState(false);
  const [ModalComponent, setModalComponent] = useState<React.ComponentType | null>(null);

  const loadModal = async () => {
    if (!ModalComponent) {
      const module = await import('./components/HeavyModal');
      setModalComponent(() => module.default);
    }
    setShowModal(true);
  };

  return (
    <div>
      <button onClick={loadModal}>Open Modal</button>
      {showModal && ModalComponent && (
        <ModalComponent onClose={() => setShowModal(false)} />
      )}
    </div>
  );
};
```

### 11.3 Virtual Scrolling

```typescript
// components/VirtualizedList.tsx
import React, { useMemo } from 'react';
import { FixedSizeList as List } from 'react-window';

interface VirtualizedListProps {
  items: any[];
  itemHeight: number;
  height: number;
  renderItem: (item: any, index: number) => React.ReactNode;
}

const VirtualizedList: React.FC<VirtualizedListProps> = ({
  items,
  itemHeight,
  height,
  renderItem
}) => {
  const Row = useMemo(() => 
    ({ index, style }: { index: number; style: React.CSSProperties }) => (
      <div style={style}>
        {renderItem(items[index], index)}
      </div>
    ), [items, renderItem]
  );

  return (
    <List
      height={height}
      itemCount={items.length}
      itemSize={itemHeight}
      overscanCount={5}
    >
      {Row}
    </List>
  );
};

// Usage
const ProductList: React.FC = () => {
  const { data: products = [] } = useProducts();

  const renderProduct = useCallback((product: Product, index: number) => (
    <div className="p-4 border-b">
      <h3>{product.title}</h3>
      <p>${product.price}</p>
    </div>
  ), []);

  return (
    <VirtualizedList
      items={products}
      itemHeight={80}
      height={600}
      renderItem={renderProduct}
    />
  );
};
```

## 12. Dự án thực tế

### 12.1 E-commerce App Structure

```typescript
// types/ecommerce.ts
export interface Product {
  id: number;
  title: string;
  price: number;
  description: string;
  category: string;
  image: string;
  rating: {
    rate: number;
    count: number;
  };
  stock: number;
}

export interface CartItem {
  product: Product;
  quantity: number;
}

export interface Order {
  id: string;
  items: CartItem[];
  total: number;
  status: 'pending' | 'processing' | 'shipped' | 'delivered';
  createdAt: string;
  shippingAddress: Address;
}

export interface Address {
  street: string;
  city: string;
  state: string;
  zipCode: string;
  country: string;
}
```
