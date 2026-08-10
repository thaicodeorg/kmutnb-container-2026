!!! tip "Slides Available"
    📊 **View Presentation** → [Open Slides](../presents/09-workshop-fullstack.html)

# Chapter 9: Workshop - Fullstack Application

## Overview

This workshop covers building a fullstack application with React+TypeScript frontend, FastAPI backend, PostgreSQL database, and Nginx reverse proxy.

---

## 9.1 Project Structure

```
fullstack-app/
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── tsconfig.json
│   ├── src/
│   │   ├── App.tsx
│   │   ├── main.tsx
│   │   └── components/
│   │       └── TodoList.tsx
│   └── nginx.conf
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── main.py
├── database/
│   └── init.sql
├── nginx/
│   └── nginx.conf
└── docker-compose.yml
```

---

## 9.2 Docker Compose Configuration

### docker-compose.yml

```yaml
version: "3.8"

services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "3000:80"
    depends_on:
      - backend
    networks:
      - app-network

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://postgres:secret@database:5432/todos
      REDIS_URL: redis://redis:6379
    depends_on:
      database:
        condition: service_healthy
    networks:
      - app-network

  database:
    image: postgres:15
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: todos
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    networks:
      - app-network

  nginx:
    image: nginx:latest
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - frontend
      - backend
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  pgdata:
```

---

## 9.3 Frontend: React + TypeScript

### frontend/Dockerfile

```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json .
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### frontend/package.json

```json
{
  "name": "todo-frontend",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "axios": "^1.5.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "@vitejs/plugin-react": "^4.0.0",
    "typescript": "^5.2.0",
    "vite": "^4.4.0"
  }
}
```

### frontend/src/App.tsx

```tsx
import { useState, useEffect } from 'react';
import TodoList from './components/TodoList';

interface Todo {
  id: number;
  title: string;
  completed: boolean;
}

function App() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [newTodo, setNewTodo] = useState('');

  useEffect(() => {
    fetchTodos();
  }, []);

  const fetchTodos = async () => {
    const response = await fetch('/api/todos');
    const data = await response.json();
    setTodos(data);
  };

  const addTodo = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!newTodo.trim()) return;

    await fetch('/api/todos', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ title: newTodo })
    });

    setNewTodo('');
    fetchTodos();
  };

  const toggleTodo = async (id: number) => {
    await fetch(`/api/todos/${id}`, { method: 'PUT' });
    fetchTodos();
  };

  const deleteTodo = async (id: number) => {
    await fetch(`/api/todos/${id}`, { method: 'DELETE' });
    fetchTodos();
  };

  return (
    <div className="app">
      <h1>Todo App</h1>
      <form onSubmit={addTodo}>
        <input
          type="text"
          value={newTodo}
          onChange={(e) => setNewTodo(e.target.value)}
          placeholder="Add a todo..."
        />
        <button type="submit">Add</button>
      </form>
      <TodoList
        todos={todos}
        onToggle={toggleTodo}
        onDelete={deleteTodo}
      />
    </div>
  );
}

export default App;
```

### frontend/src/components/TodoList.tsx

```tsx
interface Todo {
  id: number;
  title: string;
  completed: boolean;
}

interface TodoListProps {
  todos: Todo[];
  onToggle: (id: number) => void;
  onDelete: (id: number) => void;
}

