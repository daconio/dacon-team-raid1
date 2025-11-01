# 🚀 DACON: RAID - Replit 빠른 시작 가이드

**목적**: 5분 안에 프로젝트를 Replit에서 실행하기  
**대상**: 개발 경험이 있는 개발자  
**소요 시간**: 5-10분

---

## 🎯 시작하기 전에

### 필요한 것
- ✅ Replit 계정 (무료) - https://replit.com
- ✅ Neon PostgreSQL 계정 (무료) - https://neon.tech
- ✅ 기본적인 React 및 Node.js 지식

---

## 📦 Step 1: Replit 프로젝트 생성 (2분)

### 1-1. 새 Repl 만들기
```
1. Replit 홈페이지 접속: https://replit.com
2. "Create Repl" 버튼 클릭
3. 템플릿 선택: "Node.js" 또는 "React + Vite" 
4. 프로젝트명 입력: "dacon-raid"
5. Public으로 설정 (팀 협업 용이)
6. "Create Repl" 클릭
```

### 1-2. 기존 코드 업로드 (옵션)
기존 코드가 있다면:
```bash
# Shell 탭에서 실행
git clone https://github.com/your-repo/dacon-raid.git
cd dacon-raid
npm install
```

또는 파일 탭에서 드래그 앤 드롭으로 파일 업로드

---

## 🗄️ Step 2: 데이터베이스 설정 (3분)

### 2-1. Neon PostgreSQL 계정 생성
```
1. https://neon.tech 접속
2. "Sign up" 클릭 → GitHub으로 가입 (권장)
3. 이메일 인증 완료
```

### 2-2. 새 프로젝트 생성
```
1. Neon 대시보드에서 "New Project" 클릭
2. 설정:
   - 프로젝트명: "dacon-raid"
   - 리전: "AWS / US East (Ohio)" (가장 빠름)
   - PostgreSQL 버전: 최신 버전 (기본값)
3. "Create Project" 클릭
```

### 2-3. 연결 문자열 복사
```
1. 프로젝트 생성 후 "Connection String" 표시됨
2. "Pooled connection" 선택
3. 전체 문자열 복사:
   postgresql://user:password@ep-xxx.us-east-1.aws.neon.tech/neondb?sslmode=require
```

### 2-4. Replit에 연결 문자열 저장
```
1. Replit 프로젝트로 돌아가기
2. 왼쪽 사이드바에서 🔒 "Secrets" 탭 클릭
3. 새 Secret 추가:
   - Key: DATABASE_URL
   - Value: (위에서 복사한 연결 문자열 붙여넣기)
4. "Add new secret" 클릭
```

---

## 🏗️ Step 3: 프로젝트 구조 설정 (5분)

### 3-1. 폴더 구조 생성
Replit Shell에서 실행:
```bash
# 기본 폴더 구조 생성
mkdir -p client/src/{components,pages,contexts,components/ui}
mkdir -p server/src/{routes,controllers,middleware,utils}
mkdir -p server/prisma

echo "Folders created!"
```

### 3-2. package.json 생성

#### Root `package.json`
Replit Shell에서 실행:
```bash
cat > package.json << 'EOF'
{
  "name": "dacon-raid",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "npm run dev:server & npm run dev:client",
    "dev:server": "cd server && npm run dev",
    "dev:client": "cd client && npm run dev",
    "build": "cd client && npm run build && cd ../server && npm run build",
    "start": "cd server && npm start"
  },
  "workspaces": [
    "client",
    "server"
  ]
}
EOF
```

#### Client `package.json`
```bash
cat > client/package.json << 'EOF'
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
EOF
```

#### Server `package.json`
```bash
cat > server/package.json << 'EOF'
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
EOF
```

### 3-3. 의존성 설치
```bash
# Root에서 실행 (workspaces 사용)
npm install

# 또는 개별 설치
cd client && npm install && cd ..
cd server && npm install && cd ..
```

---

## 🎨 Step 4: Prisma 스키마 설정 (3분)

### 4-1. Prisma 초기화
```bash
cd server
npx prisma init
```

### 4-2. 스키마 파일 작성
`server/prisma/schema.prisma` 파일을 열고 다음 내용을 붙여넣기:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id                String   @id @default(uuid())
  replitUserId      String   @unique
  email             String   @unique
  displayName       String
  photoURL          String?
  bio               String   @default("")
  
  roles             String[]
  specializations   String[]
  skills            String[]
  
  githubUrl         String?
  kaggleUrl         String?
  blogUrl           String?
  
  trustCommitment      Float    @default(0)
  trustContribution    Float    @default(0)
  trustCommunication   Float    @default(0)
  trustCollaboration   Float    @default(0)
  totalReviews         Int      @default(0)
  
  createdAt         DateTime @default(now())
  updatedAt         DateTime @updatedAt
  
  createdRaids      Raid[]        @relation("RaidCreator")
  raidMemberships   RaidMember[]
  applications      Application[]
  givenReviews      Review[]      @relation("ReviewGiver")
  receivedReviews   Review[]      @relation("ReviewReceiver")
  
  @@index([replitUserId])
  @@index([email])
}

