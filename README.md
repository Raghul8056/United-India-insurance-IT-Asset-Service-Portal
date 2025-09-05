<<<<<<< HEAD
# IT Support Portal

A comprehensive AMC (Annual Maintenance Contract) hardware issue ticket raising portal with separate employee and vendor portals, plus a secure admin panel for ticket management.

## Features

- **Employee Portal** - Submit IT support tickets with department tracking
- **Vendor Portal** - External vendor support request system
- **Admin Panel** - Secure ticket management and status tracking
- **Indian Mobile Validation** - Supports +91 format with proper validation
- **Comprehensive Issue Tracking** - System, Printer, and Other issue categories
- **Real-time Updates** - Instant ticket creation and status updates
- **Responsive Design** - Works on desktop, tablet, and mobile devices

## Screenshots

### Employee Portal
- Employee name, department, and floor tracking
- Designation levels: Assistant, Sr. Assistant, Scale 1-7
- Severity levels: Low, Medium, High, Critical
- Dynamic issue sub-types for System and Printer issues

### Admin Panel
- Secure login (admin/admin123)
- View all tickets with filtering
- Update ticket status (Open, In Progress, Resolved, Closed)
- Ticket analytics and reporting

## Quick Start

### Prerequisites

- **Node.js** (v18.0.0 or higher)
- **npm** (v9.0.0 or higher)

### Installation

1. **Download the project**
   ```bash
   # Extract the downloaded ZIP file
   unzip it-support-portal.zip
   cd it-support-portal
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Start the development server**
   ```bash
   PORT=3000 npm run dev
   ```

4. **Open your browser**
   ```
   http://localhost:3000
   ```

## Platform-Specific Setup

### Windows

1. **Install Node.js**
   - Download from [nodejs.org](https://nodejs.org)
   - Choose the LTS version
   - Run the installer with default settings

2. **Open Command Prompt or PowerShell**
   ```cmd
   cd path\to\it-support-portal
   npm install
   npm run dev
   ```

### macOS

1. **Install Node.js**
   ```bash
   # Using Homebrew (recommended)
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   brew install node
   
   # Or download from nodejs.org
   ```

2. **Run the application**
   ```bash
   cd path/to/it-support-portal
   npm install
   npm run dev
   ```

### Linux (Ubuntu/Debian)

1. **Install Node.js**
   ```bash
   # Using NodeSource repository
   curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
   sudo apt-get install -y nodejs
   
   # Or using package manager
   sudo apt update
   sudo apt install nodejs npm
   ```

2. **Run the application**
   ```bash
   cd path/to/it-support-portal
   npm install
   npm run dev
   ```

### Linux (CentOS/RHEL/Fedora)

1. **Install Node.js**
   ```bash
   # CentOS/RHEL
   sudo yum install nodejs npm
   
   # Fedora
   sudo dnf install nodejs npm
   ```

2. **Run the application**
   ```bash
   cd path/to/it-support-portal
   npm install
   npm run dev
   ```

## Available Commands

```bash
# Start development server with hot reload
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Type checking
npm run type-check
```

## Usage

### Employee Ticket Submission

1. Navigate to **Employee Portal** tab
2. Fill out the required information:
   - Employee Name
   - Floor Number (1-14)
   - Department
   - Mobile Number (+91 XXXXXXXXXX format)
   - Designation
   - Severity Level
   - Issue Type and Sub-type
3. Click **Submit Ticket**
4. Note the generated Ticket ID for tracking

### Vendor Ticket Submission

1. Navigate to **Vendor Portal** tab
2. Fill out vendor-specific information:
   - Vendor Name
   - Mobile Number (+91 XXXXXXXXXX format)
   - Floor Number
   - Designation
   - Issue details
3. Submit ticket and receive Ticket ID

### Admin Panel Access

1. Navigate to **Admin Panel** tab
2. Click **Login as Admin**
3. Enter credentials:
   - **Username:** admin
   - **Password:** admin123
4. View and manage all tickets
5. Update ticket status as needed
6. Use **Back to Portal** to return to main interface

## Configuration

### Environment Variables

Create a `.env` file in the root directory for custom configuration:

```env
# Server Configuration
PORT=5000
HOST=127.0.0.1

# Database (for production)
DATABASE_URL=postgresql://username:password@localhost:5432/dbname

# Admin Credentials (change for production)
ADMIN_USERNAME=admin
ADMIN_PASSWORD=admin123
```

### Custom Port

To run on a different port:

```bash
PORT=3000 npm run dev
```

## Troubleshooting

### Common Issues

**1. Port Already in Use**
```bash
# Find and kill process using the port
# Windows
netstat -ano | findstr :5000
taskkill /PID [PID_NUMBER] /F

