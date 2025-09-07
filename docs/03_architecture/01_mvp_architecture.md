---
title: "Time2Work MVP Architecture"
description: "Simple, focused architecture for 1-2 week MVP"
version: "1.0"
date: "2025-09-07"
type: "MVP Architecture"
audience: "Development Team"
document_number: "01"
input: "../02_product/04_requirements_specification.md"
output: "MVP implementation plan"
---

# Time2Work MVP Architecture

## Overview

This document describes a minimal viable product (MVP) architecture for Time2Work that can be built in 1-2 weeks. Focus is on Phase 1 compliance requirements only.

## Tech Stack

### Backend
- **Framework**: FastAPI (Python 3.11+)
- **Database**: PostgreSQL 15
- **ORM**: SQLAlchemy 2.0
- **Validation**: Pydantic v2
- **Testing**: pytest

### Frontend
- **Framework**: React Router 7
- **UI Components**: shadcn/ui (Radix UI + Tailwind CSS)
- **Styling**: Tailwind CSS
- **Build Tool**: Vite
- **Testing**: Vitest
- **Icons**: Lucide React

### Infrastructure
- **Development**: Docker Compose
- **Production**: Single VPS with Docker

## Project Structure

```
time-2-work/
├── backend/
│   ├── app/
│   │   ├── main.py              # FastAPI app
│   │   ├── models.py            # SQLAlchemy models
│   │   ├── schemas.py           # Pydantic schemas
│   │   ├── crud.py              # Database operations
│   │   ├── api/
│   │   │   ├── employees.py     # Employee endpoints
│   │   │   ├── time_entries.py  # Time tracking
│   │   │   └── leave.py         # Leave management
│   │   ├── core/
│   │   │   ├── config.py        # Settings
│   │   │   └── compliance.py    # Polish law calculations
│   │   └── db.py                # Database setup
│   ├── tests/
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   ├── app/
│   │   ├── root.tsx
│   │   ├── routes/
│   │   │   ├── index.tsx        # Dashboard
│   │   │   ├── time-entry.tsx   # Clock in/out
│   │   │   ├── timesheet.tsx    # View entries
│   │   │   └── leave.tsx        # Request leave
│   │   ├── components/
│   │   │   ├── ui/              # shadcn/ui components
│   │   │   └── time-clock.tsx   # Custom components
│   │   ├── lib/
│   │   │   └── utils.ts         # cn() helper
│   ├── package.json
│   ├── components.json          # shadcn/ui config
│   ├── tailwind.config.js
│   └── Dockerfile
├── docker-compose.yml
└── .env.example
```

## Core Features (MVP Only)

### Employee Features
1. View profile and leave balance
2. Clock in/out
3. View own time entries
4. Request leave

### Manager Features
1. View team time entries
2. Approve/reject time entries
3. Approve/reject leave requests
4. Basic compliance dashboard

### Compliance Features
1. Automatic overtime calculation (50%/100%)
2. Leave entitlement calculation (20/26 days)
3. Rest period validation (11h daily, 35h weekly)
4. Block overtime for special statuses

## Authentication Strategy

- **MVP**: No authentication implementation
- **Future**: SuperTokens integration
- **Mapping**: `employees.supertokens_user_id` → SuperTokens User ID
- **Roles**: Managed by SuperTokens (employee/manager)

For MVP testing:
- Use hardcoded employee ID in API calls
- Add `X-Employee-ID` header to simulate different users
- No login screen needed

## Database Design Philosophy

- Simple normalized tables
- Compliance fields embedded in main tables
- Calculated fields stored for performance
- No event sourcing or complex patterns

## API Design

- RESTful endpoints
- JSON request/response
- Pydantic for validation
- FastAPI automatic OpenAPI docs

Example:
```python
@app.post("/time-entries/clock-in")
async def clock_in(employee_id: int = Header(alias="X-Employee-ID")):
    # Clock in logic
    return {"status": "clocked_in", "timestamp": datetime.now()}
```

## Frontend Setup

### Initialize Project

