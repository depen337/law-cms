# Legal Case Management System - Project Handoff

## Project Overview
AI-powered legal case management system with document analysis, timeline visualization, and PDF viewing capabilities. This document contains the complete handoff from Appsmith prototype to Next.js production application.

## 🎯 Core Functionality

### Completed in Appsmith Prototype
- ✅ Cases dashboard with data table
- ✅ New case creation modal and form
- ✅ Database CRUD operations
- ✅ PostgreSQL schema and sample data
- ✅ Case listing and filtering

### Target Features for Next.js App
- 📊 **Dashboard**: Cases table with search, filter, and pagination
- 📄 **Case Details**: Individual case view with document timeline
- 📁 **Document Management**: Upload, view, and organize case documents
- 🔍 **PDF Viewer**: In-browser document viewing
- 🤖 **AI Analysis**: Automated document summarization and fact extraction
- 📱 **Responsive UI**: Modern, professional legal interface

## 🗄️ Database Schema (Neon PostgreSQL)

### Cases Table
```sql
CREATE TABLE cases (
  case_id SERIAL PRIMARY KEY,
  case_name VARCHAR(255) NOT NULL,
  client_name VARCHAR(255) NOT NULL,
  status VARCHAR(50) DEFAULT 'Active',
  date_opened TIMESTAMP DEFAULT NOW(),
  summary TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

### Documents Table
```sql
CREATE TABLE documents (
  document_id SERIAL PRIMARY KEY,
  case_id INTEGER REFERENCES cases(case_id),
  document_name VARCHAR(255) NOT NULL,
  s3_path TEXT,
  event_date DATE,
  document_type VARCHAR(100),
  ai_analysis JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);
```

### Sample Data
```sql
-- Test cases
INSERT INTO cases (case_name, client_name, summary) VALUES 
('Contract Dispute Resolution', 'ABC Corporation', 'Breach of service contract terms'),
('Personal Injury Claim', 'John Smith', 'Workplace accident compensation case'),
('Trademark Registration', 'Tech Startup Inc', 'Software trademark filing and protection'),
('Real Estate Transaction', 'Property Holdings LLC', 'Commercial property purchase agreement'),
('Employment Law Case', 'Jane Doe', 'Wrongful termination and discrimination claim');

-- Sample documents (for testing)
INSERT INTO documents (case_id, document_name, document_type, event_date) VALUES 
(1, 'Initial_Contract_2024.pdf', 'Contract', '2024-01-15'),
(1, 'Amendment_Letter.pdf', 'Amendment', '2024-03-20'),
(2, 'Medical_Report.pdf', 'Medical', '2024-02-10'),
(2, 'Witness_Statement.pdf', 'Statement', '2024-02-25'),
(3, 'Trademark_Application.pdf', 'Application', '2024-01-05');
```

### AI Analysis JSON Structure
```json
{
  "summary": "Brief document summary and key points",
  "key_facts": [
    "Important fact extracted from document",
    "Another significant detail",
    "Legal precedent or citation found"
  ],
  "legal_citations": [
    "Case Name v. Defendant, Court Year",
    "Relevant statute or regulation reference"
  ],
  "extracted_tables": [
    "| Column 1 | Column 2 | Column 3 |\n|----------|----------|----------|\n| Data     | Data     | Data     |"
  ],
  "document_type": "contract|motion|brief|evidence|correspondence",
  "confidence_score": 0.85,
  "processing_timestamp": "2024-08-06T14:30:00Z"
}
```

## 🚀 Next.js Implementation Guide

### Recommended Tech Stack
```json
{
  "framework": "Next.js 14+ with App Router",
  "database": "Neon PostgreSQL",
  "orm": "Prisma",
  "ui": "Tailwind CSS + shadcn/ui",
  "file_storage": "AWS S3 or Vercel Blob Storage",
  "pdf_viewer": "react-pdf or @react-pdf-viewer/core",
  "ai": "Google Gemini API or OpenAI GPT-4",
  "deployment": "Vercel",
  "auth": "NextAuth.js (for future user management)"
}
```

### Environment Variables
```env
# Database (Neon)
DATABASE_URL="postgresql://username:password@ep-xyz.us-east-1.aws.neon.tech/dbname?sslmode=require"
DIRECT_URL="postgresql://username:password@ep-xyz.us-east-1.aws.neon.tech/dbname?sslmode=require"

