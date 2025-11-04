# Prisma Schema and Seed Script for My Family Clinic

Create an enhanced version that focuses on performance optimization, comprehensive data modeling, and production-ready features.

## File: prisma/schema.prisma

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// -------- Enums --------
enum EnquiryTopic {
  GENERAL
  APPOINTMENT
  CORPORATE
}

enum EnquiryStatus {
  RECEIVED
  ROUTED
  RESPONDED
}

enum AdminRole {
  ADMIN
  EDITOR
}

enum ServiceCategory {
  GP
  ACUTE
  CHRONIC
  SCREENING
  VACCINATION
  CORPORATE
}

// -------- Models --------

// Clinics hold core public info and relationships to services and doctors
model Clinic {
  id                         String    @id @default(cuid())
  name                       String
  slug                       String    @unique
  address                    String
  postal_code                String
  district                   String
  phone                      String
  email                      String?
  // Geo: store as WKT or lat/lng JSON; for PostGIS point, use a separate native column via SQL migration.
  latitude                   Float?
  longitude                  Float?

  // Operational metadata
  hours_json                 Json      // structured opening hours per weekday
  services_json              Json?     // optional denormalized mirror for fast UI rendering
  is_healthier_sg_participant Boolean   @default(false)
  accessibility_tags         String[]  // e.g., ["wheelchair", "baby-changing"]
  
  // Enhanced fields for production features
  description                String?   // Brief description of the clinic
  photo_url                  String?   // Main clinic photo
  facilities                 String[]  // e.g., ["x-ray", "ultrasound", "pharmacy"]
  status                     String    @default("active") // "active", "inactive", "coming-soon"

  // Relations
  clinicServices             ClinicsOnServices[]
  clinicDoctors              ClinicsOnDoctors[]
  clinicSchemes              ClinicsOnSchemes[]
  auditLogs                  AuditLog[]

  created_at                 DateTime  @default(now())
  updated_at                 DateTime  @updatedAt

  @@index([district])
  @@index([postal_code])
  @@index([is_healthier_sg_participant])
  @@index([status])
  // Full-text search index for name/address (Prisma fulltext uses tsvector; for pg_trgm create via SQL migration)
  @@fulltext([name, address])
}

// Medical services offered across clinics
model Service {
  id                 String   @id @default(cuid())
  name               String
  slug               String   @unique
  category           ServiceCategory // e.g., "GP", "Acute", "Chronic", "Screening", "Vaccination", "Corporate"
  short_description  String
  content_md         String   // markdown body
  preparation_md     String?
  eligibility_md     String?
  duration           Int?     // Duration in minutes
  price_range        String?  // e.g., "$30-$50" or "From $30"
  accepted_schemes   String[] // e.g., ["CHAS", "Medisave", "Panel-X"]
  faq_json           Json?
  status             String   @default("active") // "active", "inactive"
  
  // Enhanced fields for production features
  icon_name          String?  // Icon identifier for UI
  photo_url          String?  // Service photo
  featured           Boolean  @default(false) // Highlight in UI

  // Relations
  clinicServices     ClinicsOnServices[]
  serviceSchemes     ServicesOnSchemes[]
  auditLogs          AuditLog[]

  created_at         DateTime @default(now())
  updated_at         DateTime @updatedAt

  @@index([category])
  @@index([status])
  @@index([featured])
  @@fulltext([name, short_description, content_md])
}

// Doctors directory
model Doctor {
  id            String   @id @default(cuid())
  full_name     String
  slug          String   @unique
  credentials   String?  // e.g., "MBBS (NUS), MMed (Fam Med)"
  specialties   String[] // e.g., ["Family Medicine", "Women's Health"]
  languages     String[] // e.g., ["English", "Mandarin", "Malay"]
  bio_md        String?
  photo_url     String?
  
  // Enhanced fields for production features
  consultation_fee Float? // Consultation fee in SGD
  gender        String?  // "male", "female", "other"
  years_experience Int?   // Years of experience
  status        String   @default("active") // "active", "inactive", "on-leave"

  // Relations
  clinicDoctors ClinicsOnDoctors[]
  auditLogs     AuditLog[]

  created_at    DateTime @default(now())
  updated_at    DateTime @updatedAt

  @@index([status])
  @@fulltext([full_name, credentials, bio_md])
}

// News articles
model News {
  id            String   @id @default(cuid())
  title         String
  slug          String   @unique
  summary       String?
  content_md    String
  published_at  DateTime?
  author        String?
  tags          String[]
  featured_image_url String? // Main image for the article
  status        String   @default("draft") // "draft", "published", "archived"
  
  // Enhanced fields for production features
  meta_title    String?  // SEO meta title
  meta_description String? // SEO meta description
  reading_time  Int?     // Estimated reading time in minutes

  created_at    DateTime @default(now())
  updated_at    DateTime @updatedAt

  @@index([published_at])
  @@index([status])
  @@fulltext([title, summary, content_md])
}

