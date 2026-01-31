# Cosmetic Booking Website - Design Document

**Date**: 2026-02-01
**Project**: Lian Cosmetic Booking System
**Status**: Design Complete - Ready for Implementation

## Executive Summary

A modern, mobile-first cosmetic treatment booking website with two interfaces:
1. **Public website**: Customers browse treatments and book appointments via shareable link
2. **Admin dashboard**: Cosmetic professional manages treatments, availability, and bookings

**Key Goals**:
- Frictionless booking experience (no customer login required)
- Complete schedule control for admin
- Modern, smooth user interface
- Analytics to optimize business operations
- Zero cost to start (free tier services)

---

## Technology Stack

### Frontend
- **Framework**: Next.js 14+ (React) with TypeScript
- **Styling**: Tailwind CSS + shadcn/ui components
- **State Management**: React Context + React Query
- **Forms**: React Hook Form with Zod validation
- **Deployment**: Vercel (free tier)

### Backend
- **Framework**: FastAPI (Python 3.14)
- **ORM**: SQLAlchemy
- **Database**: PostgreSQL
- **Deployment**: Railway or Render (free tier)

### External Services (All Free Tier)
- **File Storage**: Cloudinary (25 GB storage, 25 GB bandwidth)
- **Email**: Resend (100 emails/day, 3,000/month)
- **Analytics**: Vercel Analytics

### Cost Summary
- **Initial**: $0
- **Scaling**: Only pay when exceeding free tier limits

---

## Architecture Overview

```
Customer Device (Browser)
    ↓
Next.js Frontend (Vercel)
    ↓
FastAPI Backend (Railway/Render)
    ↓
PostgreSQL Database
```

**Two Main Interfaces**:
1. Public routes: `/`, `/treatments/*`, `/book`, `/booking/{token}`
2. Admin routes: `/admin/*` (password-protected)

**Authentication**:
- Customers: No login, manage bookings via unique token link
- Admin: Email + password with session-based auth

---

## Database Schema

### Practitioners (Staff Members)
```sql
- id (primary key)
- name (string)
- email (string, unique)
- phone (string)
- bio (text)
- profile_photo_url (string)
- credentials (text)
- status (enum: active/inactive)
- created_at, updated_at
```

### Treatments (Services)
```sql
- id (primary key)
- name (string)
- slug (string, unique, for URLs)
- category (string)
- description (text, rich content)
- duration_minutes (integer)
- price (decimal)
- buffer_time_minutes (integer, cleanup/setup)
- active (boolean)
- display_order (integer, for sorting)
- created_at, updated_at
```

### Treatment_Images
```sql
- id (primary key)
- treatment_id (foreign key)
- image_url (string)
- is_primary (boolean)
- is_before_after (boolean)
- display_order (integer)
```

### Practitioner_Treatments (Many-to-Many)
```sql
- practitioner_id (foreign key)
- treatment_id (foreign key)
```

### Availability_Rules (Weekly Schedule)
```sql
- id (primary key)
- practitioner_id (foreign key)
- day_of_week (enum: 0-6, Monday-Sunday)
- start_time (time)
- end_time (time)
- is_available (boolean)
```

### Availability_Breaks (Lunch, breaks within day)
```sql
- id (primary key)
- availability_rule_id (foreign key)
- start_time (time)
- end_time (time)
```

### Availability_Overrides (Holidays, Custom Days)
```sql
- id (primary key)
- practitioner_id (foreign key)
- date (date)
- override_type (enum: closed, custom_hours)
- start_time (time, nullable)
- end_time (time, nullable)
- reason (string, nullable)
```

### Bookings (Appointments)
```sql
- id (primary key)
- unique_token (string, unique, for customer access)
- treatment_id (foreign key)
- practitioner_id (foreign key)
- customer_name (string)
- customer_email (string)
- customer_phone (string)
- appointment_date (date)
- appointment_time (time)
- status (enum: pending, confirmed, completed, cancelled, no_show)
- customer_notes (text, nullable)
- admin_notes (text, nullable)
- cancellation_reason (text, nullable)
- created_at, updated_at
```

### Admin_Users
```sql
- id (primary key)
- email (string, unique)
- password_hash (string)
- name (string)
- role (enum: owner, staff)
- created_at, updated_at
```

---

## Customer Booking Flow