# AI Processing
GEMINI_API_KEY="your-google-gemini-api-key"
# OR
OPENAI_API_KEY="your-openai-api-key"

# File Storage (Choose one option)
# Option 1: AWS S3
AWS_ACCESS_KEY_ID="your-aws-access-key"
AWS_SECRET_ACCESS_KEY="your-aws-secret-key"
S3_BUCKET_NAME="legal-cms-documents"
AWS_REGION="us-east-1"

# Option 2: Vercel Blob (Easier setup)
BLOB_READ_WRITE_TOKEN="vercel_blob_token"

# App Configuration
NEXT_PUBLIC_APP_URL="http://localhost:3000"
NEXT_PUBLIC_APP_NAME="Legal Case Management System"
```

### Prisma Schema
```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")
}

model Case {
  id          Int       @id @default(autoincrement()) @map("case_id")
  caseName    String    @map("case_name")
  clientName  String    @map("client_name") 
  status      String    @default("Active")
  dateOpened  DateTime  @default(now()) @map("date_opened")
  summary     String?
  createdAt   DateTime  @default(now()) @map("created_at")
  updatedAt   DateTime  @updatedAt @map("updated_at")
  documents   Document[]
  
  @@map("cases")
}

model Document {
  id           Int       @id @default(autoincrement()) @map("document_id")
  caseId       Int       @map("case_id")
  documentName String    @map("document_name")
  s3Path       String?   @map("s3_path")
  eventDate    DateTime? @map("event_date")
  documentType String?   @map("document_type")
  aiAnalysis   Json?     @map("ai_analysis")
  createdAt    DateTime  @default(now()) @map("created_at")
  case         Case      @relation(fields: [caseId], references: [id], onDelete: Cascade)
  
  @@map("documents")
}
```

### Project Structure
```
legal-cms/
├── app/
│   ├── dashboard/
│   │   └── page.tsx                 # Main cases dashboard
│   ├── cases/
│   │   └── [id]/
│   │       ├── page.tsx             # Case details view
│   │       └── loading.tsx          # Loading state
│   ├── api/
│   │   ├── cases/
│   │   │   ├── route.ts             # GET /api/cases, POST /api/cases
│   │   │   └── [id]/
│   │   │       ├── route.ts         # GET /api/cases/[id]
│   │   │       └── documents/
│   │   │           └── route.ts     # GET /api/cases/[id]/documents
│   │   ├── documents/
│   │   │   ├── route.ts             # POST /api/documents (upload)
│   │   │   └── [id]/
│   │   │       └── route.ts         # GET /api/documents/[id]
│   │   └── ai/
│   │       ├── analyze/
│   │       │   └── route.ts         # POST /api/ai/analyze
│   │       └── webhook/
│   │           └── route.ts         # POST /api/ai/webhook (n8n)
│   ├── components/
│   │   ├── ui/                      # shadcn/ui components
│   │   ├── dashboard/
│   │   │   ├── CasesTable.tsx
│   │   │   ├── NewCaseModal.tsx
│   │   │   └── CaseFilters.tsx
│   │   ├── cases/
│   │   │   ├── CaseHeader.tsx
│   │   │   ├── DocumentTimeline.tsx
│   │   │   ├── PDFViewer.tsx
│   │   │   ├── AIAnalysisPanel.tsx
│   │   │   └── DocumentUpload.tsx
│   │   └── layout/
│   │       ├── Navigation.tsx
│   │       └── Header.tsx
│   ├── lib/
│   │   ├── db.ts                    # Prisma client
│   │   ├── s3.ts                    # File storage utilities
│   │   ├── ai.ts                    # AI processing functions
│   │   └── utils.ts                 # General utilities
│   ├── types/
│   │   └── index.ts                 # TypeScript definitions
│   ├── globals.css
│   ├── layout.tsx
│   └── page.tsx                     # Landing/redirect to dashboard
├── prisma/
│   ├── schema.prisma
│   └── migrations/
├── public/
│   └── icons/
└── package.json
```

## 📋 Implementation Phases

### Phase 1: Setup & Basic CRUD (Week 1)
```bash
# Quick start commands
npx create-next-app@latest legal-cms --typescript --tailwind --eslint --app
cd legal-cms