// Patient enquiries (public submission)
model Enquiry {
  id           String        @id @default(cuid())
  topic        EnquiryTopic
  name         String
  email        String
  phone        String?
  message_md   String
  consent      Boolean
  status       EnquiryStatus @default(RECEIVED)
  routed_to    String?       // email or department/clinic reference
  reference_id String?       // Unique reference for tracking
  
  // Enhanced fields for production features
  ip_address   String?       // For rate limiting and spam protection
  user_agent   String?       // For analytics
  spam_score   Float?        // Spam detection score
  notes        String?       // Internal notes for staff
  
  created_at   DateTime      @default(now())
  updated_at   DateTime      @updatedAt

  @@index([created_at])
  @@index([status])
  @@index([topic])
  @@index([email])
}

// Accreditation and insurance schemes
model Scheme {
  id             String   @id @default(cuid())
  code           String   @unique // e.g., "CHAS", "MEDISAVE"
  name           String
  description_md String?
  link_url       String?
  badge_url      String?  // Badge image for UI
  status         String   @default("active") // "active", "inactive"
  
  // Enhanced fields for production features
  category       String?  // "government", "insurance", "corporate"
  eligibility_md String?  // Who is eligible for this scheme

  // Relations
  serviceSchemes ServicesOnSchemes[]
  clinicSchemes  ClinicsOnSchemes[]
  auditLogs      AuditLog[]

  updated_at     DateTime @updatedAt
}

model Accreditation {
  id             String   @id @default(cuid())
  name           String
  description_md String?
  badge_url      String?
  link_url       String?
  status         String   @default("active") // "active", "inactive"
  
  // Enhanced fields for production features
  issued_by      String?  // Issuing authority
  valid_until    DateTime? // Expiry date
  display_order  Int      @default(0) // Order in UI

  updated_at     DateTime @updatedAt
}

// Admin users for CMS (NextAuth adapter creates core tables separately)
model AdminUser {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  role      AdminRole @default(EDITOR)
  status    String   @default("active") // "active", "inactive"
  last_login DateTime?
  
  // Enhanced fields for production features
  profile_image_url String?
  bio_md    String?
  
  created_at DateTime @default(now())
  updated_at DateTime @updatedAt

  // Relations
  auditLogs AuditLog[]
}

// Audit log for admin mutations
model AuditLog {
  id            String   @id @default(cuid())
  entity        String   // "Clinic" | "Service" | "Doctor" | "News" | "Scheme"
  entityId      String
  actorUserId   String
  action        String   // "CREATE" | "UPDATE" | "DELETE"
  changes_json  Json     // diffs old/new per field
  reason        String
  timestamp     DateTime @default(now())
  
  // Enhanced fields for production features
  ipAddress     String?  // IP address of the actor
  userAgent     String?  // User agent of the actor
  sessionId     String?  // Session ID for tracking

  // Relations
  actorUser     AdminUser @relation(fields: [actorUserId], references: [id])

  @@index([entity])
  @@index([entityId])
  @@index([actorUserId])
  @@index([timestamp])
}

// Analytics events tracking
model AnalyticsEvent {
  id            String   @id @default(cuid())
  event_id      String   // e.g., "click_locate_cta"
  payload_json  Json     // Event payload
  session_id    String?  // Session identifier
  user_id       String?  // User identifier (if logged in)
  ip_address    String?  // IP address (anonymized)
  user_agent    String?  // User agent
  timestamp     DateTime @default(now())

  @@index([event_id])
  @@index([timestamp])
  @@index([session_id])
}

// -------- M:N Join Tables --------

model ClinicsOnServices {
  clinicId  String
  serviceId String
  // Enhanced fields for production features
  priority  Int      @default(0) // Display priority
  notes     String?  // Special notes about this service at this clinic
  created_at DateTime @default(now())

  clinic   Clinic  @relation(fields: [clinicId], references: [id], onDelete: Cascade)
  service  Service @relation(fields: [serviceId], references: [id], onDelete: Cascade)

  @@id([clinicId, serviceId])
  @@index([serviceId])
}

model ClinicsOnDoctors {
  clinicId String
  doctorId String
  // Enhanced fields for production features
  schedule_json Json?   // Doctor's schedule at this clinic
  consultation_fee Float? // Fee override for this clinic
  created_at DateTime @default(now())

  clinic  Clinic  @relation(fields: [clinicId], references: [id], onDelete: Cascade)
  doctor  Doctor  @relation(fields: [doctorId], references: [id], onDelete: Cascade)

  @@id([clinicId, doctorId])
  @@index([doctorId])
}