function TodoList({ todos, onToggle, onDelete }: TodoListProps) {
  return (
    <ul className="todo-list">
      {todos.map(todo => (
        <li key={todo.id} className={todo.completed ? 'completed' : ''}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => onToggle(todo.id)}
          />
          <span>{todo.title}</span>
          <button onClick={() => onDelete(todo.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}

export default TodoList;
```

### frontend/nginx.conf

```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api {
        proxy_pass http://backend:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

---

## 9.4 Backend: FastAPI

### backend/Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### backend/requirements.txt

```
fastapi==0.104.1
uvicorn[standard]==0.24.0
sqlalchemy==2.0.23
psycopg2-binary==2.9.9
pydantic==2.5.2
python-dotenv==1.0.0
```

### backend/main.py

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from sqlalchemy import create_engine, Column, Integer, String, Boolean
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://postgres:secret@localhost:5432/todos")

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

class TodoDB(Base):
    __tablename__ = "todos"
    id = Column(Integer, primary_key=True, index=True)
    title = Column(String, index=True)
    completed = Column(Boolean, default=False)

Base.metadata.create_all(bind=engine)

app = FastAPI()

class TodoCreate(BaseModel):
    title: str

class Todo(BaseModel):
    id: int
    title: str
    completed: bool

    class Config:
        from_attributes = True

@app.get("/api/todos")
def read_todos():
    db = SessionLocal()
    try:
        todos = db.query(TodoDB).all()
        return [Todo(id=t.id, title=t.title, completed=t.completed) for t in todos]
    finally:
        db.close()

@app.post("/api/todos")
def create_todo(todo: TodoCreate):
    db = SessionLocal()
    try:
        db_todo = TodoDB(title=todo.title)
        db.add(db_todo)
        db.commit()
        db.refresh(db_todo)
        return Todo(id=db_todo.id, title=db_todo.title, completed=db_todo.completed)
    finally:
        db.close()

@app.put("/api/todos/{todo_id}")
def toggle_todo(todo_id: int):
    db = SessionLocal()
    try:
        todo = db.query(TodoDB).filter(TodoDB.id == todo_id).first()
        if not todo:
            raise HTTPException(status_code=404, detail="Todo not found")
        todo.completed = not todo.completed
        db.commit()
        return Todo(id=todo.id, title=todo.title, completed=todo.completed)
    finally:
        db.close()

@app.delete("/api/todos/{todo_id}")
def delete_todo(todo_id: int):
    db = SessionLocal()
    try:
        todo = db.query(TodoDB).filter(TodoDB.id == todo_id).first()
        if not todo:
            raise HTTPException(status_code=404, detail="Todo not found")
        db.delete(todo)
        db.commit()
        return {"message": "Todo deleted"}
    finally:
        db.close()
```

---

## 9.5 Database Configuration

### database/init.sql

```sql
-- Initialize the todos table
CREATE TABLE IF NOT EXISTS todos (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    completed BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert sample data
INSERT INTO todos (title, completed) VALUES
    ('Learn Docker', false),
    ('Build fullstack app', false),
    ('Deploy to production', false);
```

---

## 9.6 Nginx Reverse Proxy

### nginx/nginx.conf

```nginx
events {
    worker_connections 1024;
}

http {
    upstream frontend {
        server frontend:80;
    }

    upstream backend {
        server backend:8000;
    }

    server {
        listen 80;
        server_name localhost;

        location / {
            proxy_pass http://frontend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        location /api {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

---

## 9.7 Running the Application

### Step 1: Clone and Setup

```bash
cd ~/fullstack-app
```

### Step 2: Build and Start

```bash
docker compose up --build -d
```

### Step 3: Verify Services

```bash
docker compose ps
docker compose logs
```

### Step 4: Access the Application

```bash
# Frontend
curl http://localhost

# Backend API
curl http://localhost/api/todos

# Create a todo
curl -X POST http://localhost/api/todos \
    -H "Content-Type: application/json" \
    -d '{"title": "New todo"}'
```

### Step 5: View Database

```bash
docker compose exec database psql -U postgres -d todos
```

### Step 6: Stop Application

```bash
docker compose down
```

### Step 7: Stop and Remove Volumes

```bash
docker compose down -v
```

---

## Summary

| Component | Technology | Port |
|-----------|------------|------|
| Frontend | React + TypeScript | 3000 |
| Backend | FastAPI + Python | 8000 |
| Database | PostgreSQL 15 | 5432 |
| Proxy | Nginx | 80 |

---

## Quiz

??? question "Question 1: What is the purpose of using Docker Compose for this application?"
    **Answer:**
    
    To define and manage multiple services (frontend, backend, database, nginx) as a single application

??? question "Question 2: What database is used in this fullstack application?"
    **Answer:**
    
    PostgreSQL 15 — configured with a health check for reliable startup order

??? question "Question 3: How does the frontend communicate with the backend?"
    **Answer:**
    
    Through Nginx reverse proxy, which routes `/api` requests to the backend service

??? question "Question 4: What Python web framework is used for the backend?"
    **Answer:**
    
    FastAPI — a modern, fast web framework for building APIs with Python

??? question "Question 5: What is the purpose of the `depends_on` directive?"
    **Answer:**
    
    Defines service startup order — frontend waits for backend, backend waits for database

??? question "Question 6: How do you build and start all services?"
    **Answer:**
    
    `docker compose up --build -d` — builds images and starts in detached mode

??? question "Question 7: What does `docker compose down -v` do?"
    **Answer:**
    
    Stops and removes containers, networks, AND volumes (deletes database data)

??? question "Question 8: How do you access the PostgreSQL database interactively?"
    **Answer:**
    
    `docker compose exec database psql -U postgres -d todos`

??? question "Question 9: What is the role of Nginx in this architecture?"
    **Answer:**
    
    Acts as a reverse proxy, routing requests to frontend and backend services

??? question "Question 10: How do you view logs from all services?"
    **Answer:**
    
    `docker compose logs` — shows combined logs; use `docker compose logs <service>` for specific service