# Install dependencies
npm install prisma @prisma/client
npm install @radix-ui/react-dialog @radix-ui/react-table
npm install lucide-react class-variance-authority clsx tailwind-merge

# Setup Prisma
npx prisma init
# Copy schema to prisma/schema.prisma
# Add DATABASE_URL to .env.local
npx prisma generate
npx prisma db push

# Setup shadcn/ui
npx shadcn-ui@latest init
npx shadcn-ui@latest add table button dialog input label textarea select
```

**Deliverables:**
- ✅ Project setup and database connection
- ✅ Dashboard with cases table
- ✅ New case creation modal
- ✅ Case detail view (basic)
- ✅ Responsive layout

### Phase 2: Document Management (Week 2)
```bash
# Additional dependencies for file handling
npm install @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
# OR for Vercel Blob
npm install @vercel/blob

# PDF viewing
npm install react-pdf pdfjs-dist
npm install -D @types/react-pdf
```

**Deliverables:**
- ✅ File upload functionality
- ✅ S3/Blob storage integration
- ✅ PDF viewer component
- ✅ Document timeline UI
- ✅ Document metadata management

### Phase 3: AI Integration (Week 3)
```bash
# AI processing
npm install @google/generative-ai
# OR
npm install openai

# Additional utilities
npm install date-fns recharts (for charts/analytics)
```

**Deliverables:**
- ✅ AI document analysis
- ✅ Gemini/OpenAI integration
- ✅ Analysis results display
- ✅ Background processing
- ✅ Analysis confidence scoring

### Phase 4: Polish & Deploy (Week 4)
```bash
# Deployment preparation
npm install @vercel/analytics @vercel/speed-insights

# Performance and monitoring
npm install @sentry/nextjs (optional)
```

**Deliverables:**
- ✅ Production deployment
- ✅ Performance optimization
- ✅ Error handling and logging
- ✅ User feedback and testing
- ✅ Documentation

## 🔧 Key Implementation Details

### API Route Examples

#### Cases API (app/api/cases/route.ts)
```typescript
import { NextRequest, NextResponse } from 'next/server';
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url);
    const page = parseInt(searchParams.get('page') || '1');
    const limit = parseInt(searchParams.get('limit') || '10');
    const search = searchParams.get('search') || '';
    
    const where = search ? {
      OR: [
        { caseName: { contains: search, mode: 'insensitive' } },
        { clientName: { contains: search, mode: 'insensitive' } }
      ]
    } : {};

    const [cases, total] = await Promise.all([
      prisma.case.findMany({
        where,
        orderBy: { createdAt: 'desc' },
        skip: (page - 1) * limit,
        take: limit,
        include: {
          _count: { select: { documents: true } }
        }
      }),
      prisma.case.count({ where })
    ]);

    return NextResponse.json({
      cases,
      pagination: {
        page,
        limit,
        total,
        pages: Math.ceil(total / limit)
      }
    });
  } catch (error) {
    console.error('Failed to fetch cases:', error);
    return NextResponse.json(
      { error: 'Failed to fetch cases' }, 
      { status: 500 }
    );
  }
}