### Homepage (`/`)
- **Hero Section**: Business name, tagline, "Book Your Treatment" CTA
- **Treatment Gallery**: Card grid with images, name, duration, price
- **Category Filter**: Quick navigation buttons
- **About Section**: Practitioner profile with photo, bio, credentials
- **Mobile-first**: Smooth animations, touch-optimized

### Treatment Detail Page (`/treatments/[slug]`)
- Large image gallery with lightbox
- Before/after photos carousel
- Detailed description
- Duration, price prominently displayed
- "Book Now" button (sticky on mobile)
- Related treatments suggestions

### Booking Flow (`/book`)

**Step 1: Select Treatment**
- Pre-selected if coming from treatment page
- Or choose from dropdown/grid

**Step 2: Choose Date & Time**
- Calendar view (unavailable dates greyed out)
- Time slot picker for selected date
- Slots calculated in real-time:
  - Treatment duration + buffer time
  - Practitioner availability rules
  - Existing bookings considered
- Shows practitioner name per slot

**Step 3: Customer Information**
- Simple form: name, email, phone
- Optional notes field
- No password or account creation
- Privacy note

**Step 4: Confirmation**
- Review summary
- "Confirm Booking" button
- Instant confirmation email sent

### Booking Management (`/booking/[token]`)
- Customer accesses via unique link from email
- View booking details
- Cancel (if before cutoff window)
- Request reschedule (notifies admin)

---

## Admin Dashboard

### Authentication (`/admin/login`)
- Email + password
- Session-based with secure cookies
- "Remember me" option
- Password reset via email

### Dashboard Overview (`/admin`)
- **Today's Summary**: Appointments count, expected revenue
- **Upcoming Appointments**: Next 5 bookings with quick actions
- **Quick Stats**: This week vs last week comparison
- **Calendar Widget**: Month view with booking density

### Bookings Management (`/admin/bookings`)

**Calendar View**:
- Monthly/weekly/daily views
- Color-coded by treatment type
- Drag-and-drop to reschedule
- Click to view details

**List View**:
- Filterable table (date range, status, treatment, customer)
- Search by customer name/email
- Export to CSV

**Booking Details Modal**:
- Full information display
- Customer contact (mailto links)
- Quick actions:
  - Mark as completed
  - Cancel with reason
  - Reschedule
  - Add internal notes

### Schedule Management (`/admin/schedule`)

**Weekly Schedule Builder**:
- Visual grid for each day
- Drag to set hours (e.g., 9 AM - 5 PM)
- Add break times
- Copy schedule to other days

**Date Overrides**:
- Calendar picker
- Mark days off/holidays
- Set custom hours for specific dates
- Last-minute availability changes

**Buffer Time Settings**:
- Global default
- Per-treatment override

### Treatments Management (`/admin/treatments`)

**Treatment List**:
- Card/table view with thumbnails
- Quick edit/delete
- Drag-and-drop reordering

**Add/Edit Treatment**:
- Name, category, description (rich text)
- Pricing
- Duration + buffer time
- Image uploads (drag-drop, multiple)
- Before/after gallery
- Practitioner assignment (checkboxes)
- Active/inactive toggle

**Categories Management**:
- Create/edit categories
- Reorder display

### Practitioner Profiles (`/admin/practitioners`)
- Edit bio, credentials
- Upload profile photo
- Portfolio/gallery management
- Skills assignment (treatments)
- For future: manage multiple practitioners

### Analytics (`/admin/analytics`)

**Time Period Selector**: Week, 30 days, 3 months, custom

**Key Metrics**:
- Total bookings with trend (↑5% vs previous)
- Total revenue with trend
- Average booking value
- Cancellation rate

**Visual Charts**:
- Bookings over time (line chart)
- Revenue trends (bar chart)
- Popular treatments (pie chart)
- Busiest days/times heatmap

**Customer Insights**:
- New vs returning ratio
- Most booked treatment
- Peak booking times

---

## API Endpoints

### Public APIs (No Auth)
```
GET    /api/treatments              - List active treatments
GET    /api/treatments/{id}         - Treatment details
GET    /api/practitioners           - Practitioner public profiles
GET    /api/availability            - Available slots (query: treatment_id, date)
POST   /api/bookings                - Create booking
GET    /api/bookings/{token}        - View booking by token
PUT    /api/bookings/{token}/cancel - Cancel booking
```

