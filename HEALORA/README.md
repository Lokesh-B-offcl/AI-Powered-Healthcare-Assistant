# Helora - Hospital Management System

A modern, full-stack healthcare management platform built with React, TypeScript, Tailwind CSS, and Supabase backend.

## 🎯 Project Overview

Helora is a comprehensive hospital management system designed for healthcare professionals and patients. It provides role-based portals for patients, doctors, and administrators with real-time appointment management, medical records tracking, and AI-powered health guidance.

## 🏗️ Architecture

### Frontend
- **Framework**: Next.js 16 with React 19
- **Language**: TypeScript (strict mode)
- **Styling**: Tailwind CSS v4 with custom theme
- **Animations**: Framer Motion
- **UI Components**: Radix UI + shadcn/ui
- **Icons**: Lucide React

### Backend
- **Database**: Supabase (PostgreSQL)
- **Authentication**: Supabase Auth
- **File Storage**: Supabase Storage
- **Real-time**: Supabase Realtime subscriptions

### Design System
- **Theme**: Dark healthcare aesthetic
  - Background: Deep Navy (#0a0f1a)
  - Primary: Teal/Cyan (hsl(172, 66%, 50%))
  - Accent: Amber (hsl(45, 93%, 58%))
- **Display Font**: Space Grotesk (bold, geometric, professional)
- **Body Font**: DM Sans (clean, readable)
- **Effects**: Glassmorphism cards, gradient text, backdrop blur

## 📄 Pages & Features

### 1. Landing Page `/`
- Hero section with Carl Jung healthcare quote
- 5 portal cards for quick access (Patient, Doctor, Admin, Reports, AI)
- Features grid (6 key benefits)
- Mission statement with statistics
- How-it-works step guide (4 steps)
- Footer with links and social media

### 2. Authentication `/auth`
- Sign up/Sign in toggle
- Email & password validation
- Role selection (Patient/Doctor/Admin)
- Doctor specialty selection
- Phone number collection
- Email verification (prepared for Supabase)
- Role-based redirect

### 3. Patient Portal `/patient` (Protected)
- Dashboard with stats: Member since, Total visits, Upcoming appointments, Active prescriptions
- Quick action cards: Book appointment, Medical records, Find doctors, AI consultation
- Upcoming appointments table with doctor details
- Active prescriptions list with dosage and frequency
- Appointment management (reschedule/cancel)

### 4. Doctor Portal `/doctor` (Protected)
- Dashboard with stats: Total patients, Today's appointments, Completed today, Pending reviews
- Today's appointment schedule (6 slots)
- Appointment status management (Start/Complete)
- Recent patients sidebar
- Quick patient access (View records, profile, history)

### 5. Admin Portal `/admin` (Protected)
- System KPIs: Total patients, doctors, appointments, completion rate, system health
- Patient management table with search
- Doctor management with ratings and status
- Recent appointments overview table
- Add new doctor functionality
- Patient and doctor profile views

### 6. View Reports `/view-reports`
- Patient search by phone number
- Visit history with 5 visits displayed
- Patient summary (name, phone, registration date, visit count)
- Visit details: doctor name, specialty, date, diagnosis, notes
- Export options (PDF, Excel, Email)

### 7. Find Doctors `/find-doctors`
- Search by name or specialty
- Filter by specialty (8 specialties)
- Doctor cards with ratings, experience, bio
- Clinic address and phone
- Availability status
- Book appointment link

### 8. Book Appointment `/book-appointment`
- 3-step wizard: Select doctor → Details → Confirm
- Progress indicator
- Doctor selection from dropdown
- Date & time selection
- Visit type: In-Person or Video
- Additional notes field
- Success confirmation screen

### 9. AI Consultation `/ai-consultation`
- Real-time chat interface
- Keyword-based health guidance bot
- Suggested questions for new users
- Message timestamps
- Typing indicator
- Safety disclaimer
- 10+ health topics supported (headache, fever, cold, flu, etc.)

### 10. Pharmacy `/pharmacy`
- Medicine catalog (8+ medicines)
- Search by name
- Filter by category (Pain Relief, Diabetes, BP, etc.)
- Medicine details: dosage, price, manufacturer
- Stock status indicator
- Add to cart functionality
- Shopping cart with quantity controls
- Cart summary with total
- Checkout button

## 🗄️ Database Schema (Supabase)

### Tables

#### `profiles`
- `id` (UUID, PK) - References auth.users
- `email` (TEXT, unique)
- `full_name` (TEXT)
- `phone_number` (TEXT)
- `date_of_birth` (DATE)
- `role` (ENUM: patient, doctor, admin)
- `avatar_url` (TEXT)
- `created_at` (TIMESTAMP)
- `updated_at` (TIMESTAMP)

#### `doctors`
- `id` (UUID, PK) - References profiles
- `specialty` (TEXT) - Cardiology, Dermatology, etc.
- `experience_years` (INTEGER)
- `rating` (DECIMAL 0-5)
- `bio` (TEXT)
- `clinic_address` (TEXT)
- `availability_status` (BOOLEAN)
- `created_at` (TIMESTAMP)
- `updated_at` (TIMESTAMP)

#### `appointments`
- `id` (UUID, PK)
- `patient_id` (UUID, FK → profiles)
- `doctor_id` (UUID, FK → doctors)
- `appointment_date` (DATE)
- `appointment_time` (TIME)
- `visit_type` (ENUM: In-Person, Video)
- `notes` (TEXT)
- `status` (ENUM: scheduled, completed, cancelled, confirmed)
- `created_at` (TIMESTAMP)
- `updated_at` (TIMESTAMP)

#### `prescriptions`
- `id` (UUID, PK)
- `patient_id` (UUID, FK → profiles)
- `doctor_id` (UUID, FK → doctors)
- `appointment_id` (UUID, FK → appointments)
- `medication_name` (TEXT)
- `dosage` (TEXT)
- `frequency` (TEXT)
- `duration` (TEXT)
- `notes` (TEXT)
- `prescribed_date` (TIMESTAMP)
- `created_at` (TIMESTAMP)
- `updated_at` (TIMESTAMP)

#### `medicines`
- `id` (UUID, PK)
- `name` (TEXT, unique)
- `category` (TEXT)
- `description` (TEXT)
- `price` (DECIMAL)
- `dosage_form` (TEXT)
- `manufacturer` (TEXT)
- `in_stock` (BOOLEAN)
- `quantity_available` (INTEGER)
- `created_at` (TIMESTAMP)
- `updated_at` (TIMESTAMP)

#### `visit_history`
- `id` (UUID, PK)
- `patient_id` (UUID, FK → profiles)
- `doctor_id` (UUID, FK → doctors)
- `appointment_id` (UUID, FK → appointments)
- `diagnosis` (TEXT)
- `treatment_notes` (TEXT)
- `visit_date` (TIMESTAMP)
- `created_at` (TIMESTAMP)
- `updated_at` (TIMESTAMP)

### Row Level Security (RLS) Policies

**Profiles**
- Users can view their own profile
- Doctors can view patient profiles for appointments
- Admins can view all profiles

**Doctors**
- Anyone can view doctor profiles
- Doctors can update their own profile
- Admins can manage all doctor profiles

**Appointments**
- Patients see their own appointments
- Doctors see their appointments
- Admins see all appointments

**Prescriptions**
- Patients see their own prescriptions
- Doctors see their prescriptions
- (Admins can view all)

**Medicines**
- Public read access
- Admin-only modifications

**Visit History**
- Patients see their own history
- Doctors see their patient history

## 🚀 Getting Started

### Prerequisites
- Node.js 18+
- npm or yarn
- Supabase account (for backend features)

### Installation

```bash
# Install dependencies
npm install

# Set up environment variables
cp .env.example .env.local

# Start development server
npm run dev
```

Visit http://localhost:3000

### Connecting Supabase

1. Click the cloud button (☁️) in the preview panel
2. Connect your Supabase project
3. The backend will be automatically configured
4. Run the SQL schema from `src/lib/db-schema.sql` in Supabase
5. API routes in `src/app/api/` will automatically integrate

## 📱 Features

- ✅ Role-based authentication (Patient/Doctor/Admin)
- ✅ Appointment scheduling and management
- ✅ Real-time appointment status updates
- ✅ Medical records and prescription tracking
- ✅ Doctor search and filtering
- ✅ AI health guidance chatbot
- ✅ Pharmacy medicine catalog
- ✅ Patient visit history reports
- ✅ Admin dashboard with analytics
- ✅ Email notifications (prepared)
- ✅ Dark theme with glassmorphism design
- ✅ Framer Motion animations
- ✅ Responsive design (mobile/tablet/desktop)
- ✅ TypeScript type safety
- ✅ Accessible UI components

## 🎨 Design Features

- **Dark Healthcare Aesthetic**: Professional, trustworthy dark theme
- **Glassmorphism**: Frosted glass effects on cards with backdrop blur
- **Gradient Text**: Teal to amber gradient effects
- **Smooth Animations**: Staggered reveals, hover effects, transitions
- **Icon System**: Lucide React icons throughout
- **Responsive Grid**: Flexible layouts for all screen sizes
- **Semantic Colors**: Primary, secondary, accent, destructive, muted

## 📦 File Structure

```
src/
├── app/
│   ├── page.tsx              # Landing page
│   ├── auth/                 # Authentication
│   ├── patient/              # Patient portal
│   ├── doctor/               # Doctor portal
│   ├── admin/                # Admin dashboard
│   ├── find-doctors/         # Doctor search
│   ├── book-appointment/     # Appointment wizard
│   ├── ai-consultation/      # Health chatbot
│   ├── pharmacy/             # Medicine catalog
│   ├── view-reports/         # Patient reports
│   └── api/                  # API routes
├── components/
│   └── ui/                   # shadcn/ui components
├── lib/
│   ├── db-schema.sql         # Database schema
│   └── utils.ts              # Utility functions
├── hooks/                    # Custom React hooks
└── theme.css                 # Global theme variables
```

## 🔌 API Routes

### Authentication
- `POST /api/auth/signup` - Create new user
- `POST /api/auth/signin` - Sign in user
- `POST /api/auth/signout` - Sign out user

### Appointments
- `GET /api/appointments` - Fetch appointments
- `POST /api/appointments` - Create appointment
- `PATCH /api/appointments/[id]` - Update appointment

### Doctors
- `GET /api/doctors` - Get doctor list
- `GET /api/doctors/[id]` - Get doctor profile
- `PATCH /api/doctors/[id]` - Update doctor profile

## 📝 Environment Variables

```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
```

## 🧪 Testing

All 10 pages have been tested and are working:
- ✅ Landing page (/)
- ✅ Auth page (/auth)
- ✅ Patient portal (/patient)
- ✅ Doctor portal (/doctor)
- ✅ Admin portal (/admin)
- ✅ View reports (/view-reports)
- ✅ Find doctors (/find-doctors)
- ✅ Book appointment (/book-appointment)
- ✅ AI consultation (/ai-consultation)
- ✅ Pharmacy (/pharmacy)

## 🔐 Security

- Row Level Security (RLS) on all database tables
- Role-based access control
- Email verification for sign-up
- Secure password handling via Supabase Auth
- Protected API routes
- CSRF protection with Next.js

## 📄 License

MIT License - See LICENSE file for details

## 👥 Contributing

Contributions are welcome! Please follow the existing code style and create feature branches.

## 📞 Support

For support, please reach out to team@raccoonai.tech