model ServicesOnSchemes {
  serviceId String
  schemeId  String
  // Enhanced fields for production features
  coverage_details String? // Specific coverage details
  created_at DateTime @default(now())

  service Service @relation(fields: [serviceId], references: [id], onDelete: Cascade)
  scheme  Scheme  @relation(fields: [schemeId], references: [id], onDelete: Cascade)

  @@id([serviceId, schemeId])
  @@index([schemeId])
}

model ClinicsOnSchemes {
  clinicId String
  schemeId String
  // Enhanced fields for production features
  notes    String? // Special notes about this scheme at this clinic
  created_at DateTime @default(now())

  clinic Clinic @relation(fields: [clinicId], references: [id], onDelete: Cascade)
  scheme Scheme @relation(fields: [schemeId], references: [id], onDelete: Cascade)

  @@id([clinicId, schemeId])
  @@index([schemeId])
}
```

## File: prisma/migrations/001_add_trigram_indexes.sql

```sql
-- Enable extensions
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE EXTENSION IF NOT EXISTS postgis;

-- Trigram indexes for fuzzy search
CREATE INDEX IF NOT EXISTS clinic_name_trgm_idx ON "Clinic" USING GIN (name gin_trgm_ops);
CREATE INDEX IF NOT EXISTS clinic_slug_trgm_idx ON "Clinic" USING GIN (slug gin_trgm_ops);
CREATE INDEX IF NOT EXISTS service_name_trgm_idx ON "Service" USING GIN (name gin_trgm_ops);
CREATE INDEX IF NOT EXISTS doctor_full_name_trgm_idx ON "Doctor" USING GIN (full_name gin_trgm_ops);

-- PostGIS point + GiST index for geolocation
ALTER TABLE "Clinic" ADD COLUMN IF NOT EXISTS geo geography(Point, 4326);
-- Backfill from lat/lng if available:
UPDATE "Clinic" SET geo = ST_SetSRID(ST_MakePoint(longitude, latitude), 4326)::geography WHERE latitude IS NOT NULL AND longitude IS NOT NULL;
CREATE INDEX IF NOT EXISTS clinic_geo_gist_idx ON "Clinic" USING GIST (geo);

-- Additional indexes for performance optimization
CREATE INDEX IF NOT EXISTS clinic_district_postal_idx ON "Clinic" (district, postal_code);
CREATE INDEX IF NOT EXISTS service_category_status_idx ON "Service" (category, status);
CREATE INDEX IF NOT EXISTS doctor_status_idx ON "Doctor" (status);
CREATE INDEX IF NOT EXISTS news_status_published_idx ON "News" (status, published_at);
CREATE INDEX IF NOT EXISTS enquiry_status_created_idx ON "Enquiry" (status, created_at);
CREATE INDEX IF NOT EXISTS audit_entity_timestamp_idx ON "AuditLog" (entity, timestamp);
```

## File: prisma/seed.ts

```ts
// prisma/seed.ts
import { PrismaClient, ServiceCategory, EnquiryTopic, EnquiryStatus, AdminRole } from "@prisma/client";

const prisma = new PrismaClient();