### Admin APIs (Auth Required)
```
POST   /api/admin/login             - Authentication
GET    /api/admin/bookings          - List all bookings (filters)
GET    /api/admin/bookings/{id}     - Booking details
PUT    /api/admin/bookings/{id}     - Update booking
DELETE /api/admin/bookings/{id}     - Cancel booking

POST   /api/admin/treatments        - Create treatment
GET    /api/admin/treatments        - List all treatments
PUT    /api/admin/treatments/{id}   - Update treatment
DELETE /api/admin/treatments/{id}   - Soft delete treatment

POST   /api/admin/availability/rules     - Set weekly schedule
GET    /api/admin/availability/rules     - Get schedule
PUT    /api/admin/availability/rules/{id} - Update rule
DELETE /api/admin/availability/rules/{id} - Delete rule

POST   /api/admin/availability/overrides - Add date override
GET    /api/admin/availability/overrides - List overrides
DELETE /api/admin/availability/overrides/{id} - Remove override

GET    /api/admin/analytics         - Analytics data (query: period)
POST   /api/admin/upload            - Upload image to cloud
```

---

## Smart Slot Generation Algorithm

**Purpose**: Calculate available booking slots for a given date and treatment

**Inputs**:
- Treatment ID (determines duration + buffer)
- Date
- Practitioner ID (optional, auto-select if not provided)