```bash
# Create React Router 7 app
pnpm create vite@latest frontend -- --template react-ts
cd frontend
pnpm install react-router

# Install Tailwind CSS
pnpm install -D tailwindcss postcss autoprefixer
pnpm dlx tailwindcss init -p

# Install shadcn/ui
pnpm install tailwind-merge clsx
pnpm install lucide-react
pnpm dlx shadcn@latest init
```

### Add shadcn/ui Components

```bash
# Add components as needed
pnpm dlx shadcn@latest add button
pnpm dlx shadcn@latest add card
pnpm dlx shadcn@latest add form
pnpm dlx shadcn@latest add input
pnpm dlx shadcn@latest add label
pnpm dlx shadcn@latest add toast
pnpm dlx shadcn@latest add calendar
pnpm dlx shadcn@latest add select
```

### Example Components

```tsx
// app/routes/time-entry.tsx
import { Form } from "react-router";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Clock } from "lucide-react";

export async function action({ request }) {
  const formData = await request.formData();
  const action = formData.get("action");
  
  const response = await fetch(`/api/time-entries/${action}`, {
    method: "POST",
    headers: { "X-Employee-ID": "1" }
  });
  
  return response.json();
}

export default function TimeEntry() {
  return (
    <div className="container mx-auto p-4">
      <Card className="max-w-md mx-auto">
        <CardHeader>
          <CardTitle className="flex items-center gap-2">
            <Clock className="h-5 w-5" />
            Time Clock
          </CardTitle>
        </CardHeader>
        <CardContent className="space-y-4">
          <Form method="post">
            <input type="hidden" name="action" value="clock-in" />
            <Button type="submit" className="w-full" size="lg">
              Clock In
            </Button>
          </Form>
          
          <Form method="post">
            <input type="hidden" name="action" value="clock-out" />
            <Button 
              type="submit" 
              variant="secondary" 
              className="w-full" 
              size="lg"
            >
              Clock Out
            </Button>
          </Form>
        </CardContent>
      </Card>
    </div>
  );
}
```

## Development Workflow

1. **Backend First**
   - Set up database schema
   - Implement compliance calculations
   - Create API endpoints
   - Write tests for compliance rules

2. **Frontend Second**
   - Set up React Router 7 with Vite
   - Initialize shadcn/ui and Tailwind
   - Add necessary UI components
   - Build forms with React Router 7 actions
   - Connect to FastAPI backend

3. **Integration**
   - Docker Compose setup
   - End-to-end testing
   - Deploy to single VPS

## Deployment (Week 2)

```yaml
# docker-compose.yml
version: '3.8'
services:
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: time2work
      POSTGRES_USER: time2work
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data

  backend:
    build: ./backend
    depends_on:
      - db
    environment:
      DATABASE_URL: postgresql://time2work:${DB_PASSWORD}@db/time2work
    ports:
      - "8000:8000"

  frontend:
    build: ./frontend
    depends_on:
      - backend
    ports:
      - "3000:3000"

volumes:
  postgres_data:
```

## MVP Scope Boundaries

### What's IN:
- Basic time tracking
- Overtime calculation
- Leave balance tracking
- Manager approvals
- Polish labor law compliance for `employment_contract` only

### What's OUT:
- Authentication (SuperTokens later)
- Other contract types
- Complex work systems
- Mobile app
- Reporting beyond basic views
- Data corrections/audit trail
- Multiple departments/companies

## Success Criteria

1. Employee can clock in/out
2. System calculates overtime correctly
3. Manager can approve entries
4. Validates rest periods
5. Tracks leave balance
6. Runs in Docker
7. Has basic tests

## Timeline

**Week 1:**
- Day 1-2: Database schema and models
- Day 3-4: API endpoints and compliance logic
- Day 5: Frontend routes and forms

**Week 2:**
- Day 1-2: Integration and testing
- Day 3-4: Docker setup and deployment
- Day 5: Buffer and documentation

This architecture prioritizes simplicity and speed while maintaining Polish labor law compliance for the MVP.