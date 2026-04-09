# Supabase Setup Guide for Helora

This guide will help you set up the Supabase backend for Helora Hospital Management System.

## Prerequisites

- Supabase account (https://supabase.com)
- Project created in Supabase
- API keys from your project

## Step 1: Connect Supabase to Helora

1. Click the cloud button (☁️) in the Helora preview panel
2. Select Supabase as your backend provider
3. Enter your Supabase project credentials:
   - **Project URL**: Found in Project Settings → API
   - **Anon Key**: Found in Project Settings → API
   - **Service Role Key**: Found in Project Settings → API

## Step 2: Create Database Schema

### Option A: Using SQL (Recommended)

1. Open your Supabase project dashboard
2. Go to SQL Editor
3. Click "New Query"
4. Copy and paste the SQL schema from `/workspace/web/src/lib/db-schema.sql`
5. Click "Run"

### Option B: Manual Table Creation

If you prefer to create tables manually, follow the table definitions below:

#### 1. Profiles Table
```sql
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  email TEXT UNIQUE NOT NULL,
  full_name TEXT NOT NULL,
  phone_number TEXT,
  date_of_birth DATE,
  role TEXT NOT NULL DEFAULT 'patient',
  avatar_url TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_profiles_email ON profiles(email);
CREATE INDEX idx_profiles_phone ON profiles(phone_number);

ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
```

#### 2. Doctors Table
```sql
CREATE TABLE doctors (
  id UUID PRIMARY KEY REFERENCES profiles(id) ON DELETE CASCADE,
  specialty TEXT NOT NULL,
  experience_years INTEGER,
  rating DECIMAL(3, 2) DEFAULT 0,
  bio TEXT,
  clinic_address TEXT,
  availability_status BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_doctors_specialty ON doctors(specialty);

ALTER TABLE doctors ENABLE ROW LEVEL SECURITY;
```

#### 3. Appointments Table
```sql
CREATE TABLE appointments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  patient_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  doctor_id UUID NOT NULL REFERENCES doctors(id) ON DELETE CASCADE,
  appointment_date DATE NOT NULL,
  appointment_time TIME NOT NULL,
  visit_type TEXT NOT NULL,
  notes TEXT,
  status TEXT DEFAULT 'scheduled',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_appointments_patient ON appointments(patient_id);
CREATE INDEX idx_appointments_doctor ON appointments(doctor_id);
CREATE INDEX idx_appointments_date ON appointments(appointment_date);

ALTER TABLE appointments ENABLE ROW LEVEL SECURITY;
```

#### 4. Prescriptions Table
```sql
CREATE TABLE prescriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  patient_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  doctor_id UUID NOT NULL REFERENCES doctors(id) ON DELETE CASCADE,
  appointment_id UUID REFERENCES appointments(id) ON DELETE SET NULL,
  medication_name TEXT NOT NULL,
  dosage TEXT NOT NULL,
  frequency TEXT NOT NULL,
  duration TEXT NOT NULL,
  notes TEXT,
  prescribed_date TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_prescriptions_patient ON prescriptions(patient_id);

ALTER TABLE prescriptions ENABLE ROW LEVEL SECURITY;
```

#### 5. Medicines Table
```sql
CREATE TABLE medicines (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL UNIQUE,
  category TEXT NOT NULL,
  description TEXT,
  price DECIMAL(10, 2) NOT NULL,
  dosage_form TEXT,
  manufacturer TEXT,
  in_stock BOOLEAN DEFAULT true,
  quantity_available INTEGER,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

ALTER TABLE medicines ENABLE ROW LEVEL SECURITY;
```

#### 6. Visit History Table
```sql
CREATE TABLE visit_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  patient_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  doctor_id UUID NOT NULL REFERENCES doctors(id) ON DELETE CASCADE,
  appointment_id UUID NOT NULL REFERENCES appointments(id) ON DELETE CASCADE,
  diagnosis TEXT,
  treatment_notes TEXT,
  visit_date TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_visit_history_patient ON visit_history(patient_id);
CREATE INDEX idx_visit_history_date ON visit_history(visit_date);

ALTER TABLE visit_history ENABLE ROW LEVEL SECURITY;
```

## Step 3: Set Up Row Level Security (RLS) Policies

After creating tables, set up RLS policies for security. Apply these policies in the SQL Editor:

### Profiles Policies
```sql
-- Users can view their own profile
CREATE POLICY "Users can view own profile"
  ON profiles FOR SELECT
  USING (auth.uid() = id);

-- Users can update their own profile
CREATE POLICY "Users can update own profile"
  ON profiles FOR UPDATE
  USING (auth.uid() = id)
  WITH CHECK (auth.uid() = id);

-- Doctors can view patient profiles for appointments
CREATE POLICY "Doctors can view patient profiles"
  ON profiles FOR SELECT
  USING (
    (SELECT role FROM profiles WHERE id = auth.uid()) = 'doctor'
    OR auth.uid() = id
  );

-- Admins can view all profiles
CREATE POLICY "Admins can view all profiles"
  ON profiles FOR SELECT
  USING ((SELECT role FROM profiles WHERE id = auth.uid()) = 'admin');
```

### Doctors Policies
```sql
-- Anyone can view doctor profiles
CREATE POLICY "Anyone can view doctor profiles"
  ON doctors FOR SELECT
  USING (true);

-- Doctors can update their own profile
CREATE POLICY "Doctors can update own profile"
  ON doctors FOR UPDATE
  USING (auth.uid() = id)
  WITH CHECK (auth.uid() = id);

-- Admins can manage doctor profiles
CREATE POLICY "Admins can manage doctor profiles"
  ON doctors FOR ALL
  USING ((SELECT role FROM profiles WHERE id = auth.uid()) = 'admin');
```

### Appointments Policies
```sql
-- Patients can view their own appointments
CREATE POLICY "Patients can view own appointments"
  ON appointments FOR SELECT
  USING (auth.uid() = patient_id);

-- Doctors can view their appointments
CREATE POLICY "Doctors can view their appointments"
  ON appointments FOR SELECT
  USING (auth.uid() = doctor_id);

-- Patients can create appointments
CREATE POLICY "Patients can create appointments"
  ON appointments FOR INSERT
  WITH CHECK (auth.uid() = patient_id);

-- Patients can update their appointments
CREATE POLICY "Patients can update own appointments"
  ON appointments FOR UPDATE
  USING (auth.uid() = patient_id)
  WITH CHECK (auth.uid() = patient_id);

-- Doctors can update appointment status
CREATE POLICY "Doctors can update appointment status"
  ON appointments FOR UPDATE
  USING (auth.uid() = doctor_id)
  WITH CHECK (auth.uid() = doctor_id);

-- Admins can view all appointments
CREATE POLICY "Admins can view all appointments"
  ON appointments FOR SELECT
  USING ((SELECT role FROM profiles WHERE id = auth.uid()) = 'admin');
```

### Prescriptions Policies
```sql
-- Patients can view their own prescriptions
CREATE POLICY "Patients can view own prescriptions"
  ON prescriptions FOR SELECT
  USING (auth.uid() = patient_id);

-- Doctors can view their prescriptions
CREATE POLICY "Doctors can view their prescriptions"
  ON prescriptions FOR SELECT
  USING (auth.uid() = doctor_id);

-- Doctors can create prescriptions
CREATE POLICY "Doctors can create prescriptions"
  ON prescriptions FOR INSERT
  WITH CHECK (auth.uid() = doctor_id);
```

### Medicines Policies
```sql
-- Anyone can view medicines
CREATE POLICY "Anyone can view medicines"
  ON medicines FOR SELECT
  USING (true);

-- Admins can manage medicines
CREATE POLICY "Admins can manage medicines"
  ON medicines FOR ALL
  USING ((SELECT role FROM profiles WHERE id = auth.uid()) = 'admin');
```

### Visit History Policies
```sql
-- Patients can view their own visit history
CREATE POLICY "Patients can view own visit history"
  ON visit_history FOR SELECT
  USING (auth.uid() = patient_id);

-- Doctors can view their visit history
CREATE POLICY "Doctors can view their visit history"
  ON visit_history FOR SELECT
  USING (auth.uid() = doctor_id);

-- Doctors can create visit history
CREATE POLICY "Doctors can create visit history"
  ON visit_history FOR INSERT
  WITH CHECK (auth.uid() = doctor_id);
```

## Step 4: Create Helper Functions (Optional)

For advanced use cases, create these helper functions:

```sql
-- Function to get user role
CREATE OR REPLACE FUNCTION get_user_role()
RETURNS TEXT AS $$
SELECT role FROM profiles WHERE id = auth.uid()
$$ LANGUAGE SQL SECURITY DEFINER;

-- Function to update timestamps
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Apply triggers
CREATE TRIGGER update_profiles_updated_at BEFORE UPDATE ON profiles
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_doctors_updated_at BEFORE UPDATE ON doctors
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_appointments_updated_at BEFORE UPDATE ON appointments
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_prescriptions_updated_at BEFORE UPDATE ON prescriptions
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_medicines_updated_at BEFORE UPDATE ON medicines
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_visit_history_updated_at BEFORE UPDATE ON visit_history
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

## Step 5: Populate Sample Data (Optional)

For testing, you can populate sample data:

```sql
-- Sample Doctors
INSERT INTO profiles (id, email, full_name, phone_number, role, created_at, updated_at)
VALUES 
  ('doctor-id-1'::uuid, 'dr.chen@helora.com', 'Dr. Michael Chen', '+1 (555) 111-2222', 'doctor', NOW(), NOW()),
  ('doctor-id-2'::uuid, 'dr.rodriguez@helora.com', 'Dr. Emily Rodriguez', '+1 (555) 222-3333', 'doctor', NOW(), NOW());

-- Sample Doctor Profiles
INSERT INTO doctors (id, specialty, experience_years, rating, bio, clinic_address, availability_status)
VALUES
  ('doctor-id-1'::uuid, 'Cardiology', 12, 4.8, 'Expert cardiologist', '123 Medical Plaza', true),
  ('doctor-id-2'::uuid, 'General Practice', 8, 4.6, 'General practitioner', '456 Health Center', true);

-- Sample Patient
INSERT INTO profiles (id, email, full_name, phone_number, role, created_at, updated_at)
VALUES ('patient-id-1'::uuid, 'sarah@helora.com', 'Sarah Johnson', '+1 (555) 123-4567', 'patient', NOW(), NOW());

-- Sample Medicines
INSERT INTO medicines (name, category, description, price, dosage_form, manufacturer, in_stock, quantity_available)
VALUES
  ('Aspirin 500mg', 'Pain Relief', 'Effective pain relief', 5.99, '500mg tablets', 'Bayer', true, 150),
  ('Metformin 1000mg', 'Diabetes', 'Blood sugar management', 12.99, '1000mg tablets', 'Mylan', true, 200);
```

## Step 6: Set Up Environment Variables

Update your `.env.local` file with your Supabase credentials:

```env
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key-here
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key-here
```

## Step 7: Test the Integration

1. Restart the Helora development server
2. Try signing up with a new account
3. Create a test appointment
4. Check the Supabase dashboard to verify data is being saved

## Troubleshooting

### Authentication Issues
- Ensure "Enable email confirmations" is turned off in Supabase Auth settings for testing
- Check that the `profiles` table is being created before user signup

### RLS Policy Errors
- Verify all RLS policies are enabled on each table
- Check that `auth.uid()` is properly returning the authenticated user ID

### Data Not Appearing
- Confirm Row Level Security is enabled on all tables
- Check the browser console for API errors
- Verify credentials in `.env.local` are correct

## Additional Resources

- [Supabase Documentation](https://supabase.com/docs)
- [Row Level Security Guide](https://supabase.com/docs/guides/auth/row-level-security)
- [Supabase CLI](https://supabase.com/docs/reference/cli)

## Support

For issues or questions, reach out to team@raccoonai.tech