export async function POST(request: NextRequest) {
  try {
    const { caseName, clientName, summary } = await request.json();
    
    const newCase = await prisma.case.create({
      data: {
        caseName,
        clientName,
        summary,
        status: 'Active'
      }
    });

    return NextResponse.json(newCase, { status: 201 });
  } catch (error) {
    console.error('Failed to create case:', error);
    return NextResponse.json(
      { error: 'Failed to create case' }, 
      { status: 500 }
    );
  }
}
```

#### AI Analysis API (app/api/ai/analyze/route.ts)
```typescript
import { NextRequest, NextResponse } from 'next/server';
import { GoogleGenerativeAI } from '@google/generative-ai';
import { PrismaClient } from '@prisma/client';

const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY!);
const prisma = new PrismaClient();

export async function POST(request: NextRequest) {
  try {
    const { documentId, documentContent, documentType } = await request.json();
    
    const model = genAI.getGenerativeModel({ model: "gemini-1.5-flash" });
    
    const prompt = `
    Analyze this legal document and extract the following information in JSON format:
    
    1. summary: A brief 2-3 sentence summary of the document
    2. key_facts: Array of important facts extracted from the document
    3. legal_citations: Array of any legal citations, case law, or statutes mentioned
    4. extracted_tables: Array of any tables or structured data (in markdown format)
    5. document_type: Classification of document type
    6. confidence_score: Your confidence in the analysis (0-1)
    
    Document content:
    ${documentContent}
    
    Return only valid JSON without any markdown formatting.
    `;

    const result = await model.generateContent(prompt);
    const analysisText = result.response.text();
    
    let analysis;
    try {
      analysis = JSON.parse(analysisText);
    } catch (parseError) {
      // Fallback if AI doesn't return valid JSON
      analysis = {
        summary: "Analysis completed but formatting error occurred",
        key_facts: [],
        legal_citations: [],
        extracted_tables: [],
        document_type: documentType || "unknown",
        confidence_score: 0.5,
        raw_response: analysisText
      };
    }

    // Save analysis to database
    await prisma.document.update({
      where: { id: documentId },
      data: { 
        aiAnalysis: analysis,
        updatedAt: new Date()
      }
    });

    return NextResponse.json({ analysis });
  } catch (error) {
    console.error('AI analysis failed:', error);
    return NextResponse.json(
      { error: 'AI analysis failed' }, 
      { status: 500 }
    );
  }
}
```

### Component Examples

#### Cases Table Component
```typescript
// components/dashboard/CasesTable.tsx
'use client';

import { useState, useEffect } from 'react';
import { useRouter } from 'next/navigation';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Search, Plus } from 'lucide-react';

interface Case {
  id: number;
  caseName: string;
  clientName: string;
  status: string;
  dateOpened: string;
  _count: { documents: number };
}

