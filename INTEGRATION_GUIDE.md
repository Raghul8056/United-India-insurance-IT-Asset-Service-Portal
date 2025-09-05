# IT Support Portal - Integration Guide

## Overview
This guide explains how to integrate the IT Support Portal components into your existing web application.

## Core Components

### 1. Employee Form (`client/src/components/employee-form.tsx`)
- **Purpose**: Allows employees to submit IT support tickets
- **Dependencies**: React Hook Form, Zod validation, TanStack Query
- **Required Props**: None (self-contained)

### 2. Vendor Form (`client/src/components/vendor-form.tsx`)
- **Purpose**: Allows vendors to submit IT support tickets
- **Dependencies**: React Hook Form, Zod validation, TanStack Query
- **Required Props**: None (self-contained)

### 3. Admin Panel (`client/src/components/admin-panel.tsx`)
- **Purpose**: View and manage all submitted tickets
- **Dependencies**: TanStack Query, date-fns for formatting
- **Required Props**: None (self-contained)

### 4. Admin Login (`client/src/components/admin-login.tsx`)
- **Purpose**: Authentication for admin access
- **Dependencies**: React Hook Form, Zod validation
- **Required Props**: 
  - `onLoginSuccess: () => void` - Callback when login succeeds
  - `onBack: () => void` - Callback for back button

## Integration Options

### Option 1: As Separate Pages in Your Router
```tsx
// In your main router component
import { Routes, Route } from 'react-router-dom';
import EmployeePortal from './components/employee-form';
import VendorPortal from './components/vendor-form';
import AdminPortal from './components/admin-panel';

function AppRoutes() {
  return (
    <Routes>
      <Route path="/support/employee" element={<EmployeePortal />} />
      <Route path="/support/vendor" element={<VendorPortal />} />
      <Route path="/support/admin" element={<AdminPortal />} />
    </Routes>
  );
}
```

### Option 2: As Modal/Overlay Components
```tsx
// In your existing component
import { useState } from 'react';
import { Dialog, DialogContent } from '@/components/ui/dialog';
import EmployeeForm from './components/employee-form';

function YourExistingComponent() {
  const [showSupportForm, setShowSupportForm] = useState(false);

  return (
    <div>
      <button onClick={() => setShowSupportForm(true)}>
        Get IT Support
      </button>
      
      <Dialog open={showSupportForm} onOpenChange={setShowSupportForm}>
        <DialogContent className="max-w-4xl">
          <EmployeeForm />
        </DialogContent>
      </Dialog>
    </div>
  );
}
```

### Option 3: Embedded in Your Dashboard
```tsx
// In your dashboard component
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import EmployeeForm from './components/employee-form';
import VendorForm from './components/vendor-form';

function Dashboard() {
  return (
    <div className="dashboard-container">
      {/* Your existing dashboard content */}
      
      <section className="support-section">
        <h2>IT Support</h2>
        <Tabs defaultValue="employee">
          <TabsList>
            <TabsTrigger value="employee">Employee</TabsTrigger>
            <TabsTrigger value="vendor">Vendor</TabsTrigger>
          </TabsList>
          
          <TabsContent value="employee">
            <EmployeeForm />
          </TabsContent>
          
          <TabsContent value="vendor">
            <VendorForm />
          </TabsContent>
        </Tabs>
      </section>
    </div>
  );
}
```

## Required Dependencies

### Frontend Dependencies
Install these packages in your existing project:
```bash
npm install @tanstack/react-query react-hook-form @hookform/resolvers zod date-fns
```

### UI Components (if not using shadcn/ui)
If your project doesn't use shadcn/ui, you'll need to either:
1. Install shadcn/ui and copy the required components
2. Replace the UI components with your existing component library
3. Create custom components matching the interface

Required shadcn/ui components:
- Button, Card, Input, Select, Textarea
- Form components (Form, FormField, FormItem, etc.)
- Tabs, Dialog, Badge, Table

## Backend Integration

### Database Schema
The ticket system requires this database structure:
```sql
CREATE TABLE tickets (
  id VARCHAR PRIMARY KEY DEFAULT gen_random_uuid(),
  ticket_id TEXT UNIQUE NOT NULL,
  user_type TEXT NOT NULL, -- 'employee' or 'vendor'
  
  -- Employee fields
  emp_name TEXT,
  department TEXT,
  
  -- Vendor fields  
  vendor_name TEXT,
  
  -- Common fields
  floor_no TEXT NOT NULL,
  mobile TEXT NOT NULL,
  designation TEXT NOT NULL,
  severity TEXT NOT NULL, -- 'low', 'medium', 'high', 'critical'
  issue_type TEXT NOT NULL, -- 'system', 'printer', 'others'
  system_subtype TEXT,
  printer_subtype TEXT,
  others_description TEXT,
  
  status TEXT NOT NULL DEFAULT 'open',
  created_at TIMESTAMP DEFAULT NOW()
);
```