async function main() {
  console.log("Starting seed process...");

  // Schemes
  const chas = await prisma.scheme.upsert({
    where: { code: "CHAS" },
    update: {},
    create: {
      code: "CHAS",
      name: "Community Health Assist Scheme",
      description_md:
        "CHAS provides subsidies for medical and dental care at participating clinics.",
      link_url: "https://www.chas.sg/",
      badge_url: "/images/schemes/chas-badge.png",
      category: "government",
      eligibility_md: "Singapore Citizens with household monthly income per person of $1,800 and below.",
    },
  });

  const medisave = await prisma.scheme.upsert({
    where: { code: "MEDISAVE" },
    update: {},
    create: {
      code: "MEDISAVE",
      name: "MediSave",
      description_md:
        "MediSave allows Singaporeans to pay for certain medical expenses using their savings.",
      link_url: "https://www.cpf.gov.sg/member/growing-your-savings/medisave",
      badge_url: "/images/schemes/medisave-badge.png",
      category: "government",
      eligibility_md: "All Singapore Citizens and Permanent Residents with MediSave accounts.",
    },
  });

  const panelCompany = await prisma.scheme.upsert({
    where: { code: "PANEL-COMPANY" },
    update: {},
    create: {
      code: "PANEL-COMPANY",
      name: "Corporate Panel",
      description_md:
        "Corporate health panels for employee medical benefits.",
      category: "corporate",
      eligibility_md: "Employees of companies with corporate health plans.",
    },
  });

  // Services
  const gpConsult = await prisma.service.upsert({
    where: { slug: "gp-consultation" },
    update: {},
    create: {
      name: "GP Consultation",
      slug: "gp-consultation",
      category: ServiceCategory.GP,
      short_description:
        "General practitioner consultations for acute and chronic conditions.",
      content_md:
        "Our experienced family doctors provide evidence-based care for common illnesses and ongoing conditions.",
      preparation_md: "Bring your identification card and any relevant medical records.",
      eligibility_md: "Available to all patients.",
      duration: 15,
      price_range: "$30-$50",
      accepted_schemes: ["CHAS", "MEDISAVE"],
      faq_json: {
        items: [
          { q: "Do I need an appointment?", a: "Walk-ins are welcome but appointments are recommended." },
          { q: "Are subsidies available?", a: "Yes, via CHAS and MediSave." },
          { q: "What should I bring?", a: "Please bring your ID and any relevant medical documents." },
        ],
      },
      icon_name: "stethoscope",
      featured: true,
    },
  });

  const vaccination = await prisma.service.upsert({
    where: { slug: "vaccination" },
    update: {},
    create: {
      name: "Vaccination",
      slug: "vaccination",
      category: ServiceCategory.VACCINATION,
      short_description:
        "Childhood and adult vaccinations, including travel immunisations.",
      content_md:
        "We offer a broad range of vaccinations according to MOH guidelines, including influenza, COVID-19, and travel vaccines.",
      preparation_md: "Bring your vaccination records if available.",
      eligibility_md: "Available to all patients. Some vaccines have specific age or health requirements.",
      duration: 15,
      price_range: "$20-$150",
      accepted_schemes: ["CHAS"],
      icon_name: "syringe",
      featured: true,
    },
  });

  const healthScreening = await prisma.service.upsert({
    where: { slug: "health-screening" },
    update: {},
    create: {
      name: "Health Screening",
      slug: "health-screening",
      category: ServiceCategory.SCREENING,
      short_description:
        "Comprehensive health screening packages for early detection of diseases.",
      content_md:
        "Our health screening packages include blood tests, physical examinations, and specialist referrals if needed.",
      preparation_md: "Fast for 8-10 hours before the screening. Drink water only.",
      eligibility_md: "Recommended for adults aged 18 and above.",
      duration: 30,
      price_range: "$80-$300",
      accepted_schemes: ["MEDISAVE"],
      icon_name: "clipboard",
      featured: true,
    },
  });

  const chronicCare = await prisma.service.upsert({
    where: { slug: "chronic-disease-management" },
    update: {},
    create: {
      name: "Chronic Disease Management",
      slug: "chronic-disease-management",
      category: ServiceCategory.CHRONIC,
      short_description:
        "Long-term management of chronic conditions like diabetes, hypertension, and asthma.",
      content_md:
        "Our team provides comprehensive care for chronic conditions, including medication management and lifestyle counseling.",
      preparation_md: "Bring your current medications and recent test results.",
      eligibility_md: "For patients diagnosed with chronic conditions.",
      duration: 20,
      price_range: "$40-$60",
      accepted_schemes: ["CHAS", "MEDISAVE"],
      icon_name: "heart",
    },
  });

  const corporateHealth = await prisma.service.upsert({
    where: { slug: "corporate-healthcare" },
    update: {},
    create: {
      name: "Corporate Healthcare",
      slug: "corporate-healthcare",
      category: ServiceCategory.CORPORATE,
      short_description:
        "Corporate health packages for employee wellness and medical benefits.",
      content_md:
        "We offer customized corporate health packages including regular check-ups, vaccinations, and health talks.",
      preparation_md: "Company registration required.",
      eligibility_md: "For companies with 5 or more employees.",
      duration: 30,
      price_range: "Custom packages",
      icon_name: "briefcase",
    },
  });

  // Clinics
  const clinicCentral = await prisma.clinic.upsert({
    where: { slug: "my-family-clinic-central" },
    update: {},
    create: {
      name: "My Family Clinic (Central)",
      slug: "my-family-clinic-central",
      address: "123 Central Street, #01-01, Singapore 123456",
      postal_code: "123456",
      district: "Central",
      phone: "+65 6000 0000",
      email: "central@myfamilyclinic.com.sg",
      latitude: 1.3000,
      longitude: 103.8000,
      description: "Our flagship clinic in the heart of Singapore with comprehensive family healthcare services.",
      photo_url: "/images/clinics/central-clinic.jpg",
      hours_json: {
        mon: { open: "08:30", close: "17:30" },
        tue: { open: "08:30", close: "17:30" },
        wed: { open: "08:30", close: "17:30" },
        thu: { open: "08:30", close: "17:30" },
        fri: { open: "08:30", close: "17:30" },
        sat: { open: "09:00", close: "13:00" },
        sun: null,
        ph: null,
      },
      services_json: {
        highlight: ["GP Consultation", "Vaccination", "Health Screening"],
      },
      is_healthier_sg_participant: true,
      accessibility_tags: ["wheelchair", "baby-changing"],
      facilities: ["x-ray", "ultrasound", "pharmacy", "laboratory"],
      status: "active",
    },
  });

  const clinicEast = await prisma.clinic.upsert({
    where: { slug: "my-family-clinic-east" },
    update: {},
    create: {
      name: "My Family Clinic (East)",
      slug: "my-family-clinic-east",
      address: "456 East Avenue, #02-10, Singapore 234567",
      postal_code: "234567",
      district: "East",
      phone: "+65 6000 0001",
      email: "east@myfamilyclinic.com.sg",
      latitude: 1.3200,
      longitude: 103.9400,
      description: "Conveniently located in the East, providing quality healthcare for the whole family.",
      photo_url: "/images/clinics/east-clinic.jpg",
      hours_json: {
        mon: { open: "08:30", close: "17:30" },
        tue: { open: "08:30", close: "17:30" },
        wed: { open: "10:00", close: "19:00" },
        thu: { open: "08:30", close: "17:30" },
        fri: { open: "08:30", close: "17:30" },
        sat: { open: "09:00", close: "13:00" },
        sun: null,
        ph: null,
      },
      is_healthier_sg_participant: false,
      accessibility_tags: ["wheelchair", "baby-changing"],
      facilities: ["pharmacy", "laboratory"],
      status: "active",
    },
  });

  const clinicWest = await prisma.clinic.upsert({
    where: { slug: "my-family-clinic-west" },
    update: {},
    create: {
      name: "My Family Clinic (West)",
      slug: "my-family-clinic-west",
      address: "789 West Boulevard, #03-05, Singapore 345678",
      postal_code: "345678",
      district: "West",
      phone: "+65 6000 0002",
      email: "west@myfamilyclinic.com.sg",
      latitude: 1.3500,
      longitude: 103.7500,
      description: "Serving the West community with dedicated healthcare professionals.",
      photo_url: "/images/clinics/west-clinic.jpg",
      hours_json: {
        mon: { open: "08:30", close: "17:30" },
        tue: { open: "08:30", close: "17:30" },
        wed: { open: "08:30", close: "17:30" },
        thu: { open: "08:30", close: "17:30" },
        fri: { open: "08:30", close: "17:30" },
        sat: { open: "09:00", close: "13:00" },
        sun: null,
        ph: null,
      },
      is_healthier_sg_participant: true,
      accessibility_tags: ["wheelchair"],
      facilities: ["pharmacy"],
      status: "active",
    },
  });

  // Doctors
  const drTan = await prisma.doctor.upsert({
    where: { slug: "dr-tan-wei-ling" },
    update: {},
    create: {
      full_name: "Dr Tan Wei Ling",
      slug: "dr-tan-wei-ling",
      credentials: "MBBS (NUS), MMed (Fam Med)",
      specialties: ["Family Medicine", "Women's Health"],
      languages: ["English", "Mandarin"],
      bio_md:
        "Dr Tan is dedicated to family medicine with a focus on preventive care and women's health. With over 15 years of experience, she believes in building long-term relationships with her patients.",
      photo_url: "/images/doctors/dr-tan.jpg",
      consultation_fee: 45.0,
      gender: "female",
      years_experience: 15,
      status: "active",
    },
  });

  const drKumar = await prisma.doctor.upsert({
    where: { slug: "dr-arun-kumar" },
    update: {},
    create: {
      full_name: "Dr Arun Kumar",
      slug: "dr-arun-kumar",
      credentials: "MBBS, MRCGP",
      specialties: ["Family Medicine", "Chronic Disease Management"],
      languages: ["English", "Tamil"],
      bio_md:
        "Dr Kumar focuses on chronic disease management and patient education. He has a special interest in diabetes and hypertension management.",
      photo_url: "/images/doctors/dr-kumar.jpg",
      consultation_fee: 40.0,
      gender: "male",
      years_experience: 12,
      status: "active",
    },
  });

  const drLim = await prisma.doctor.upsert({
    where: { slug: "dr-lim-jia-hui" },
    update: {},
    create: {
      full_name: "Dr Lim Jia Hui",
      slug: "dr-lim-jia-hui",
      credentials: "MBBS (Singapore), Dip Dermatology",
      specialties: ["Family Medicine", "Dermatology"],
      languages: ["English", "Mandarin", "Hokkien"],
      bio_md:
        "Dr Lim has a special interest in dermatology and minor surgical procedures. She provides comprehensive care for patients of all ages.",
      photo_url: "/images/doctors/dr-lim.jpg",
      consultation_fee: 50.0,
      gender: "female",
      years_experience: 8,
      status: "active",
    },
  });

  // M:N links for Clinics-Services
  await prisma.clinicsOnServices.upsert({
    where: { clinicId_serviceId: { clinicId: clinicCentral.id, serviceId: gpConsult.id } },
    update: {},
    create: { 
      clinicId: clinicCentral.id, 
      serviceId: gpConsult.id,
      priority: 1,
      notes: "Available daily with extended hours on Wednesdays"
    },
  });

  await prisma.clinicsOnServices.upsert({
    where: { clinicId_serviceId: { clinicId: clinicCentral.id, serviceId: vaccination.id } },
    update: {},
    create: { 
      clinicId: clinicCentral.id, 
      serviceId: vaccination.id,
      priority: 2,
      notes: "All vaccines available including COVID-19 boosters"
    },
  });

  await prisma.clinicsOnServices.upsert({
    where: { clinicId_serviceId: { clinicId: clinicCentral.id, serviceId: healthScreening.id } },
    update: {},
    create: { 
      clinicId: clinicCentral.id, 
      serviceId: healthScreening.id,
      priority: 3,
      notes: "Comprehensive packages available"
    },
  });

  await prisma.clinicsOnServices.upsert({
    where: { clinicId_serviceId: { clinicId: clinicEast.id, serviceId: gpConsult.id } },
    update: {},
    create: { 
      clinicId: clinicEast.id, 
      serviceId: gpConsult.id,
      priority: 1,
    },
  });

  await prisma.clinicsOnServices.upsert({
    where: { clinicId_serviceId: { clinicId: clinicEast.id, serviceId: chronicCare.id } },
    update: {},
    create: { 
      clinicId: clinicEast.id, 
      serviceId: chronicCare.id,
      priority: 2,
      notes: "Special focus on diabetes management"
    },
  });

  await prisma.clinicsOnServices.upsert({
    where: { clinicId_serviceId: { clinicId: clinicWest.id, serviceId: gpConsult.id } },
    update: {},
    create: { 
      clinicId: clinicWest.id, 
      serviceId: gpConsult.id,
      priority: 1,
    },
  });

  await prisma.clinicsOnServices.upsert({
    where: { clinicId_serviceId: { clinicId: clinicWest.id, serviceId: vaccination.id } },
    update: {},
    create: { 
      clinicId: clinicWest.id, 
      serviceId: vaccination.id,
      priority: 2,
    },
  });

  // M:N links for Clinics-Doctors
  await prisma.clinicsOnDoctors.upsert({
    where: { clinicId_doctorId: { clinicId: clinicCentral.id, doctorId: drTan.id } },
    update: {},
    create: { 
      clinicId: clinicCentral.id, 
      doctorId: drTan.id,
      schedule_json: {
        mon: { morning: true, afternoon: true, evening: false },
        tue: { morning: true, afternoon: true, evening: false },
        wed: { morning: true, afternoon: true, evening: true },
        thu: { morning: true, afternoon: true, evening: false },
        fri: { morning: true, afternoon: true, evening: false },
        sat: { morning: true, afternoon: false, evening: false },
      },
      consultation_fee: 45.0,
    },
  });

  await prisma.clinicsOnDoctors.upsert({
    where: { clinicId_doctorId: { clinicId: clinicEast.id, doctorId: drKumar.id } },
    update: {},
    create: { 
      clinicId: clinicEast.id, 
      doctorId: drKumar.id,
      schedule_json: {
        mon: { morning: true, afternoon: true, evening: false },
        tue: { morning: true, afternoon: true, evening: false },
        wed: { morning: false, afternoon: true, evening: true },
        thu: { morning: true, afternoon: true, evening: false },
        fri: { morning: true, afternoon: true, evening: false },
        sat: { morning: true, afternoon: false, evening: false },
      },
      consultation_fee: 40.0,
    },
  });

  await prisma.clinicsOnDoctors.upsert({
    where: { clinicId_doctorId: { clinicId: clinicWest.id, doctorId: drLim.id } },
    update: {},
    create: { 
      clinicId: clinicWest.id, 
      doctorId: drLim.id,
      schedule_json: {
        mon: { morning: true, afternoon: true, evening: false },
        tue: { morning: true, afternoon: true, evening: false },
        wed: { morning: true, afternoon: true, evening: false },
        thu: { morning: true, afternoon: true, evening: false },
        fri: { morning: true, afternoon: true, evening: false },
        sat: { morning: true, afternoon: false, evening: false },
      },
      consultation_fee: 50.0,
    },
  });

  // M:N links for Services-Schemes
  await prisma.servicesOnSchemes.upsert({
    where: { serviceId_schemeId: { serviceId: gpConsult.id, schemeId: chas.id } },
    update: {},
    create: { 
      serviceId: gpConsult.id, 
      schemeId: chas.id,
      coverage_details: "Subsidies available for eligible Singapore Citizens"
    },
  });

  await prisma.servicesOnSchemes.upsert({
    where: { serviceId_schemeId: { serviceId: gpConsult.id, schemeId: medisave.id } },
    update: {},
    create: { 
      serviceId: gpConsult.id, 
      schemeId: medisave.id,
      coverage_details: "MediSave can be used for part of the consultation fee"
    },
  });

  await prisma.servicesOnSchemes.upsert({
    where: { serviceId_schemeId: { serviceId: vaccination.id, schemeId: chas.id } },
    update: {},
    create: { 
      serviceId: vaccination.id, 
      schemeId: chas.id,
      coverage_details: "Subsidies available for recommended vaccinations"
    },
  });

  await prisma.servicesOnSchemes.upsert({
    where: { serviceId_schemeId: { serviceId: corporateHealth.id, schemeId: panelCompany.id } },
    update: {},
    create: { 
      serviceId: corporateHealth.id, 
      schemeId: panelCompany.id,
      coverage_details: "Corporate packages can be customized based on company requirements"
    },
  });

  // M:N links for Clinics-Schemes
  await prisma.clinicsOnSchemes.upsert({
    where: { clinicId_schemeId: { clinicId: clinicCentral.id, schemeId: chas.id } },
    update: {},
    create: { 
      clinicId: clinicCentral.id, 
      schemeId: chas.id,
      notes: "Full CHAS support available"
    },
  });

  await prisma.clinicsOnSchemes.upsert({
    where: { clinicId_schemeId: { clinicId: clinicEast.id, schemeId: medisave.id } },
    update: {},
    create: { 
      clinicId: clinicEast.id, 
      schemeId: medisave.id,
      notes: "MediSave accepted for eligible services"
    },
  });

  // News
  await prisma.news.upsert({
    where: { slug: "healthier-sg-update" },
    update: {},
    create: {
      title: "Healthier SG Programme Update",
      slug: "healthier-sg-update",
      summary: "Latest information about Healthier SG participation.",
      content_md:
        "We are pleased to announce that our Central and West clinics are now participating in the Healthier SG programme. This means eligible residents can enjoy more comprehensive and coordinated care.\n\nTo enroll, simply visit our clinic with your NRIC and we'll guide you through the process. As a Healthier SG participant, you'll receive:\n\n- Free recommended health screenings\n- Subsidies for chronic condition management\n- Personalized health plans\n\nFor more information, please contact our clinics.",
      published_at: new Date(),
      author: "Editorial Team",
      tags: ["HealthierSG", "Programme", "Subsidies"],
      featured_image_url: "/images/news/healthier-sg.jpg",
      status: "published",
      meta_title: "Healthier SG Programme Update | My Family Clinic",
      meta_description: "Learn about our participation in the Healthier SG programme and how you can benefit.",
      reading_time: 2,
    },
  });

  await prisma.news.upsert({
    where: { slug: "influenza-vaccination-campaign" },
    update: {},
    create: {
      title: "Annual Influenza Vaccination Campaign",
      slug: "influenza-vaccination-campaign",
      summary: "Protect yourself and your family with our annual flu vaccination.",
      content_md:
        "It's time for our annual influenza vaccination campaign! Flu vaccines are now available at all our clinics.\n\nWhy get vaccinated?\n- Protect yourself from the flu\n- Reduce the risk of complications\n- Protect vulnerable family members\n\nSpecial promotion: Get 10% off when you vaccinate as a family (3 or more members).\n\nWalk-ins are welcome, but appointments are recommended to avoid waiting.",
      published_at: new Date(),
      author: "Nursing Team",
      tags: ["Vaccination", "Promotion", "Influenza"],
      featured_image_url: "/images/news/flu-vaccine.jpg",
      status: "published",
      meta_title: "Annual Influenza Vaccination Campaign | My Family Clinic",
      meta_description: "Protect yourself with our annual flu vaccination. Special family promotion available.",
      reading_time: 1,
    },
  });

  // Accreditation
  await prisma.accreditation.upsert({
    where: { id: "accreditation-singapore-health" },
    update: {},
    create: {
      id: "accreditation-singapore-health",
      name: "Singapore Health Promotion Board",
      description_md:
        "Accredited clinic under the Health Promotion Board's programmes.",
      badge_url: "/images/accreditations/hpb-badge.png",
      link_url: "https://www.hpb.gov.sg/",
      issued_by: "Health Promotion Board",
      display_order: 1,
    },
  });

  await prisma.accreditation.upsert({
    where: { id: "accreditation-singapore-medical-council" },
    update: {},
    create: {
      id: "accreditation-singapore-medical-council",
      name: "Singapore Medical Council",
      description_md:
        "All our doctors are registered with the Singapore Medical Council.",
      badge_url: "/images/accreditations/smc-badge.png",
      link_url: "https://www.smc.gov.sg/",
      issued_by: "Singapore Medical Council",
      display_order: 2,
    },
  });

  // Admin and audit
  const admin = await prisma.adminUser.upsert({
    where: { email: "admin@myfamilyclinic.com.sg" },
    update: {},
    create: {
      email: "admin@myfamilyclinic.com.sg",
      name: "Site Admin",
      role: AdminRole.ADMIN,
      bio_md: "System administrator for My Family Clinic website.",
    },
  });

  const editor = await prisma.adminUser.upsert({
    where: { email: "editor@myfamilyclinic.com.sg" },
    update: {},
    create: {
      email: "editor@myfamilyclinic.com.sg",
      name: "Content Editor",
      role: AdminRole.EDITOR,
      bio_md: "Content editor for My Family Clinic website.",
    },
  });

  // Sample audit logs
  await prisma.auditLog.create({
    data: {
      entity: "Clinic",
      entityId: clinicCentral.id,
      actorUserId: admin.id,
      action: "CREATE",
      changes_json: { 
        name: clinicCentral.name, 
        slug: clinicCentral.slug,
        address: clinicCentral.address,
        district: clinicCentral.district
      },
      reason: "Initial seed data",
      ipAddress: "127.0.0.1",
      userAgent: "Seed Script",
    },
  });

  await prisma.auditLog.create({
    data: {
      entity: "Service",
      entityId: gpConsult.id,
      actorUserId: admin.id,
      action: "CREATE",
      changes_json: { 
        name: gpConsult.name, 
        slug: gpConsult.slug,
        category: gpConsult.category
      },
      reason: "Initial seed data",
      ipAddress: "127.0.0.1",
      userAgent: "Seed Script",
    },
  });

  // Sample analytics events
  await prisma.analyticsEvent.create({
    data: {
      event_id: "click_locate_cta",
      payload_json: { 
        page: "homepage", 
        position: "header" 
      },
      session_id: "session_12345",
      ip_address: "192.168.1.1",
      user_agent: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36",
    },
  });

  await prisma.analyticsEvent.create({
    data: {
      event_id: "locator_search",
      payload_json: { 
        method: "geolocation", 
        filters: { services: ["gp-consultation"] }
      },
      session_id: "session_12345",
      ip_address: "192.168.1.1",
      user_agent: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36",
    },
  });

  console.log("Seed completed successfully.");
}