export default function CasesTable() {
  const [cases, setCases] = useState<Case[]>([]);
  const [search, setSearch] = useState('');
  const [loading, setLoading] = useState(true);
  const router = useRouter();

  useEffect(() => {
    fetchCases();
  }, [search]);

  const fetchCases = async () => {
    try {
      const params = new URLSearchParams({
        ...(search && { search }),
        limit: '10'
      });
      
      const response = await fetch(`/api/cases?${params}`);
      const data = await response.json();
      setCases(data.cases);
    } catch (error) {
      console.error('Failed to fetch cases:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="space-y-4">
      <div className="flex justify-between items-center">
        <div className="relative w-72">
          <Search className="absolute left-2 top-2.5 h-4 w-4 text-muted-foreground" />
          <Input
            placeholder="Search cases..."
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="pl-8"
          />
        </div>
        <Button onClick={() => router.push('/cases/new')}>
          <Plus className="h-4 w-4 mr-2" />
          New Case
        </Button>
      </div>

      <div className="border rounded-lg">
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead>Case Name</TableHead>
              <TableHead>Client</TableHead>
              <TableHead>Status</TableHead>
              <TableHead>Documents</TableHead>
              <TableHead>Date Opened</TableHead>
              <TableHead>Actions</TableHead>
            </TableRow>
          </TableHeader>
          <TableBody>
            {cases.map((case_) => (
              <TableRow 
                key={case_.id}
                className="cursor-pointer hover:bg-muted/50"
                onClick={() => router.push(`/cases/${case_.id}`)}
              >
                <TableCell className="font-medium">
                  {case_.caseName}
                </TableCell>
                <TableCell>{case_.clientName}</TableCell>
                <TableCell>
                  <span className={`px-2 py-1 rounded-full text-xs ${
                    case_.status === 'Active' 
                      ? 'bg-green-100 text-green-800' 
                      : 'bg-gray-100 text-gray-800'
                  }`}>
                    {case_.status}
                  </span>
                </TableCell>
                <TableCell>{case_._count.documents}</TableCell>
                <TableCell>
                  {new Date(case_.dateOpened).toLocaleDateString()}
                </TableCell>
                <TableCell>
                  <Button 
                    variant="ghost" 
                    size="sm"
                    onClick={(e) => {
                      e.stopPropagation();
                      router.push(`/cases/${case_.id}`);
                    }}
                  >
                    View
                  </Button>
                </TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </div>
    </div>
  );
}
```

## 🔄 Migration Notes

### From Appsmith Prototype
- **Working Queries**: All SQL queries are tested and functional
- **Database Schema**: Validated with real data
- **UI Patterns**: Modal forms, table interactions, navigation flows
- **Business Logic**: Case creation, filtering, basic CRUD operations

### Key Improvements in Next.js Version
- **Performance**: Static generation, optimized loading
- **Type Safety**: Full TypeScript integration
- **Modern UI**: Component-based, responsive design
- **Scalability**: API routes, middleware, caching
- **SEO**: Server-side rendering, meta tags
- **Deployment**: Vercel integration, environment management

## 🚢 Deployment Checklist

### Pre-deployment
- [ ] Environment variables configured in Vercel
- [ ] Neon database provisioned and migrated
- [ ] AI API keys obtained and tested
- [ ] S3 bucket created and configured (if using AWS)
- [ ] Domain name configured (optional)

### Production Setup
```bash
# Build and test locally
npm run build
npm run start

# Deploy to Vercel
vercel --prod

# Run database migrations
npx prisma migrate deploy
```

### Post-deployment
- [ ] Test all functionality in production
- [ ] Monitor performance and errors
- [ ] Set up analytics (Vercel Analytics)
- [ ] Configure monitoring (Sentry, etc.)
- [ ] Create user documentation

## 📚 Additional Resources

### Documentation Links
- [Next.js App Router](https://nextjs.org/docs/app)
- [Prisma with Neon](https://www.prisma.io/docs/guides/database/neon)
- [shadcn/ui Components](https://ui.shadcn.com/)
- [Vercel Deployment](https://vercel.com/docs)
- [Google Gemini API](https://ai.google.dev/)

### Useful Packages
```json
{
  "pdf-parsing": ["pdf-parse", "pdf2pic"],
  "date-utilities": ["date-fns", "dayjs"],
  "icons": ["lucide-react", "@heroicons/react"],
  "forms": ["react-hook-form", "@hookform/resolvers"],
  "validation": ["zod", "yup"],
  "charts": ["recharts", "chart.js"],
  "notifications": ["sonner", "react-hot-toast"]
}
```

## 🎯 Success Metrics

### Technical Goals
- [ ] Page load time < 2 seconds
- [ ] 100% TypeScript coverage
- [ ] 90%+ Lighthouse score
- [ ] Zero console errors
- [ ] Responsive on all devices

### Functional Goals
- [ ] Create case in < 30 seconds
- [ ] Upload and analyze document in < 2 minutes
- [ ] Search through cases instantly
- [ ] PDF viewing works in all browsers
- [ ] AI analysis accuracy > 80%

---

*This handoff document contains everything needed to rebuild the Legal Case Management System in Next.js. The Appsmith prototype provided valuable insights into user workflows and database design that inform this production-ready implementation.*

**Estimated Timeline**: 4 weeks for full implementation
**Complexity Level**: Intermediate to Advanced
**Primary Skills Required**: Next.js, TypeScript, PostgreSQL, AI Integration