model Competition {
  id          String   @id @default(uuid())
  daconId     String   @unique
  name        String
  type        String
  startDate   DateTime
  endDate     DateTime
  isActive    Boolean  @default(true)
  
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  raids       Raid[]
  
  @@index([daconId])
  @@index([isActive])
}

model Raid {
  id                    String   @id @default(uuid())
  name                  String
  description           String   @db.Text
  status                String   @default("recruiting")
  
  goal                  String
  expectedHoursPerWeek  String
  collaborationMethod   String
  
  createdBy             String
  creator               User         @relation("RaidCreator", fields: [createdBy], references: [id])
  competitionId         String
  competition           Competition  @relation(fields: [competitionId], references: [id])
  
  createdAt             DateTime @default(now())
  updatedAt             DateTime @updatedAt
  
  slots                 RaidSlot[]
  members               RaidMember[]
  applications          Application[]
  reviews               Review[]
  
  @@index([createdBy])
  @@index([competitionId])
  @@index([status])
}

model RaidSlot {
  id        String @id @default(uuid())
  role      String
  level     String
  count     Int
  filled    Int    @default(0)
  
  raidId    String
  raid      Raid   @relation(fields: [raidId], references: [id], onDelete: Cascade)
  
  members   RaidMember[]
  
  @@index([raidId])
}

model RaidMember {
  id        String   @id @default(uuid())
  role      String
  
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  raidId    String
  raid      Raid     @relation(fields: [raidId], references: [id], onDelete: Cascade)
  slotId    String
  slot      RaidSlot @relation(fields: [slotId], references: [id])
  
  joinedAt  DateTime @default(now())
  
  @@unique([userId, raidId])
  @@index([userId])
  @@index([raidId])
}

model Application {
  id              String   @id @default(uuid())
  appliedRole     String
  message         String   @db.Text
  status          String   @default("pending")
  fitScore        Float
  
  applicantId     String
  applicant       User     @relation(fields: [applicantId], references: [id])
  raidId          String
  raid            Raid     @relation(fields: [raidId], references: [id], onDelete: Cascade)
  appliedSlotId   String
  
  appliedAt       DateTime @default(now())
  respondedAt     DateTime?
  
  @@unique([applicantId, raidId])
  @@index([applicantId])
  @@index([raidId])
  @@index([status])
}

model Review {
  id              String   @id @default(uuid())
  
  commitment      Int
  contribution    Int
  communication   Int
  collaboration   Int
  feedback        String?  @db.Text
  
  reviewerId      String
  reviewer        User     @relation("ReviewGiver", fields: [reviewerId], references: [id])
  revieweeId      String
  reviewee        User     @relation("ReviewReceiver", fields: [revieweeId], references: [id])
  raidId          String
  raid            Raid     @relation(fields: [raidId], references: [id], onDelete: Cascade)
  
  createdAt       DateTime @default(now())
  
  @@unique([reviewerId, revieweeId, raidId])
  @@index([revieweeId])
  @@index([raidId])
}
```

### 4-3. 마이그레이션 실행
```bash
# Prisma Client 생성
npx prisma generate

# 데이터베이스 마이그레이션
npx prisma migrate dev --name init

# 성공 메시지 확인
# ✅ Your database is now in sync with your schema.
```

---

## 🖥️ Step 5: Express 서버 생성 (5분)

### 5-1. 서버 진입점 생성
`server/src/index.ts` 파일 생성:

```typescript
import express, { Express, Request, Response } from 'express';
import cors from 'cors';
import dotenv from 'dotenv';
import { PrismaClient } from '@prisma/client';

dotenv.config();

export const prisma = new PrismaClient();

const app: Express = express();
const PORT = process.env.PORT || 3000;

// 미들웨어
app.use(cors({
  origin: process.env.CLIENT_URL || 'http://localhost:5173',
  credentials: true,
}));
app.use(express.json());

// Health Check
app.get('/health', (req: Request, res: Response) => {
  res.json({ status: 'OK', timestamp: new Date().toISOString() });
});

// 간단한 테스트 엔드포인트
app.get('/api/test', async (req: Request, res: Response) => {
  try {
    // DB 연결 테스트
    await prisma.$queryRaw`SELECT 1`;
    res.json({ 
      message: 'Server is running!',
      database: 'Connected',
      timestamp: new Date().toISOString()
    });
  } catch (error) {
    res.status(500).json({ 
      message: 'Server error',
      database: 'Disconnected',
      error: error instanceof Error ? error.message : 'Unknown error'
    });
  }
});

// 404 핸들러
app.use((req: Request, res: Response) => {
  res.status(404).json({ error: 'Endpoint not found' });
});