### API Endpoints
Your backend needs these endpoints:

```typescript
// GET /api/tickets - Get all tickets
// POST /api/tickets - Create new ticket
// PATCH /api/tickets/:id/status - Update ticket status
```

Example Express.js implementation:
```javascript
// Get all tickets
app.get('/api/tickets', async (req, res) => {
  const tickets = await db.query('SELECT * FROM tickets ORDER BY created_at DESC');
  res.json(tickets.rows);
});

// Create ticket
app.post('/api/tickets', async (req, res) => {
  const ticketData = req.body;
  const ticketId = `TKT-${new Date().getFullYear()}-${String(counter).padStart(3, '0')}`;
  
  const result = await db.query(
    'INSERT INTO tickets (...columns) VALUES (...values) RETURNING *',
    [...values]
  );
  
  res.status(201).json(result.rows[0]);
});

// Update status
app.patch('/api/tickets/:id/status', async (req, res) => {
  const { id } = req.params;
  const { status } = req.body;
  
  const result = await db.query(
    'UPDATE tickets SET status = $1 WHERE id = $2 RETURNING *',
    [status, id]
  );
  
  res.json(result.rows[0]);
});
```

## Authentication Integration

### Simple Implementation (Current)
```tsx
// Simple localStorage-based auth
const isAdmin = localStorage.getItem('adminAuthenticated') === 'true';
```

### Production Implementation Options

1. **JWT Token Authentication**:
```tsx
const [isAdmin, setIsAdmin] = useState(false);

useEffect(() => {
  const token = localStorage.getItem('adminToken');
  if (token) {
    // Verify token with your backend
    verifyAdminToken(token).then(setIsAdmin);
  }
}, []);
```

2. **Integration with Existing Auth System**:
```tsx
// If you already have user roles in your app
const { user } = useAuth(); // Your existing auth hook
const isAdmin = user?.role === 'admin' || user?.permissions?.includes('tickets:manage');
```

3. **Session-based Authentication**:
```tsx
const checkAdminSession = async () => {
  const response = await fetch('/api/auth/check-admin');
  return response.ok;
};
```

## Customization

### Styling
- All components use Tailwind CSS classes
- Customize colors by modifying CSS variables in your main CSS file
- Override component styles by adding more specific CSS classes

### Form Fields
To add/remove form fields:
1. Update the Zod schema in the form component
2. Update the database schema
3. Update the backend API to handle new fields
4. Add/remove form fields in the JSX

### Validation Rules
Modify validation in the Zod schemas:
```tsx
const employeeFormSchema = z.object({
  empName: z.string().min(2, "Name must be at least 2 characters"),
  mobile: z.string().regex(/^\d{10}$/, "Must be 10 digits"), // Custom validation
  // ... other fields
});
```

## Environment Variables
Set these in your application:
```env
# Backend API URL (if different from current domain)
VITE_API_URL=https://your-api-domain.com

# Admin credentials (for simple auth - not recommended for production)
VITE_ADMIN_USERNAME=admin
VITE_ADMIN_PASSWORD=your-secure-password
```

## Security Considerations

1. **Admin Authentication**: Replace simple localStorage auth with proper JWT tokens
2. **API Security**: Add authentication middleware to admin-only endpoints
3. **Input Validation**: Server-side validation for all form inputs
4. **Rate Limiting**: Prevent spam ticket creation
5. **CSRF Protection**: Implement CSRF tokens for form submissions

## Testing

### Component Testing
```javascript
// Example test for employee form
import { render, screen, fireEvent } from '@testing-library/react';
import EmployeeForm from './employee-form';

test('submits employee ticket', async () => {
  render(<EmployeeForm />);
  
  fireEvent.change(screen.getByLabelText('Employee Name'), {
    target: { value: 'John Doe' }
  });
  
  fireEvent.click(screen.getByText('Submit Ticket'));
  
  // Assert success behavior
});
```

## Deployment Checklist

- [ ] Database schema created
- [ ] API endpoints implemented and tested
- [ ] Frontend components integrated
- [ ] Authentication system implemented
- [ ] Environment variables configured
- [ ] Input validation added (client and server)
- [ ] Error handling implemented
- [ ] Security measures in place
- [ ] Components tested
- [ ] Admin credentials secured

## Support

For questions about integration:
1. Check the component props and interfaces
2. Review the API endpoint requirements
3. Test with the provided admin credentials first
4. Verify database schema matches expectations