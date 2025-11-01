# 🚀 DACON: RAID - Replit 개발 가이드

**프로젝트명**: DACON: RAID (팀 빌딩 시스템)  
**개발 플랫폼**: Replit  
**예상 개발 기간**: 2.5개월 (MVP)  
**작성일**: 2025년 11월 1일

---

## 📋 목차

1. [프로젝트 개요](#1-프로젝트-개요)
2. [Replit 프로젝트 설정](#2-replit-프로젝트-설정)
3. [프론트엔드 아키텍처](#3-프론트엔드-아키텍처)
4. [백엔드 아키텍처](#4-백엔드-아키텍처)
5. [데이터베이스 설계](#5-데이터베이스-설계)
6. [API 엔드포인트 설계](#6-api-엔드포인트-설계)
7. [주요 기능 구현 가이드](#7-주요-기능-구현-가이드)
8. [개발 로드맵](#8-개발-로드맵)
9. [배포 및 운영](#9-배포-및-운영)
10. [문제 해결 가이드](#10-문제-해결-가이드)

---

## 1. 프로젝트 개요

### 1.1 제품 설명
DACON: RAID는 데이콘 플랫폼 사용자들이 효과적으로 팀을 구성하고 협업할 수 있도록 지원하는 '가이드 마켓플레이스' 기반의 팀 빌딩 시스템입니다.

### 1.2 핵심 가치
- **사용자 주체성**: 팀 구성에 대한 완전한 통제권
- **데이터 기반 투명성**: 검증 가능한 역량 지표와 적합도 점수
- **부정적 경험 최소화**: 명확한 규칙과 신뢰 시스템
- **다학제적 협업**: 다양한 역할군 매칭 지원

### 1.3 Replit 선택 이유

#### ⚡ 빠른 개발 속도 (20% 시간 단축)
- Zero Configuration으로 즉시 시작
- Hot Reload로 실시간 미리보기
- 원클릭 배포로 CI/CD 불필요

#### 👥 효율적인 협업
- Multiplayer Mode로 실시간 페어 프로그래밍
- Live Code Review
- URL 공유만으로 베타 테스터에게 즉시 배포

#### 💰 비용 절감 (초기 투자 65% 감소)
- IDE + 호스팅 + 협업 도구 + DB 올인원
- Neon PostgreSQL 무료 티어 활용
- 종량제 호스팅으로 초기 고정비 없음

### 1.4 기술 스택 요약

```
Frontend:  React 19 + TypeScript + Vite
Styling:   Tailwind CSS + Lucide Icons
Backend:   Express.js + Node.js
Database:  Neon PostgreSQL (Serverless)
ORM:       Prisma
Auth:      Replit Auth (Google OAuth)
Hosting:   Replit Deployments
```

---

## 2. Replit 프로젝트 설정

### 2.1 새 프로젝트 생성

#### Step 1: Replit에서 프로젝트 생성
```
1. Replit 접속 (https://replit.com)
2. "Create Repl" 클릭
3. 템플릿 선택: "React + Vite" ⭐ 권장
4. 프로젝트명 입력: "dacon-raid"
5. Public으로 설정 (팀 협업 및 베타 테스트 용이)
6. "Create Repl" 클릭
```

#### Step 2: TypeScript 설정
Replit의 React + Vite 템플릿은 기본적으로 JavaScript입니다. TypeScript로 전환하려면:

```bash
# TypeScript 및 타입 정의 설치
npm install --save-dev typescript @types/react @types/react-dom @types/node

# tsconfig.json 생성
npm create vite@latest . -- --template react-ts
```

**또는** 처음부터 TypeScript 템플릿 선택:
```
템플릿 선택 시 "React TypeScript" 선택
```

### 2.2 프로젝트 구조 설정

```
dacon-raid/
├── client/                     # 프론트엔드
│   ├── src/
│   │   ├── components/        # UI 컴포넌트
│   │   │   ├── ui/           # 기본 UI (Button, Card 등)
│   │   │   ├── Header.tsx
│   │   │   └── RaidCard.tsx
│   │   ├── contexts/         # React Context
│   │   │   ├── AuthContext.tsx
│   │   │   └── DataContext.tsx
│   │   ├── pages/            # 페이지 컴포넌트
│   │   │   ├── HomePage.tsx
│   │   │   ├── RaidDetailPage.tsx
│   │   │   ├── ProfilePage.tsx
│   │   │   ├── CreateRaidPage.tsx
│   │   │   └── DashboardPage.tsx
│   │   ├── types.ts          # TypeScript 타입 정의
│   │   ├── constants.ts      # 상수 정의
│   │   ├── App.tsx           # 메인 앱 컴포넌트
│   │   ├── main.tsx          # 엔트리 포인트
│   │   └── index.css         # 전역 스타일
│   ├── index.html
│   ├── package.json
│   ├── tsconfig.json
│   └── vite.config.ts
├── server/                    # 백엔드
│   ├── src/
│   │   ├── routes/           # API 라우트
│   │   ├── controllers/      # 비즈니스 로직
│   │   ├── middleware/       # Express 미들웨어
│   │   ├── utils/            # 유틸리티 함수
│   │   └── index.ts          # Express 서버 진입점
│   ├── prisma/
│   │   └── schema.prisma     # Prisma 스키마
│   ├── package.json
│   └── tsconfig.json
├── .replit                    # Replit 설정
├── replit.nix                 # Nix 환경 설정
└── README.md
```

### 2.3 필수 패키지 설치

#### 프론트엔드 (`client/package.json`)
```json
{
  "name": "dacon-raid-client",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^19.2.0",
    "react-dom": "^19.2.0",
    "react-router-dom": "^7.9.5",
    "lucide-react": "^0.548.0"
  },
  "devDependencies": {
    "@types/node": "^22.14.0",
    "@types/react": "^18.3.0",
    "@types/react-dom": "^18.3.0",
    "@vitejs/plugin-react": "^5.0.0",
    "autoprefixer": "^10.4.20",
    "postcss": "^8.4.47",
    "tailwindcss": "^3.4.15",
    "typescript": "~5.8.2",
    "vite": "^6.2.0"
  }
}
```

#### 백엔드 (`server/package.json`)
```json
{
  "name": "dacon-raid-server",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "prisma:generate": "prisma generate",
    "prisma:migrate": "prisma migrate dev"
  },
  "dependencies": {
    "@prisma/client": "^5.22.0",
    "express": "^4.21.2",
    "cors": "^2.8.5",
    "express-rate-limit": "^7.5.0",
    "dotenv": "^16.4.7"
  },
  "devDependencies": {
    "@types/express": "^4.17.21",
    "@types/cors": "^2.8.17",
    "@types/node": "^22.14.0",
    "prisma": "^5.22.0",
    "tsx": "^4.19.2",
    "typescript": "~5.8.2"
  }
}
```

### 2.4 Replit 설정 파일

#### `.replit` 파일
```toml
run = "npm run dev"
entrypoint = "client/src/main.tsx"

[nix]
channel = "stable-24_05"

[deployment]
run = ["npm", "run", "start"]
deploymentTarget = "cloudrun"

[[ports]]
localPort = 5173
externalPort = 80
```

#### `replit.nix` 파일
```nix
{ pkgs }: {
  deps = [
    pkgs.nodejs-18_x
    pkgs.nodePackages.typescript
  ];
}
```

---

## 3. 프론트엔드 아키텍처

### 3.1 Tailwind CSS 설정

#### `tailwind.config.js`
```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#f0f9ff',
          100: '#e0f2fe',
          500: '#0ea5e9',
          600: '#0284c7',
          700: '#0369a1',
        },
        neutral: {
          900: '#171717',
          800: '#262626',
          700: '#404040',
        }
      }
    },
  },
  plugins: [],
}
```

#### `postcss.config.js`
```javascript
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

#### `client/src/index.css`
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  body {
    @apply bg-neutral-900 text-white;
  }
}

@layer components {
  .btn-primary {
    @apply bg-primary-500 hover:bg-primary-600 text-white font-semibold py-2 px-4 rounded-lg transition-colors;
  }
  
  .btn-secondary {
    @apply bg-neutral-700 hover:bg-neutral-600 text-white font-semibold py-2 px-4 rounded-lg transition-colors;
  }
  
  .card {
    @apply bg-neutral-800 rounded-xl p-6 shadow-lg;
  }
}
```

### 3.2 TypeScript 타입 정의

#### `client/src/types.ts`
```typescript
export interface User {
  id: string;
  displayName: string;
  photoURL: string;
  email?: string;
  roles: string[];
  specializations: string[];
  skills: string[];
  bio: string;
  portfolioLinks: {
    github?: string;
    kaggle?: string;
    blog?: string;
  };
  trustScore: {
    commitment: number;
    contribution: number;
    communication: number;
    collaboration: number;
    totalReviews: number;
  };
  createdAt: string;
  updatedAt: string;
}

export interface RaidSlot {
  slotId: string;
  role: string;
  level: '초급' | '중급' | '고급';
  count: number;
  filled: number;
}

export interface RaidMember {
  userId: string;
  role: string;
  slotId: string;
  joinedAt: string;
}

export interface Raid {
  id: string;
  createdBy: string;
  creator?: User; // 확장 정보
  status: 'recruiting' | 'full' | 'active' | 'completed';
  competitionId: string;
  competitionName: string;
  name: string;
  description: string;
  goal: '학습 중심' | '상위권 목표' | '프로덕트 완성';
  expectedHoursPerWeek: '5시간 미만' | '5-10시간' | '10시간 이상';
  collaborationMethod: string; // "디스코드", "슬랙", "줌" 등
  slots: RaidSlot[];
  members: RaidMember[];
  createdAt: string;
  updatedAt: string;
}

export interface Application {
  id: string;
  raidId: string;
  applicantId: string;
  applicant?: User; // 확장 정보
  appliedRole: string;
  appliedSlotId: string;
  appliedAt: string;
  status: 'pending' | 'accepted' | 'rejected';
  message: string;
  fitScore: number;
}

export interface Review {
  id: string;
  raidId: string;
  reviewerId: string;
  revieweeId: string;
  reviewer?: User; // 확장 정보
  reviewee?: User; // 확장 정보
  createdAt: string;
  commitment: number;
  contribution: number;
  communication: number;
  collaboration: number;
  feedback?: string;
}

export interface DaconCompetition {
  id: string;
  name: string;
  type: 'Tabular' | 'CV' | 'NLP' | 'RL';
  startDate: string;
  endDate: string;
  isActive: boolean;
}
```

### 3.3 주요 컴포넌트 구조

#### `client/src/App.tsx`
```typescript
import React from 'react';
import { HashRouter, Routes, Route } from 'react-router-dom';
import { DataProvider } from './contexts/DataContext';
import { AuthProvider } from './contexts/AuthContext';
import Header from './components/Header';
import HomePage from './pages/HomePage';
import RaidDetailPage from './pages/RaidDetailPage';
import ProfilePage from './pages/ProfilePage';
import CreateRaidPage from './pages/CreateRaidPage';
import DashboardPage from './pages/DashboardPage';

const App: React.FC = () => {
  return (
    <AuthProvider>
      <DataProvider>
        <HashRouter>
          <div className="min-h-screen bg-neutral-900">
            <Header />
            <main className="container mx-auto px-4 py-8">
              <Routes>
                <Route path="/" element={<HomePage />} />
                <Route path="/raid/:raidId" element={<RaidDetailPage />} />
                <Route path="/profile/:userId" element={<ProfilePage />} />
                <Route path="/create-raid" element={<CreateRaidPage />} />
                <Route path="/dashboard" element={<DashboardPage />} />
              </Routes>
            </main>
          </div>
        </HashRouter>
      </DataProvider>
    </AuthProvider>
  );
};

export default App;
```

---

## 4. 백엔드 아키텍처

### 4.1 Express.js 서버 설정

#### `server/src/index.ts`
```typescript
import express, { Express, Request, Response } from 'express';
import cors from 'cors';
import dotenv from 'dotenv';
import rateLimit from 'express-rate-limit';
import { PrismaClient } from '@prisma/client';

// 환경 변수 로드
dotenv.config();

// Prisma 클라이언트 초기화
export const prisma = new PrismaClient();

// Express 앱 생성
const app: Express = express();
const PORT = process.env.PORT || 3000;

// 미들웨어 설정
app.use(cors({
  origin: process.env.CLIENT_URL || 'http://localhost:5173',
  credentials: true,
}));
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Rate Limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15분
  max: 100, // 최대 100 요청
  message: 'Too many requests from this IP, please try again later.',
});
app.use('/api/', limiter);

// 라우트 등록
import authRoutes from './routes/auth.routes';
import userRoutes from './routes/user.routes';
import raidRoutes from './routes/raid.routes';
import applicationRoutes from './routes/application.routes';
import reviewRoutes from './routes/review.routes';
import competitionRoutes from './routes/competition.routes';

app.use('/api/auth', authRoutes);
app.use('/api/users', userRoutes);
app.use('/api/raids', raidRoutes);
app.use('/api/applications', applicationRoutes);
app.use('/api/reviews', reviewRoutes);
app.use('/api/competitions', competitionRoutes);

// Health Check
app.get('/health', (req: Request, res: Response) => {
  res.json({ status: 'OK', timestamp: new Date().toISOString() });
});

// 404 핸들러
app.use((req: Request, res: Response) => {
  res.status(404).json({ error: 'Endpoint not found' });
});

// 에러 핸들러
app.use((err: Error, req: Request, res: Response) => {
  console.error(err.stack);
  res.status(500).json({ 
    error: 'Internal server error',
    message: process.env.NODE_ENV === 'development' ? err.message : undefined
  });
});

// 서버 시작
app.listen(PORT, () => {
  console.log(`🚀 Server is running on port ${PORT}`);
  console.log(`📝 Health check: http://localhost:${PORT}/health`);
});

// Graceful Shutdown
process.on('SIGTERM', async () => {
  console.log('SIGTERM signal received: closing HTTP server');
  await prisma.$disconnect();
  process.exit(0);
});
```

### 4.2 Replit Auth 통합

#### `server/src/middleware/auth.middleware.ts`
```typescript
import { Request, Response, NextFunction } from 'express';

export interface AuthRequest extends Request {
  userId?: string;
  user?: {
    id: string;
    email: string;
    name: string;
  };
}

// Replit Auth 미들웨어
export const authenticateUser = async (
  req: AuthRequest,
  res: Response,
  next: NextFunction
) => {
  try {
    // Replit Auth 헤더 확인
    const replitUserId = req.headers['x-replit-user-id'] as string;
    const replitUserName = req.headers['x-replit-user-name'] as string;
    const replitUserEmail = req.headers['x-replit-user-email'] as string;

    if (!replitUserId) {
      return res.status(401).json({ error: 'Authentication required' });
    }

    // 사용자 정보 설정
    req.userId = replitUserId;
    req.user = {
      id: replitUserId,
      email: replitUserEmail,
      name: replitUserName,
    };

    next();
  } catch (error) {
    console.error('Authentication error:', error);
    res.status(401).json({ error: 'Invalid authentication' });
  }
};

// 선택적 인증 (로그인 안해도 접근 가능)
export const optionalAuth = async (
  req: AuthRequest,
  res: Response,
  next: NextFunction
) => {
  const replitUserId = req.headers['x-replit-user-id'] as string;
  
  if (replitUserId) {
    req.userId = replitUserId;
    req.user = {
      id: replitUserId,
      email: req.headers['x-replit-user-email'] as string,
      name: req.headers['x-replit-user-name'] as string,
    };
  }
  
  next();
};
```

---

## 5. 데이터베이스 설계

### 5.1 Neon PostgreSQL 설정

#### Replit에서 Neon 연결하기

1. **Neon 계정 생성**
   - https://neon.tech 접속
   - "Sign up" 클릭
   - GitHub 또는 Google로 가입

2. **프로젝트 생성**
   - "New Project" 클릭
   - 프로젝트명: `dacon-raid`
   - 리전 선택: 가장 가까운 리전 (예: `us-east-1`)
   - "Create Project" 클릭

3. **연결 문자열 복사**
   ```
   postgresql://user:password@ep-xxx.us-east-1.aws.neon.tech/neondb?sslmode=require
   ```

4. **Replit Secrets에 추가**
   - Replit의 Secrets 탭 (🔒 아이콘) 클릭
   - Key: `DATABASE_URL`
   - Value: 위 연결 문자열 붙여넣기

### 5.2 Prisma 스키마

#### `server/prisma/schema.prisma`
```prisma
// Prisma 설정
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// 사용자 모델
model User {
  id                String   @id @default(uuid())
  replitUserId      String   @unique // Replit Auth ID
  email             String   @unique
  displayName       String
  photoURL          String?
  bio               String   @default("")
  
  // 역할 및 전문성
  roles             String[] // ["데이터 분석가", "ML 엔지니어"]
  specializations   String[] // ["CV", "NLP", "시계열"]
  skills            String[] // ["Python", "PyTorch", "SQL"]
  
  // 포트폴리오
  githubUrl         String?
  kaggleUrl         String?
  blogUrl           String?
  
  // 신뢰 점수
  trustCommitment      Float    @default(0)
  trustContribution    Float    @default(0)
  trustCommunication   Float    @default(0)
  trustCollaboration   Float    @default(0)
  totalReviews         Int      @default(0)
  
  // 타임스탬프
  createdAt         DateTime @default(now())
  updatedAt         DateTime @updatedAt
  
  // 관계
  createdRaids      Raid[]        @relation("RaidCreator")
  raidMemberships   RaidMember[]
  applications      Application[]
  givenReviews      Review[]      @relation("ReviewGiver")
  receivedReviews   Review[]      @relation("ReviewReceiver")
  
  @@index([replitUserId])
  @@index([email])
}

// 대회 모델
model Competition {
  id          String   @id @default(uuid())
  daconId     String   @unique // 데이콘 API ID
  name        String
  type        String   // "Tabular", "CV", "NLP", "RL"
  startDate   DateTime
  endDate     DateTime
  isActive    Boolean  @default(true)
  
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  // 관계
  raids       Raid[]
  
  @@index([daconId])
  @@index([isActive])
}

// 원정대 모델
model Raid {
  id                    String   @id @default(uuid())
  name                  String
  description           String   @db.Text
  status                String   @default("recruiting") // "recruiting", "full", "active", "completed"
  
  // 목표 및 기대사항
  goal                  String   // "학습 중심", "상위권 목표", "프로덕트 완성"
  expectedHoursPerWeek  String   // "5시간 미만", "5-10시간", "10시간 이상"
  collaborationMethod   String   // "디스코드", "슬랙", "줌", "오프라인"
  
  // 외래 키
  createdBy             String
  creator               User         @relation("RaidCreator", fields: [createdBy], references: [id])
  competitionId         String
  competition           Competition  @relation(fields: [competitionId], references: [id])
  
  // 타임스탬프
  createdAt             DateTime @default(now())
  updatedAt             DateTime @updatedAt
  
  // 관계
  slots                 RaidSlot[]
  members               RaidMember[]
  applications          Application[]
  reviews               Review[]
  
  @@index([createdBy])
  @@index([competitionId])
  @@index([status])
}

// 역할 슬롯 모델
model RaidSlot {
  id        String @id @default(uuid())
  role      String // "데이터 분석가", "ML 엔지니어"
  level     String // "초급", "중급", "고급"
  count     Int    // 필요 인원 수
  filled    Int    @default(0) // 채워진 인원 수
  
  // 외래 키
  raidId    String
  raid      Raid   @relation(fields: [raidId], references: [id], onDelete: Cascade)
  
  // 관계
  members   RaidMember[]
  
  @@index([raidId])
}

// 팀원 모델
model RaidMember {
  id        String   @id @default(uuid())
  role      String   // 할당된 역할
  
  // 외래 키
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  raidId    String
  raid      Raid     @relation(fields: [raidId], references: [id], onDelete: Cascade)
  slotId    String
  slot      RaidSlot @relation(fields: [slotId], references: [id])
  
  // 타임스탬프
  joinedAt  DateTime @default(now())
  
  @@unique([userId, raidId]) // 한 사용자는 한 원정대에 한 번만 참여
  @@index([userId])
  @@index([raidId])
}

// 지원 모델
model Application {
  id              String   @id @default(uuid())
  appliedRole     String   // 지원한 역할
  message         String   @db.Text // 지원 메시지
  status          String   @default("pending") // "pending", "accepted", "rejected"
  fitScore        Float    // 적합도 점수 (0-100)
  
  // 외래 키
  applicantId     String
  applicant       User     @relation(fields: [applicantId], references: [id])
  raidId          String
  raid            Raid     @relation(fields: [raidId], references: [id], onDelete: Cascade)
  appliedSlotId   String   // 지원한 슬롯 ID
  
  // 타임스탬프
  appliedAt       DateTime @default(now())
  respondedAt     DateTime? // 수락/거절 시간
  
  @@unique([applicantId, raidId]) // 한 사용자는 한 원정대에 한 번만 지원
  @@index([applicantId])
  @@index([raidId])
  @@index([status])
}

// 평가 모델
model Review {
  id              String   @id @default(uuid())
  
  // 평가 항목 (1-5점)
  commitment      Int      // 커밋먼트
  contribution    Int      // 기여도
  communication   Int      // 소통
  collaboration   Int      // 협업성
  feedback        String?  @db.Text // 자유 피드백
  
  // 외래 키
  reviewerId      String
  reviewer        User     @relation("ReviewGiver", fields: [reviewerId], references: [id])
  revieweeId      String
  reviewee        User     @relation("ReviewReceiver", fields: [revieweeId], references: [id])
  raidId          String
  raid            Raid     @relation(fields: [raidId], references: [id], onDelete: Cascade)
  
  // 타임스탬프
  createdAt       DateTime @default(now())
  
  @@unique([reviewerId, revieweeId, raidId]) // 한 원정대에서 한 번만 평가
  @@index([revieweeId])
  @@index([raidId])
}
```

### 5.3 Prisma 마이그레이션

#### 초기 마이그레이션 실행
```bash
# Prisma Client 생성
cd server
npx prisma generate

# 마이그레이션 생성 및 실행
npx prisma migrate dev --name init

# 데이터베이스 스키마 확인
npx prisma studio
```

#### `server/.env` 파일 (Replit Secrets 사용 권장)
```env
DATABASE_URL="postgresql://user:password@ep-xxx.us-east-1.aws.neon.tech/neondb?sslmode=require"
NODE_ENV="development"
PORT=3000
CLIENT_URL="http://localhost:5173"
```

---

## 6. API 엔드포인트 설계

### 6.1 인증 관련 API

#### `server/src/routes/auth.routes.ts`
```typescript
import { Router } from 'express';
import { authenticateUser, AuthRequest } from '../middleware/auth.middleware';
import { prisma } from '../index';

const router = Router();

// 현재 사용자 정보 조회
router.get('/me', authenticateUser, async (req: AuthRequest, res) => {
  try {
    const user = await prisma.user.findUnique({
      where: { replitUserId: req.userId! }
    });

    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }

    res.json(user);
  } catch (error) {
    console.error('Get user error:', error);
    res.status(500).json({ error: 'Failed to fetch user' });
  }
});

// 사용자 등록 또는 업데이트
router.post('/register', authenticateUser, async (req: AuthRequest, res) => {
  try {
    const { displayName, photoURL, roles, specializations, skills, bio } = req.body;

    const user = await prisma.user.upsert({
      where: { replitUserId: req.userId! },
      update: {
        displayName,
        photoURL,
        roles,
        specializations,
        skills,
        bio,
      },
      create: {
        replitUserId: req.userId!,
        email: req.user!.email,
        displayName: displayName || req.user!.name,
        photoURL,
        roles: roles || [],
        specializations: specializations || [],
        skills: skills || [],
        bio: bio || '',
      },
    });

    res.json(user);
  } catch (error) {
    console.error('Register error:', error);
    res.status(500).json({ error: 'Failed to register user' });
  }
});

export default router;
```

### 6.2 원정대 관련 API

#### `server/src/routes/raid.routes.ts`
```typescript
import { Router } from 'express';
import { authenticateUser, optionalAuth, AuthRequest } from '../middleware/auth.middleware';
import { prisma } from '../index';

const router = Router();

// 원정대 목록 조회 (필터링 및 정렬)
router.get('/', optionalAuth, async (req: AuthRequest, res) => {
  try {
    const { 
      status, 
      competitionId, 
      role, 
      goal, 
      sortBy = 'createdAt',
      order = 'desc',
      page = '1',
      limit = '20'
    } = req.query;

    const where: any = {};
    
    if (status) where.status = status;
    if (competitionId) where.competitionId = competitionId;
    if (goal) where.goal = goal;

    const skip = (parseInt(page as string) - 1) * parseInt(limit as string);
    const take = parseInt(limit as string);

    const [raids, total] = await Promise.all([
      prisma.raid.findMany({
        where,
        include: {
          creator: true,
          competition: true,
          slots: true,
          members: {
            include: {
              user: true
            }
          },
          _count: {
            select: {
              applications: {
                where: { status: 'pending' }
              }
            }
          }
        },
        orderBy: { [sortBy as string]: order },
        skip,
        take,
      }),
      prisma.raid.count({ where })
    ]);

    res.json({
      raids,
      pagination: {
        page: parseInt(page as string),
        limit: parseInt(limit as string),
        total,
        totalPages: Math.ceil(total / parseInt(limit as string))
      }
    });
  } catch (error) {
    console.error('Get raids error:', error);
    res.status(500).json({ error: 'Failed to fetch raids' });
  }
});

// 특정 원정대 상세 조회
router.get('/:id', optionalAuth, async (req: AuthRequest, res) => {
  try {
    const { id } = req.params;

    const raid = await prisma.raid.findUnique({
      where: { id },
      include: {
        creator: true,
        competition: true,
        slots: true,
        members: {
          include: {
            user: true,
            slot: true
          }
        },
        applications: {
          where: { status: 'pending' },
          include: {
            applicant: true
          }
        }
      }
    });

    if (!raid) {
      return res.status(404).json({ error: 'Raid not found' });
    }

    res.json(raid);
  } catch (error) {
    console.error('Get raid error:', error);
    res.status(500).json({ error: 'Failed to fetch raid' });
  }
});

// 원정대 생성
router.post('/', authenticateUser, async (req: AuthRequest, res) => {
  try {
    const {
      name,
      description,
      competitionId,
      goal,
      expectedHoursPerWeek,
      collaborationMethod,
      slots
    } = req.body;

    // 입력 검증
    if (!name || !description || !competitionId || !slots || slots.length === 0) {
      return res.status(400).json({ error: 'Missing required fields' });
    }

    const raid = await prisma.raid.create({
      data: {
        name,
        description,
        competitionId,
        goal,
        expectedHoursPerWeek,
        collaborationMethod,
        createdBy: req.userId!,
        slots: {
          create: slots.map((slot: any) => ({
            role: slot.role,
            level: slot.level,
            count: slot.count,
            filled: 0
          }))
        }
      },
      include: {
        creator: true,
        competition: true,
        slots: true
      }
    });

    res.status(201).json(raid);
  } catch (error) {
    console.error('Create raid error:', error);
    res.status(500).json({ error: 'Failed to create raid' });
  }
});

// 원정대 수정
router.patch('/:id', authenticateUser, async (req: AuthRequest, res) => {
  try {
    const { id } = req.params;
    const updateData = req.body;

    // 원정대 소유자 확인
    const raid = await prisma.raid.findUnique({
      where: { id }
    });

    if (!raid) {
      return res.status(404).json({ error: 'Raid not found' });
    }

    if (raid.createdBy !== req.userId) {
      return res.status(403).json({ error: 'Not authorized to update this raid' });
    }

    const updatedRaid = await prisma.raid.update({
      where: { id },
      data: updateData,
      include: {
        creator: true,
        competition: true,
        slots: true,
        members: {
          include: {
            user: true
          }
        }
      }
    });

    res.json(updatedRaid);
  } catch (error) {
    console.error('Update raid error:', error);
    res.status(500).json({ error: 'Failed to update raid' });
  }
});

// 원정대 삭제
router.delete('/:id', authenticateUser, async (req: AuthRequest, res) => {
  try {
    const { id } = req.params;

    // 원정대 소유자 확인
    const raid = await prisma.raid.findUnique({
      where: { id }
    });

    if (!raid) {
      return res.status(404).json({ error: 'Raid not found' });
    }

    if (raid.createdBy !== req.userId) {
      return res.status(403).json({ error: 'Not authorized to delete this raid' });
    }

    await prisma.raid.delete({
      where: { id }
    });

    res.json({ message: 'Raid deleted successfully' });
  } catch (error) {
    console.error('Delete raid error:', error);
    res.status(500).json({ error: 'Failed to delete raid' });
  }
});

export default router;
```

### 6.3 지원 관련 API

#### `server/src/routes/application.routes.ts`
```typescript
import { Router } from 'express';
import { authenticateUser, AuthRequest } from '../middleware/auth.middleware';
import { prisma } from '../index';
import { calculateFitScore } from '../utils/fitScore';

const router = Router();

// 원정대에 지원하기
router.post('/', authenticateUser, async (req: AuthRequest, res) => {
  try {
    const { raidId, appliedRole, appliedSlotId, message } = req.body;

    // 입력 검증
    if (!raidId || !appliedRole || !appliedSlotId || !message) {
      return res.status(400).json({ error: 'Missing required fields' });
    }

    // 이미 지원했는지 확인
    const existingApplication = await prisma.application.findUnique({
      where: {
        applicantId_raidId: {
          applicantId: req.userId!,
          raidId
        }
      }
    });

    if (existingApplication) {
      return res.status(400).json({ error: 'Already applied to this raid' });
    }

    // 원정대 및 사용자 정보 조회
    const [raid, user] = await Promise.all([
      prisma.raid.findUnique({
        where: { id: raidId },
        include: {
          slots: true,
          creator: true
        }
      }),
      prisma.user.findUnique({
        where: { id: req.userId! }
      })
    ]);

    if (!raid) {
      return res.status(404).json({ error: 'Raid not found' });
    }

    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }

    // 적합도 점수 계산
    const fitScore = calculateFitScore(user, raid, appliedRole);

    // 지원 생성
    const application = await prisma.application.create({
      data: {
        applicantId: req.userId!,
        raidId,
        appliedRole,
        appliedSlotId,
        message,
        fitScore
      },
      include: {
        applicant: true,
        raid: {
          include: {
            creator: true,
            competition: true
          }
        }
      }
    });

    res.status(201).json(application);
  } catch (error) {
    console.error('Create application error:', error);
    res.status(500).json({ error: 'Failed to create application' });
  }
});

// 특정 원정대의 지원자 목록 조회 (리더만)
router.get('/raid/:raidId', authenticateUser, async (req: AuthRequest, res) => {
  try {
    const { raidId } = req.params;

    // 원정대 소유자 확인
    const raid = await prisma.raid.findUnique({
      where: { id: raidId }
    });

    if (!raid) {
      return res.status(404).json({ error: 'Raid not found' });
    }

    if (raid.createdBy !== req.userId) {
      return res.status(403).json({ error: 'Not authorized to view applications' });
    }

    const applications = await prisma.application.findMany({
      where: { raidId },
      include: {
        applicant: true
      },
      orderBy: { fitScore: 'desc' }
    });

    res.json(applications);
  } catch (error) {
    console.error('Get applications error:', error);
    res.status(500).json({ error: 'Failed to fetch applications' });
  }
});

// 지원 수락/거절
router.patch('/:id', authenticateUser, async (req: AuthRequest, res) => {
  try {
    const { id } = req.params;
    const { status } = req.body; // "accepted" or "rejected"

    if (!['accepted', 'rejected'].includes(status)) {
      return res.status(400).json({ error: 'Invalid status' });
    }

    const application = await prisma.application.findUnique({
      where: { id },
      include: {
        raid: {
          include: {
            slots: true
          }
        }
      }
    });

    if (!application) {
      return res.status(404).json({ error: 'Application not found' });
    }

    // 원정대 소유자 확인
    if (application.raid.createdBy !== req.userId) {
      return res.status(403).json({ error: 'Not authorized to respond to this application' });
    }

    // 수락 시 처리
    if (status === 'accepted') {
      // 슬롯이 가득 찼는지 확인
      const slot = application.raid.slots.find(s => s.id === application.appliedSlotId);
      
      if (!slot) {
        return res.status(400).json({ error: 'Slot not found' });
      }

      if (slot.filled >= slot.count) {
        return res.status(400).json({ error: 'Slot is already full' });
      }

      // 트랜잭션으로 처리
      await prisma.$transaction([
        // 지원 상태 업데이트
        prisma.application.update({
          where: { id },
          data: {
            status,
            respondedAt: new Date()
          }
        }),
        // 팀원 추가
        prisma.raidMember.create({
          data: {
            userId: application.applicantId,
            raidId: application.raidId,
            slotId: application.appliedSlotId,
            role: application.appliedRole
          }
        }),
        // 슬롯 채워진 수 증가
        prisma.raidSlot.update({
          where: { id: application.appliedSlotId },
          data: {
            filled: {
              increment: 1
            }
          }
        })
      ]);

      // 모든 슬롯이 찼는지 확인 후 원정대 상태 업데이트
      const updatedRaid = await prisma.raid.findUnique({
        where: { id: application.raidId },
        include: { slots: true }
      });

      if (updatedRaid) {
        const allSlotsFilled = updatedRaid.slots.every(s => s.filled >= s.count);
        if (allSlotsFilled) {
          await prisma.raid.update({
            where: { id: application.raidId },
            data: { status: 'full' }
          });
        }
      }
    } else {
      // 거절 시
      await prisma.application.update({
        where: { id },
        data: {
          status,
          respondedAt: new Date()
        }
      });
    }

    const updatedApplication = await prisma.application.findUnique({
      where: { id },
      include: {
        applicant: true,
        raid: {
          include: {
            creator: true,
            competition: true
          }
        }
      }
    });

    res.json(updatedApplication);
  } catch (error) {
    console.error('Update application error:', error);
    res.status(500).json({ error: 'Failed to update application' });
  }
});

// 내 지원 현황 조회
router.get('/my-applications', authenticateUser, async (req: AuthRequest, res) => {
  try {
    const applications = await prisma.application.findMany({
      where: { applicantId: req.userId! },
      include: {
        raid: {
          include: {
            creator: true,
            competition: true
          }
        }
      },
      orderBy: { appliedAt: 'desc' }
    });

    res.json(applications);
  } catch (error) {
    console.error('Get my applications error:', error);
    res.status(500).json({ error: 'Failed to fetch applications' });
  }
});

export default router;
```

### 6.4 적합도 점수 계산 유틸리티

#### `server/src/utils/fitScore.ts`
```typescript
import { User, Raid } from '@prisma/client';

/**
 * 적합도 점수 계산 (0-100점)
 * 
 * 계산 방식:
 * - 역량 매칭 (40%): 역할, 전문 분야, 스킬 일치도
 * - 신뢰 점수 (30%): 과거 평가 점수
 * - 활동 이력 (30%): 총 평가 수, 프로필 완성도
 */
export function calculateFitScore(
  user: User,
  raid: Raid,
  appliedRole: string
): number {
  let score = 0;

  // 1. 역량 매칭 (40점)
  let roleMatchScore = 0;
  
  // 역할 일치 여부 (20점)
  if (user.roles.includes(appliedRole)) {
    roleMatchScore += 20;
  }
  
  // 스킬 일치도 (20점)
  // 원정대가 요구하는 스킬이 있다면 비교 (여기서는 간단히 스킬 개수로 평가)
  const skillScore = Math.min(user.skills.length * 2, 20);
  roleMatchScore += skillScore;

  score += roleMatchScore;

  // 2. 신뢰 점수 (30점)
  const avgTrust = (
    user.trustCommitment +
    user.trustContribution +
    user.trustCommunication +
    user.trustCollaboration
  ) / 4;
  
  // 신뢰 점수를 30점 만점으로 환산
  const trustScore = (avgTrust / 5) * 30;
  score += trustScore;

  // 3. 활동 이력 (30점)
  let activityScore = 0;
  
  // 평가 받은 횟수 (15점)
  const reviewScore = Math.min(user.totalReviews * 3, 15);
  activityScore += reviewScore;
  
  // 프로필 완성도 (15점)
  let profileCompletion = 0;
  if (user.bio) profileCompletion += 3;
  if (user.githubUrl) profileCompletion += 3;
  if (user.kaggleUrl) profileCompletion += 3;
  if (user.blogUrl) profileCompletion += 3;
  if (user.roles.length > 0) profileCompletion += 3;
  activityScore += profileCompletion;

  score += activityScore;

  // 최종 점수를 0-100 사이로 정규화
  return Math.min(Math.max(Math.round(score), 0), 100);
}
```

---

## 7. 주요 기능 구현 가이드

### 7.1 사용자 프로필 페이지

#### `client/src/pages/ProfilePage.tsx`
```typescript
import React, { useEffect, useState } from 'react';
import { useParams } from 'react-router-dom';
import { User } from '../types';
import { StarIcon, GithubIcon, ExternalLinkIcon } from 'lucide-react';

const ProfilePage: React.FC = () => {
  const { userId } = useParams<{ userId: string }>();
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUser();
  }, [userId]);

  const fetchUser = async () => {
    try {
      const response = await fetch(`/api/users/${userId}`);
      if (response.ok) {
        const data = await response.json();
        setUser(data);
      }
    } catch (error) {
      console.error('Failed to fetch user:', error);
    } finally {
      setLoading(false);
    }
  };

  if (loading) {
    return <div className="text-center py-20">로딩 중...</div>;
  }

  if (!user) {
    return <div className="text-center py-20">사용자를 찾을 수 없습니다.</div>;
  }

  const avgTrustScore = (
    user.trustScore.commitment +
    user.trustScore.contribution +
    user.trustScore.communication +
    user.trustScore.collaboration
  ) / 4;

  return (
    <div className="max-w-4xl mx-auto">
      {/* 프로필 헤더 */}
      <div className="card mb-6">
        <div className="flex items-start gap-6">
          {/* 프로필 사진 */}
          <img
            src={user.photoURL || 'https://via.placeholder.com/128'}
            alt={user.displayName}
            className="w-32 h-32 rounded-full object-cover"
          />
          
          {/* 기본 정보 */}
          <div className="flex-1">
            <h1 className="text-3xl font-bold mb-2">{user.displayName}</h1>
            
            {/* 역할 뱃지 */}
            <div className="flex flex-wrap gap-2 mb-4">
              {user.roles.map((role) => (
                <span key={role} className="px-3 py-1 bg-primary-500 text-white rounded-full text-sm">
                  {role}
                </span>
              ))}
            </div>

            {/* 신뢰 점수 */}
            <div className="flex items-center gap-2 mb-4">
              <StarIcon className="w-5 h-5 text-yellow-400 fill-yellow-400" />
              <span className="text-lg font-semibold">{avgTrustScore.toFixed(1)}</span>
              <span className="text-neutral-400">({user.trustScore.totalReviews}개 평가)</span>
            </div>

            {/* 포트폴리오 링크 */}
            <div className="flex gap-3">
              {user.portfolioLinks.github && (
                <a
                  href={user.portfolioLinks.github}
                  target="_blank"
                  rel="noopener noreferrer"
                  className="flex items-center gap-1 text-primary-400 hover:text-primary-300"
                >
                  <GithubIcon className="w-5 h-5" />
                  GitHub
                </a>
              )}
              {user.portfolioLinks.kaggle && (
                <a
                  href={user.portfolioLinks.kaggle}
                  target="_blank"
                  rel="noopener noreferrer"
                  className="flex items-center gap-1 text-primary-400 hover:text-primary-300"
                >
                  <ExternalLinkIcon className="w-5 h-5" />
                  Kaggle
                </a>
              )}
              {user.portfolioLinks.blog && (
                <a
                  href={user.portfolioLinks.blog}
                  target="_blank"
                  rel="noopener noreferrer"
                  className="flex items-center gap-1 text-primary-400 hover:text-primary-300"
                >
                  <ExternalLinkIcon className="w-5 h-5" />
                  Blog
                </a>
              )}
            </div>
          </div>
        </div>

        {/* 자기소개 */}
        {user.bio && (
          <div className="mt-6 pt-6 border-t border-neutral-700">
            <h3 className="text-lg font-semibold mb-2">자기소개</h3>
            <p className="text-neutral-300 whitespace-pre-wrap">{user.bio}</p>
          </div>
        )}
      </div>

      {/* 전문 분야 */}
      {user.specializations.length > 0 && (
        <div className="card mb-6">
          <h3 className="text-lg font-semibold mb-3">전문 분야</h3>
          <div className="flex flex-wrap gap-2">
            {user.specializations.map((spec) => (
              <span key={spec} className="px-3 py-1 bg-neutral-700 rounded-full text-sm">
                {spec}
              </span>
            ))}
          </div>
        </div>
      )}

      {/* 기술 스택 */}
      {user.skills.length > 0 && (
        <div className="card mb-6">
          <h3 className="text-lg font-semibold mb-3">기술 스택</h3>
          <div className="flex flex-wrap gap-2">
            {user.skills.map((skill) => (
              <span key={skill} className="px-3 py-1 bg-neutral-700 rounded-full text-sm">
                {skill}
              </span>
            ))}
          </div>
        </div>
      )}

      {/* 상세 신뢰 점수 */}
      <div className="card">
        <h3 className="text-lg font-semibold mb-4">신뢰 점수 상세</h3>
        <div className="space-y-4">
          {[
            { label: '커밋먼트', value: user.trustScore.commitment },
            { label: '기여도', value: user.trustScore.contribution },
            { label: '소통', value: user.trustScore.communication },
            { label: '협업성', value: user.trustScore.collaboration },
          ].map((item) => (
            <div key={item.label}>
              <div className="flex justify-between mb-1">
                <span className="text-sm text-neutral-400">{item.label}</span>
                <span className="text-sm font-semibold">{item.value.toFixed(1)} / 5.0</span>
              </div>
              <div className="w-full bg-neutral-700 rounded-full h-2">
                <div
                  className="bg-primary-500 h-2 rounded-full"
                  style={{ width: `${(item.value / 5) * 100}%` }}
                />
              </div>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
};

export default ProfilePage;
```

### 7.2 원정대 생성 페이지

#### `client/src/pages/CreateRaidPage.tsx`
```typescript
import React, { useState, useEffect } from 'react';
import { useNavigate } from 'react-router-dom';
import { PlusIcon, TrashIcon } from 'lucide-react';
import { DaconCompetition, RaidSlot } from '../types';

const CreateRaidPage: React.FC = () => {
  const navigate = useNavigate();
  const [competitions, setCompetitions] = useState<DaconCompetition[]>([]);
  const [loading, setLoading] = useState(false);
  
  const [formData, setFormData] = useState({
    name: '',
    description: '',
    competitionId: '',
    goal: '학습 중심',
    expectedHoursPerWeek: '5-10시간',
    collaborationMethod: '디스코드',
  });

  const [slots, setSlots] = useState<Omit<RaidSlot, 'slotId' | 'filled'>[]>([
    { role: '데이터 분석가', level: '중급', count: 1 }
  ]);

  useEffect(() => {
    fetchCompetitions();
  }, []);

  const fetchCompetitions = async () => {
    try {
      const response = await fetch('/api/competitions?isActive=true');
      if (response.ok) {
        const data = await response.json();
        setCompetitions(data);
      }
    } catch (error) {
      console.error('Failed to fetch competitions:', error);
    }
  };

  const handleAddSlot = () => {
    setSlots([...slots, { role: '데이터 분석가', level: '중급', count: 1 }]);
  };

  const handleRemoveSlot = (index: number) => {
    if (slots.length > 1) {
      setSlots(slots.filter((_, i) => i !== index));
    }
  };

  const handleSlotChange = (index: number, field: string, value: any) => {
    const newSlots = [...slots];
    newSlots[index] = { ...newSlots[index], [field]: value };
    setSlots(newSlots);
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);

    try {
      const response = await fetch('/api/raids', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          ...formData,
          slots,
        }),
      });

      if (response.ok) {
        const raid = await response.json();
        navigate(`/raid/${raid.id}`);
      } else {
        alert('원정대 생성에 실패했습니다.');
      }
    } catch (error) {
      console.error('Failed to create raid:', error);
      alert('원정대 생성 중 오류가 발생했습니다.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-3xl mx-auto">
      <h1 className="text-3xl font-bold mb-6">새 원정대 만들기</h1>

      <form onSubmit={handleSubmit} className="space-y-6">
        {/* 기본 정보 */}
        <div className="card">
          <h2 className="text-xl font-semibold mb-4">기본 정보</h2>
          
          <div className="space-y-4">
            {/* 원정대 이름 */}
            <div>
              <label className="block text-sm font-medium mb-2">
                원정대 이름 *
              </label>
              <input
                type="text"
                required
                value={formData.name}
                onChange={(e) => setFormData({ ...formData, name: e.target.value })}
                className="w-full px-4 py-2 bg-neutral-700 border border-neutral-600 rounded-lg focus:outline-none focus:border-primary-500"
                placeholder="예: 상위권 진입 원정대"
              />
            </div>

            {/* 대회 선택 */}
            <div>
              <label className="block text-sm font-medium mb-2">
                대회 선택 *
              </label>
              <select
                required
                value={formData.competitionId}
                onChange={(e) => setFormData({ ...formData, competitionId: e.target.value })}
                className="w-full px-4 py-2 bg-neutral-700 border border-neutral-600 rounded-lg focus:outline-none focus:border-primary-500"
              >
                <option value="">대회를 선택하세요</option>
                {competitions.map((comp) => (
                  <option key={comp.id} value={comp.id}>
                    {comp.name} ({comp.type})
                  </option>
                ))}
              </select>
            </div>

            {/* 원정대 소개 */}
            <div>
              <label className="block text-sm font-medium mb-2">
                원정대 소개 *
              </label>
              <textarea
                required
                rows={4}
                value={formData.description}
                onChange={(e) => setFormData({ ...formData, description: e.target.value })}
                className="w-full px-4 py-2 bg-neutral-700 border border-neutral-600 rounded-lg focus:outline-none focus:border-primary-500"
                placeholder="원정대의 목표와 분위기를 설명해주세요..."
              />
            </div>
          </div>
        </div>

        {/* 목표 및 기대사항 */}
        <div className="card">
          <h2 className="text-xl font-semibold mb-4">목표 및 기대사항</h2>
          
          <div className="space-y-4">
            {/* 원정대 목표 */}
            <div>
              <label className="block text-sm font-medium mb-2">
                원정대 목표 *
              </label>
              <select
                value={formData.goal}
                onChange={(e) => setFormData({ ...formData, goal: e.target.value })}
                className="w-full px-4 py-2 bg-neutral-700 border border-neutral-600 rounded-lg focus:outline-none focus:border-primary-500"
              >
                <option value="학습 중심">학습 중심</option>
                <option value="상위권 목표">상위권 목표</option>
                <option value="프로덕트 완성">프로덕트 완성</option>
              </select>
            </div>

            {/* 주당 예상 시간 */}
            <div>
              <label className="block text-sm font-medium mb-2">
                주당 예상 시간 *
              </label>
              <select
                value={formData.expectedHoursPerWeek}
                onChange={(e) => setFormData({ ...formData, expectedHoursPerWeek: e.target.value })}
                className="w-full px-4 py-2 bg-neutral-700 border border-neutral-600 rounded-lg focus:outline-none focus:border-primary-500"
              >
                <option value="5시간 미만">5시간 미만</option>
                <option value="5-10시간">5-10시간</option>
                <option value="10시간 이상">10시간 이상</option>
              </select>
            </div>

            {/* 협업 방식 */}
            <div>
              <label className="block text-sm font-medium mb-2">
                협업 방식 *
              </label>
              <input
                type="text"
                required
                value={formData.collaborationMethod}
                onChange={(e) => setFormData({ ...formData, collaborationMethod: e.target.value })}
                className="w-full px-4 py-2 bg-neutral-700 border border-neutral-600 rounded-lg focus:outline-none focus:border-primary-500"
                placeholder="예: 디스코드, 슬랙, 줌, 오프라인"
              />
            </div>
          </div>
        </div>

        {/* 역할 슬롯 */}
        <div className="card">
          <div className="flex justify-between items-center mb-4">
            <h2 className="text-xl font-semibold">필요한 팀원</h2>
            <button
              type="button"
              onClick={handleAddSlot}
              className="flex items-center gap-2 btn-secondary"
            >
              <PlusIcon className="w-4 h-4" />
              슬롯 추가
            </button>
          </div>

          <div className="space-y-4">
            {slots.map((slot, index) => (
              <div key={index} className="p-4 bg-neutral-700 rounded-lg">
                <div className="flex gap-3">
                  {/* 역할 */}
                  <div className="flex-1">
                    <label className="block text-sm font-medium mb-2">역할</label>
                    <select
                      value={slot.role}
                      onChange={(e) => handleSlotChange(index, 'role', e.target.value)}
                      className="w-full px-3 py-2 bg-neutral-800 border border-neutral-600 rounded-lg focus:outline-none focus:border-primary-500"
                    >
                      <option value="데이터 분석가">데이터 분석가</option>
                      <option value="ML 엔지니어">ML 엔지니어</option>
                      <option value="기획자">기획자</option>
                      <option value="디자이너">디자이너</option>
                      <option value="프론트엔드 개발자">프론트엔드 개발자</option>
                      <option value="백엔드 개발자">백엔드 개발자</option>
                    </select>
                  </div>

                  {/* 레벨 */}
                  <div className="flex-1">
                    <label className="block text-sm font-medium mb-2">레벨</label>
                    <select
                      value={slot.level}
                      onChange={(e) => handleSlotChange(index, 'level', e.target.value)}
                      className="w-full px-3 py-2 bg-neutral-800 border border-neutral-600 rounded-lg focus:outline-none focus:border-primary-500"
                    >
                      <option value="초급">초급</option>
                      <option value="중급">중급</option>
                      <option value="고급">고급</option>
                    </select>
                  </div>

                  {/* 인원 수 */}
                  <div className="w-24">
                    <label className="block text-sm font-medium mb-2">인원</label>
                    <input
                      type="number"
                      min="1"
                      max="10"
                      value={slot.count}
                      onChange={(e) => handleSlotChange(index, 'count', parseInt(e.target.value))}
                      className="w-full px-3 py-2 bg-neutral-800 border border-neutral-600 rounded-lg focus:outline-none focus:border-primary-500"
                    />
                  </div>

                  {/* 삭제 버튼 */}
                  {slots.length > 1 && (
                    <div className="flex items-end">
                      <button
                        type="button"
                        onClick={() => handleRemoveSlot(index)}
                        className="p-2 text-red-400 hover:bg-neutral-800 rounded-lg"
                      >
                        <TrashIcon className="w-5 h-5" />
                      </button>
                    </div>
                  )}
                </div>
              </div>
            ))}
          </div>
        </div>

        {/* 제출 버튼 */}
        <div className="flex justify-end gap-3">
          <button
            type="button"
            onClick={() => navigate('/')}
            className="btn-secondary"
          >
            취소
          </button>
          <button
            type="submit"
            disabled={loading}
            className="btn-primary"
          >
            {loading ? '생성 중...' : '원정대 만들기'}
          </button>
        </div>
      </form>
    </div>
  );
};

export default CreateRaidPage;
```

---

## 8. 개발 로드맵

### 8.1 Phase 1: MVP (Week 1-10)

#### Week 1-2: 프로젝트 설정 및 기반 구축
- [x] Replit 프로젝트 생성
- [x] 프론트엔드 환경 설정 (React + TypeScript + Tailwind)
- [x] 백엔드 환경 설정 (Express.js + Prisma)
- [x] Neon PostgreSQL 연결
- [x] Replit Auth 설정
- [x] 기본 라우팅 및 레이아웃

#### Week 3-4: 사용자 프로필 시스템
- [ ] 사용자 등록/로그인 기능
- [ ] 프로필 생성 및 수정 UI
- [ ] 프로필 조회 페이지
- [ ] 포트폴리오 링크 관리
- [ ] 역할 및 기술 스택 선택

#### Week 5-6: 원정대 기능
- [ ] 원정대 생성 폼
- [ ] 원정대 목록 페이지 (필터링/정렬)
- [ ] 원정대 상세 페이지
- [ ] 역할 슬롯 관리
- [ ] 팀원 현황 표시

#### Week 7-8: 지원 및 매칭 시스템
- [ ] 지원하기 기능
- [ ] 적합도 점수 계산 알고리즘
- [ ] 리더 대시보드 (지원자 관리)
- [ ] 수락/거절 기능
- [ ] 알림 시스템 (이메일)

#### Week 9-10: 평가 시스템 및 테스트
- [ ] 평가 양식 UI
- [ ] 평가 제출 및 저장
- [ ] 신뢰 점수 계산 및 표시
- [ ] 종단 간 테스트
- [ ] 버그 수정 및 최적화

### 8.2 Phase 2: 고도화 (Week 11-24)

#### Week 11-14: 협업 도구
- [ ] 팀 채팅 (WebSocket)
- [ ] 파일 공유 기능
- [ ] 작업 할당 및 트래킹 (칸반 보드)
- [ ] 코드 공유 통합

#### Week 15-18: 고급 매칭 알고리즘
- [ ] ML 기반 적합도 예측
- [ ] 협업 스타일 분석
- [ ] 팀 성공 예측 모델
- [ ] 추천 시스템

#### Week 19-22: 레퓨테이션 시스템 확장
- [ ] 역할별 전문성 레벨
- [ ] 뱃지 시스템
- [ ] 명예의 전당
- [ ] 멘토링 시스템

#### Week 23-24: 최적화 및 안정화
- [ ] 성능 최적화
- [ ] 보안 강화
- [ ] 사용자 경험 개선
- [ ] 문서화 완료

### 8.3 Phase 3: 생태계 확장 (Week 25+)

- [ ] 크로스 플랫폼 통합 (캐글, 다른 플랫폼)
- [ ] 커뮤니티 기능 (포럼, Q&A)
- [ ] 프로덕트 쇼케이스
- [ ] 모바일 앱 (React Native 또는 PWA)

---

## 9. 배포 및 운영

### 9.1 Replit Deployments

#### 배포 설정

1. **Replit Deployments 활성화**
   - Replit 프로젝트에서 "Deploy" 탭 클릭
   - "Create Deployment" 선택
   - 배포 유형: "Autoscale" 선택

2. **환경 변수 설정**
   - Secrets 탭에서 프로덕션 환경 변수 설정
   - `NODE_ENV=production`
   - `DATABASE_URL` (프로덕션 DB)

3. **빌드 스크립트 설정**
   - `.replit` 파일의 `deployment.run` 설정 확인
   ```toml
   [deployment]
   run = ["sh", "-c", "cd server && npm run build && npm start"]
   deploymentTarget = "cloudrun"
   ```

4. **도메인 설정**
   - Replit 제공 도메인: `your-project.repl.co`
   - 커스텀 도메인 연결 가능 (유료 플랜)

### 9.2 데이터베이스 백업

#### Neon 자동 백업
```bash
# Neon은 자동으로 7일간 백업 제공
# 백업 복원은 Neon 콘솔에서 수행
```

#### 수동 백업 스크립트
```bash
# server/scripts/backup.sh
#!/bin/bash

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_$DATE.sql"

# PostgreSQL 덤프
pg_dump $DATABASE_URL > $BACKUP_FILE

# 클라우드 스토리지에 업로드 (옵션)
# aws s3 cp $BACKUP_FILE s3://your-bucket/backups/
```

### 9.3 모니터링

#### 로그 관리
```typescript
// server/src/utils/logger.ts
import fs from 'fs/promises';

export async function logError(error: Error, context: string) {
  const logEntry = {
    timestamp: new Date().toISOString(),
    context,
    error: error.message,
    stack: error.stack,
  };

  // 콘솔 출력
  console.error(`[${context}]`, error);

  // 파일에 저장
  try {
    await fs.appendFile(
      'logs/errors.log',
      JSON.stringify(logEntry) + '\n'
    );
  } catch (err) {
    console.error('Failed to write log:', err);
  }
}
```

#### 성능 모니터링
```typescript
// server/src/middleware/monitoring.middleware.ts
import { Request, Response, NextFunction } from 'express';

export const monitorPerformance = (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;
    
    // 느린 요청 로깅 (500ms 이상)
    if (duration > 500) {
      console.warn(`Slow request: ${req.method} ${req.path} took ${duration}ms`);
    }
  });

  next();
};
```

### 9.4 CI/CD

#### GitHub Actions (선택 사항)
```yaml
# .github/workflows/deploy.yml
name: Deploy to Replit

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm install
      
      - name: Run tests
        run: npm test
      
      - name: Deploy to Replit
        env:
          REPLIT_TOKEN: ${{ secrets.REPLIT_TOKEN }}
        run: |
          # Replit API를 통한 배포 트리거
          curl -X POST https://replit.com/api/v1/deploys \
            -H "Authorization: Bearer $REPLIT_TOKEN" \
            -d '{"replId": "your-repl-id"}'
```

---

## 10. 문제 해결 가이드

### 10.1 Replit 관련 문제

#### Q: Repl이 느리게 작동합니다
**해결 방법:**
```bash
# 캐시 정리
npm cache clean --force

# node_modules 재설치
rm -rf node_modules
npm install

# Replit Workspace 재시작
```

#### Q: 배포 후 404 오류
**해결 방법:**
1. `.replit` 파일의 `run` 명령어 확인
2. `package.json`의 `start` 스크립트 확인
3. Express 라우팅 경로 확인

#### Q: Secrets가 로드되지 않음
**해결 방법:**
1. Replit Secrets 탭에서 키 이름 확인
2. 서버 재시작
3. `process.env.VARIABLE_NAME`으로 접근 확인

### 10.2 데이터베이스 관련 문제

#### Q: Prisma 마이그레이션 실패
**해결 방법:**
```bash
# 스키마 검증
npx prisma validate

# 강제 마이그레이션 재설정 (주의: 데이터 손실)
npx prisma migrate reset --force

# 새 마이그레이션 생성
npx prisma migrate dev --name fix_migration
```

#### Q: "P2002: Unique constraint failed" 오류
**해결 방법:**
- 중복 데이터 확인
- 스키마의 `@unique` 제약 조건 검토
- 데이터 정리 또는 제약 조건 수정

#### Q: 연결 풀 고갈
**해결 방법:**
```typescript
// Prisma 클라이언트 싱글톤 패턴 사용
import { PrismaClient } from '@prisma/client';

const globalForPrisma = global as unknown as { prisma: PrismaClient };

export const prisma =
  globalForPrisma.prisma ||
  new PrismaClient({
    log: ['query', 'error', 'warn'],
  });

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
```

### 10.3 Express.js 관련 문제

#### Q: CORS 오류 발생
**해결 방법:**
```typescript
import cors from 'cors';

app.use(cors({
  origin: process.env.CLIENT_URL || 'http://localhost:5173',
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
}));
```

#### Q: API 요청이 작동하지 않음
**해결 방법:**
1. 서버가 실행 중인지 확인 (Replit Console)
2. 포트 번호 확인 (프론트엔드와 백엔드)
3. 브라우저 개발자 도구의 Network 탭에서 요청 확인
4. Express 라우트 등록 순서 확인

#### Q: Rate Limiting으로 요청 차단
**해결 방법:**
```typescript
// 개발 환경에서는 Rate Limiting 비활성화
if (process.env.NODE_ENV !== 'development') {
  app.use('/api/', limiter);
}
```

### 10.4 일반 개발 이슈

#### Q: npm install 실패
**해결 방법:**
```bash
# package-lock.json 삭제 후 재설치
rm package-lock.json
rm -rf node_modules
npm install

# 또는 강제 설치
npm install --force
```

#### Q: TypeScript 컴파일 오류
**해결 방법:**
```bash
# TypeScript 캐시 정리
npx tsc --build --clean

# 타입 정의 재설치
npm install --save-dev @types/node @types/react @types/express
```

#### Q: Vite 빌드 오류
**해결 방법:**
1. `import` 경로 확인 (확장자 포함 여부)
2. 환경 변수 확인 (`import.meta.env`)
3. 빌드 캐시 정리
```bash
rm -rf dist
rm -rf node_modules/.vite
npm run build
```

---

## 11. 참고 자료

### 11.1 공식 문서
- [Replit Documentation](https://docs.replit.com)
- [React Documentation](https://react.dev)
- [Prisma Documentation](https://www.prisma.io/docs)
- [Express.js Guide](https://expressjs.com/en/guide/routing.html)
- [Neon PostgreSQL](https://neon.tech/docs)

### 11.2 커뮤니티
- [Replit Community](https://replit.com/talk)
- [Replit Discord](https://replit.com/discord)
- [React Discord](https://discord.gg/react)

### 11.3 도구 및 리소스
- [Tailwind CSS Docs](https://tailwindcss.com/docs)
- [Lucide Icons](https://lucide.dev)
- [Prisma Studio](https://www.prisma.io/studio)

---

## 12. 체크리스트

### 12.1 개발 시작 전
- [ ] Replit 계정 생성 (팀원 전원)
- [ ] Neon PostgreSQL 계정 생성
- [ ] 데이콘 API 문서 확인 및 테스트 계정 발급
- [ ] Figma 프로토타입 초안 완성
- [ ] GitHub Repository 생성

### 12.2 MVP 완성 체크
- [ ] Replit Auth로 로그인 작동
- [ ] 프로필 생성 및 수정 가능
- [ ] 원정대 생성 및 역할 슬롯 추가
- [ ] 원정대 리스트 필터링/정렬
- [ ] 지원 플로우 전체 작동
- [ ] 적합도 점수 표시
- [ ] 리더가 지원자 수락/거절 가능
- [ ] 평가 시스템 작동
- [ ] 신뢰 점수 계산 및 표시
- [ ] 모바일 반응형 디자인
- [ ] Replit Deployment 배포 완료

### 12.3 베타 런칭 전
- [ ] 50-100명 베타 테스터 리스트 확보
- [ ] 사용자 가이드 문서 작성
- [ ] FAQ 페이지 준비
- [ ] 피드백 수집 도구 설정
- [ ] 긴급 대응 프로세스 정의
- [ ] 데이콘과 공식 파트너십 확정

---

**문서 작성:** 2025년 11월 1일  
**버전:** 1.0  
**다음 리뷰:** 2주 후 또는 주요 기능 추가 시

---

**Contact**
- 제품 문의: product@dacon-raid.com
- 기술 지원: dev@dacon-raid.com
- 일반 문의: hello@dacon-raid.com