// 서버 시작
app.listen(PORT, () => {
  console.log(`🚀 Server is running on port ${PORT}`);
  console.log(`📝 Health check: http://localhost:${PORT}/health`);
  console.log(`🧪 Test endpoint: http://localhost:${PORT}/api/test`);
});

// Graceful Shutdown
process.on('SIGTERM', async () => {
  console.log('SIGTERM signal received: closing HTTP server');
  await prisma.$disconnect();
  process.exit(0);
});
```

### 5-2. TypeScript 설정
`server/tsconfig.json` 생성:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "lib": ["ES2022"],
    "moduleResolution": "node",
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

---

## ⚛️ Step 6: React 프론트엔드 생성 (5분)

### 6-1. Vite 설정
`client/vite.config.ts` 생성:

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    proxy: {
      '/api': {
        target: 'http://localhost:3000',
        changeOrigin: true,
      },
    },
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
  },
});
```

### 6-2. Tailwind CSS 설정

`client/tailwind.config.js`:
```javascript
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
          500: '#0ea5e9',
          600: '#0284c7',
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

`client/postcss.config.js`:
```javascript
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

`client/src/index.css`:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  body {
    @apply bg-neutral-900 text-white;
  }
}
```

### 6-3. 기본 App 컴포넌트

`client/src/App.tsx`:
```typescript
import React from 'react';

function App() {
  const [serverStatus, setServerStatus] = React.useState<string>('Checking...');

  React.useEffect(() => {
    fetch('/api/test')
      .then(res => res.json())
      .then(data => setServerStatus(`✅ ${data.message}`))
      .catch(() => setServerStatus('❌ Server not responding'));
  }, []);

  return (
    <div className="min-h-screen bg-neutral-900 text-white flex items-center justify-center">
      <div className="text-center">
        <h1 className="text-4xl font-bold mb-4">🚀 DACON: RAID</h1>
        <p className="text-xl text-neutral-400 mb-8">팀 빌딩 시스템</p>
        <div className="p-4 bg-neutral-800 rounded-lg">
          <p className="text-sm text-neutral-300">Server Status:</p>
          <p className="text-lg font-semibold mt-2">{serverStatus}</p>
        </div>
      </div>
    </div>
  );
}

export default App;
```

`client/src/main.tsx`:
```typescript
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
);
```

`client/index.html`:
```html
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>DACON: RAID</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

### 6-4. TypeScript 설정
`client/tsconfig.json`:
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

---

## 🎮 Step 7: 프로젝트 실행 (1분)

### 7-1. Replit 실행 명령 설정
`.replit` 파일 생성:
```toml
run = "npm run dev"
entrypoint = "client/src/main.tsx"

[nix]
channel = "stable-24_05"

[deployment]
run = ["npm", "run", "start"]
deploymentTarget = "cloudrun"

[[ports]]
localPort = 3000
externalPort = 80

[[ports]]
localPort = 5173
externalPort = 3000
```

### 7-2. 서버 시작
Replit Shell에서:
```bash
# Root에서 실행 (프론트엔드 + 백엔드 동시 실행)
npm run dev
```

또는 개별 실행:
```bash
# 터미널 1: 백엔드
cd server && npm run dev

# 터미널 2: 프론트엔드  
cd client && npm run dev
```

### 7-3. 확인
1. Replit Webview 또는 새 탭에서 프로젝트 URL 열기
2. "Server Status: ✅ Server is running!" 메시지 확인
3. 축하합니다! 🎉 프로젝트가 성공적으로 실행 중입니다.

---

## 🐛 문제 해결

### 데이터베이스 연결 오류
```bash
# DATABASE_URL 확인
echo $DATABASE_URL

# Prisma Client 재생성
cd server
npx prisma generate
npx prisma migrate dev
```

### 포트 충돌
```bash
# 프로세스 확인 및 종료
lsof -ti:3000 | xargs kill -9
lsof -ti:5173 | xargs kill -9
```

### npm install 실패
```bash
# 캐시 정리 후 재설치
npm cache clean --force
rm -rf node_modules package-lock.json
npm install
```

---

## 📚 다음 단계

1. **기능 추가**: [개발 가이드](./DACON_RAID_Replit_개발가이드.md)를 참고하여 주요 기능 구현
2. **UI 디자인**: Tailwind CSS로 컴포넌트 스타일링
3. **API 개발**: Express.js 라우트 및 컨트롤러 작성
4. **테스트**: Postman이나 curl로 API 테스트

---

## 🎉 완료!

이제 Replit에서 DACON: RAID 프로젝트를 실행할 준비가 되었습니다!

**도움이 필요하면:**
- 📖 [전체 개발 가이드](./DACON_RAID_Replit_개발가이드.md)
- 💬 [Replit Discord](https://replit.com/discord)
- 📧 dev@dacon-raid.com