**Process**:
1. Fetch treatment details (duration, buffer time)
2. Fetch practitioner's availability rule for day of week
3. Check for date-specific overrides (holiday, custom hours)
4. If override type = closed, return empty slots
5. Calculate time blocks: `[start_time to end_time] - [break_times]`
6. Fetch existing bookings for that date/practitioner
7. Subtract booked slots (booking time + treatment duration + buffer)
8. Generate available slots based on treatment duration
9. Filter out past slots (can't book in past)
10. Return list of available time slots

**Optimization**:
- Cache results for 5 minutes (reduce DB load)
- Invalidate cache when booking created or schedule changed

**Race Condition Handling**:
- Optimistic locking: reserve slot for 10 minutes during booking flow
- Database constraint: unique (practitioner_id, appointment_date, appointment_time)
- Transaction-level locking on booking creation

---

## Email Notification System

### Customer Emails (Resend)

**1. Booking Confirmation**
- **Trigger**: Immediately after booking created
- **Subject**: "Your [Treatment Name] appointment is confirmed!"
- **Content**:
  - Treatment details, date/time, practitioner
  - Business address and directions link
  - CTA buttons: "Add to Calendar", "Manage Booking"
  - Cancellation policy reminder
- **Mobile-responsive** HTML template

**2. Booking Reminder**
- **Trigger**: 24 hours before appointment (cron job)
- **Subject**: "Reminder: Your appointment is tomorrow"
- **Content**:
  - Quick summary
  - Preparation instructions
  - "Need to reschedule?" link

**3. Cancellation Confirmation**
- **Trigger**: When customer cancels
- **Subject**: "Your appointment has been cancelled"
- **Content**:
  - Acknowledge cancellation
  - "Book Again" button

**4. Admin-Initiated Changes**
- **Trigger**: When admin modifies booking
- **Subject**: "Your appointment has been updated"
- **Content**:
  - Before/after comparison
  - Reason if provided

### Admin Notifications

**1. New Booking Alert**
- **Trigger**: New booking created
- **To**: Admin email
- **Content**: Customer info, treatment, datetime
- **Link**: Direct link to booking in admin panel

**2. Cancellation Alert**
- **Trigger**: Customer cancels
- **To**: Admin email
- **Content**: Cancelled booking details

---

## Error Handling & Edge Cases

### Booking Conflicts
- **Optimistic Locking**: Reserve slot for 10 minutes during booking
- **Race Conditions**: First to confirm wins, second gets error + updated slots
- **Re-validation**: Check availability before final confirmation
- **User-Friendly Errors**: "This slot was just booked. Here are nearby times..."

### Common Edge Cases

**1. Double Booking Prevention**
- Database unique constraint: (practitioner_id, appointment_date, appointment_time)
- Transaction-level locking
- Admin can manually override (with warning)

**2. Cancellation Window**
- Configurable cutoff (e.g., 24 hours before)
- After cutoff: "Please call to cancel"
- Admin can always cancel

**3. Past Appointments**
- Customer link shows: "This appointment has passed"
- Admin can mark completed/no-show
- Auto-filter from "upcoming" views

**4. Timezone Handling**
- Store in UTC in database
- Display in business local timezone
- Show timezone to customer (e.g., "2:00 PM PST")

**5. Deleted Treatments**
- Soft delete (mark inactive)
- Existing bookings remain intact
- Hide from public listing

**6. Image Upload Failures**
- Save treatment without images
- Show retry option in admin
- Validate size/type before upload

---

## Mobile-First Design

### Responsive Breakpoints
- Mobile: < 768px
- Tablet: 768-1024px
- Desktop: > 1024px

### Touch Optimization
- Large tap targets (min 44x44px)
- Swipe gestures for galleries
- Pull-to-refresh on lists

### Performance
- Optimized images (WebP format)
- Lazy loading
- Code splitting per route
- Target: < 3s initial load on 3G

### PWA Features
- Can be installed on home screen
- Offline indicator
- App-like feel

### Accessibility (WCAG 2.1 AA)
- Keyboard navigation
- Screen reader support
- Color contrast compliance
- Alt text for all images
- Focus indicators

---

## Deployment Strategy

### Development Environment
- Docker Compose (PostgreSQL + FastAPI + Next.js)
- Environment variables in `.env` files
- `.env.example` for documentation

### Testing
- **Backend**: pytest for API tests
- **Frontend**: Vitest for components, Playwright for E2E
- **Manual**: Complete booking flow checklist

### Deployment Process

**Phase 1: Backend**
1. Create Railway/Render account
2. Provision PostgreSQL database
3. Deploy FastAPI application
4. Run database migrations
5. Seed initial admin user

**Phase 2: Frontend**
1. Connect GitHub repo to Vercel
2. Configure environment variables
3. Automatic deployment on push to main

**Phase 3: External Services**
1. Setup Cloudinary account
2. Configure Resend for emails
3. Test email delivery

### Launch Checklist
- [ ] Admin login works
- [ ] Admin can create treatments with images
- [ ] Admin can set weekly schedule
- [ ] Customers see treatments on homepage
- [ ] Customers can book appointments
- [ ] Confirmation emails sent
- [ ] Admin receives booking notifications
- [ ] Customers can manage bookings via link
- [ ] Analytics show correct data
- [ ] Mobile experience tested
- [ ] Payment integration ready (for future)

### Monitoring
- Vercel Analytics (frontend performance)
- Railway/Render logs (backend errors)
- Resend dashboard (email delivery)

### Custom Domain (Optional)
- Purchase domain
- Point to Vercel
- Automatic HTTPS

---

## Future Enhancements (Not in Initial Build)

### Payment Integration
- Stripe or PayPal integration
- Upfront payment or deposit option
- Refund handling

### Multi-Channel Notifications
- SMS via Twilio
- WhatsApp Business API
- Push notifications (PWA)

### Advanced Features
- Multi-session packages
- Gift certificates
- Loyalty program
- Customer accounts (optional)
- Integration with Google Calendar
- Automated review requests
- Waitlist for popular slots

### Scaling
- Multiple practitioners with individual schedules
- Multiple locations
- Staff permissions and roles
- Inventory tracking for products
- Point-of-sale integration

---

## Success Metrics

### Launch Goals (First 3 Months)
- Booking conversion rate > 60% (visitors who book)
- < 5% booking errors/failures
- < 10% cancellation rate
- Mobile users > 70%
- Page load time < 3 seconds

### Business Impact
- Reduce booking phone calls/texts by 80%
- Increase bookings through 24/7 availability
- Better schedule utilization through analytics
- Professional brand image

---

## Conclusion

This design provides a comprehensive, modern booking solution that:
- ✅ Starts at zero cost with free tier services
- ✅ Scales automatically as business grows
- ✅ Provides smooth customer experience (no login friction)
- ✅ Gives admin complete control over schedule
- ✅ Includes analytics for business optimization
- ✅ Mobile-first design for on-the-go booking
- ✅ Professional, trustworthy appearance

**Next Steps**:
1. Set up git worktree for isolated development
2. Create detailed implementation plan
3. Begin Phase 1: Backend scaffolding