# macOS/Linux
lsof -ti:5000 | xargs kill -9
```

**2. Network Socket Error (Mac)**
```bash
# Try different port
PORT=3000 npm run dev

# Or use specific host
HOST=127.0.0.1 npm run dev
```

**3. Permission Issues (Linux/Mac)**
```bash
# Fix npm permissions
sudo chown -R $(whoami) ~/.npm

# Or use different directory
npm config set prefix ~/.local
```

**4. Node.js Version Issues**
```bash
# Check version
node --version

# Should be v18.0.0 or higher
# Update if necessary
```

### Error Messages

- **"Module not found"** → Run `npm install`
- **"Permission denied"** → Check file permissions or run with appropriate privileges
- **"EADDRINUSE"** → Port is busy, use different port or kill existing process
- **"ENOTSUP"** → Network configuration issue, try different host/port

## Project Structure

```
it-support-portal/
├── client/                 # Frontend React application
│   ├── src/
│   │   ├── components/     # React components
│   │   ├── pages/         # Page components
│   │   ├── hooks/         # Custom hooks
│   │   └── lib/           # Utility functions
│   └── index.html         # Main HTML template
├── server/                # Backend Express server
│   ├── index.ts          # Server entry point
│   ├── routes.ts         # API routes
│   └── storage.ts        # Data storage layer
├── shared/               # Shared code
│   └── schema.ts        # Type definitions
├── package.json         # Dependencies
├── README.md           # This file
└── Integration guides/ # Documentation
```

## Data Storage

### Development Mode (Default)
- Uses **in-memory storage**
- Perfect for testing and development
- Data resets when server restarts
- No database setup required

### Production Mode
- Supports PostgreSQL database
- Persistent data storage
- Requires database configuration
- See integration guides for setup

## Integration

### For Existing Web Applications

The portal can be integrated into existing applications using several approaches:

1. **Separate Pages** - Add as new routes in your application
2. **Modal/Overlay** - Embed as popup forms
3. **Dashboard Integration** - Include as dashboard sections

See `INTEGRATION_GUIDE.md` and `UIIC_INTEGRATION_GUIDE.md` for detailed instructions.

### API Endpoints

```
GET    /api/tickets           # Retrieve all tickets
POST   /api/tickets           # Create new ticket
PATCH  /api/tickets/:id/status # Update ticket status
```

## Security Notes

### Development vs Production

**Development (Current Setup):**
- Admin credentials: admin/admin123
- In-memory storage (data not persistent)
- No authentication required for ticket submission

**Production Recommendations:**
- Change admin credentials
- Use secure database with SSL
- Implement proper authentication
- Add rate limiting
- Enable HTTPS
- Add input validation middleware

## Deployment Options

### Local Development
```bash
npm run dev  # Development server with hot reload
```

### Production Build
```bash
npm run build    # Create optimized build
npm run preview  # Preview production build
```

### Cloud Deployment

- **Vercel** - Frontend + serverless functions
- **Railway** - Full-stack deployment
- **Heroku** - Traditional PaaS deployment
- **Digital Ocean** - VPS deployment
- **AWS/Azure/GCP** - Cloud platform deployment

## Mobile Number Format

The application validates Indian mobile numbers in the format:
```
+91 XXXXXXXXXX
```

- Must start with +91
- Space after country code
- 10-digit mobile number
- First digit must be 6, 7, 8, or 9

Examples:
- ✅ +91 9876543210
- ✅ +91 8123456789
- ❌ +919876543210 (no space)
- ❌ +91 5123456789 (invalid first digit)

## Support

### Documentation
- `INTEGRATION_GUIDE.md` - General integration instructions
- `UIIC_INTEGRATION_GUIDE.md` - Drupal-specific integration
- `MAC_SETUP_GUIDE.md` - Mac-specific setup instructions

### Common Tasks

**Add New Form Field:**
1. Update form schema in component file
2. Update TypeScript interfaces
3. Update database schema in `shared/schema.ts`
4. Update API handlers in `server/routes.ts`

**Customize Styling:**
1. Modify Tailwind classes in components
2. Update CSS variables in `client/src/index.css`
3. Adjust theme colors in `tailwind.config.ts`

**Change Admin Credentials:**
1. Update validation in `admin-login.tsx`
2. Set environment variables for production
3. Consider implementing proper authentication system

## License

This project is provided as-is for organizational use. Modify and deploy according to your requirements.

---

**Quick Help:**
- Can't start the server? Check if Node.js is installed and port 5000 is available
- Forms not submitting? Check browser console for errors and ensure server is running
- Admin login not working? Use credentials: admin/admin123
- Need to customize? See integration guides in the project folder
=======
I created the user interface design for the IT asset service portal
>>>>>>> 5d25649b0b542a230957bde823082e7032de7320