main()
  .catch((e) => {
    console.error("Seed error:", e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

## File: package.json (Additions)

```json
{
  "scripts": {
    "db:seed": "ts-node --compiler-options {\"module\":\"CommonJS\"} prisma/seed.ts"
  }
}
```

## How to Run

1. Ensure `DATABASE_URL` points to your Supabase Postgres instance.
2. Generate Prisma client and run migrations:
   ```bash
   npm run postinstall
   npm run db:generate
   npm run db:migrate
   ```
3. Seed the database:
   ```bash
   npm run db:seed
   ```

## Key Enhancements

1. **Enhanced Schema**:
   - Added more specific enums for better type safety
   - Added additional fields to support production features
   - Improved indexing strategy for performance optimization
   - Added AnalyticsEvent model for tracking user interactions

2. **Comprehensive Seed Data**:
   - More realistic and comprehensive seed data
   - Better representation of real-world scenarios
   - Sample audit logs and analytics events
   - More detailed relationships between entities

3. **Performance Optimization**:
   - Trigram indexes for fuzzy search
   - PostGIS support for geolocation queries
   - Additional indexes for common query patterns
   - Optimized data types for better performance

4. **Production-Ready Features**:
   - Audit logging with detailed information
   - Analytics tracking for user interactions
   - Status fields for content management
   - Enhanced relationships with additional metadata

