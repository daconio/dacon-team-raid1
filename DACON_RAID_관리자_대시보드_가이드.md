# 🛡️ DACON: RAID - 관리자 대시보드 개발 가이드

**버전**: 1.0  
**작성일**: 2025년 11월 1일  
**대상**: 완전한 관리자 기능이 필요한 시스템

---

## 📋 목차

1. [관리자 시스템 개요](#1-관리자-시스템-개요)
2. [권한 및 역할 설계](#2-권한-및-역할-설계)
3. [데이터베이스 스키마 확장](#3-데이터베이스-스키마-확장)
4. [백엔드 API 설계](#4-백엔드-api-설계)
5. [프론트엔드 페이지 구조](#5-프론트엔드-페이지-구조)
6. [주요 관리 기능 구현](#6-주요-관리-기능-구현)
7. [통계 및 분석 대시보드](#7-통계-및-분석-대시보드)
8. [보안 및 감사 로그](#8-보안-및-감사-로그)

---

## 1. 관리자 시스템 개요

### 1.1 주요 기능

```
📊 통계 대시보드
   └─ 실시간 KPI 모니터링
   └─ 사용자 증가율 차트
   └─ 팀 매칭 성공률 분석

🏆 대회 관리
   └─ 대회 CRUD (생성, 조회, 수정, 삭제)
   └─ 대회 상태 관리 (활성/비활성)
   └─ 대회 통계 확인

👥 사용자 관리
   └─ 사용자 검색 및 조회
   └─ 계정 상태 관리 (활성/정지/삭제)
   └─ 권한 부여 (관리자, 모더레이터)
   └─ 사용자 활동 내역 조회

🎮 원정대 관리
   └─ 모든 원정대 모니터링
   └─ 부적절한 콘텐츠 삭제
   └─ 원정대 강제 종료

🚨 신고 관리
   └─ 신고 접수 목록
   └─ 신고 처리 (승인/거부)
   └─ 처벌 이력 관리

📈 분석 및 리포트
   └─ 사용자 행동 분석
   └─ 팀 구성 패턴 분석
   └─ 월간/주간 리포트 생성
```

### 1.2 관리자 페이지 접근 경로

```
일반 사용자: https://dacon-raid.com
관리자:      https://dacon-raid.com/admin
```

---

## 2. 권한 및 역할 설계

### 2.1 역할 정의

```typescript
enum UserRole {
  USER = 'user',              // 일반 사용자
  MODERATOR = 'moderator',    // 모더레이터 (콘텐츠 관리)
  ADMIN = 'admin',            // 관리자 (전체 관리)
  SUPER_ADMIN = 'super_admin' // 최고 관리자 (시스템 설정)
}
```

### 2.2 권한 매트릭스

| 기능 | User | Moderator | Admin | Super Admin |
|------|------|-----------|-------|-------------|
| 원정대 생성/참여 | ✅ | ✅ | ✅ | ✅ |
| 부적절한 콘텐츠 삭제 | ❌ | ✅ | ✅ | ✅ |
| 사용자 계정 정지 | ❌ | ❌ | ✅ | ✅ |
| 대회 관리 | ❌ | ❌ | ✅ | ✅ |
| 통계 조회 | ❌ | ✅ | ✅ | ✅ |
| 권한 부여 | ❌ | ❌ | ❌ | ✅ |
| 시스템 설정 | ❌ | ❌ | ❌ | ✅ |

### 2.3 권한 체크 미들웨어

#### `server/src/middleware/admin.middleware.ts`

```typescript
import { Response, NextFunction } from 'express';
import { AuthRequest } from './auth.middleware';
import { prisma } from '../index';

export enum UserRole {
  USER = 'user',
  MODERATOR = 'moderator',
  ADMIN = 'admin',
  SUPER_ADMIN = 'super_admin'
}

// 관리자 권한 체크
export const requireAdmin = async (
  req: AuthRequest,
  res: Response,
  next: NextFunction
) => {
  try {
    if (!req.userId) {
      return res.status(401).json({ error: 'Authentication required' });
    }

    const user = await prisma.user.findUnique({
      where: { id: req.userId },
      select: { role: true }
    });

    if (!user || !['admin', 'super_admin'].includes(user.role)) {
      return res.status(403).json({ error: 'Admin access required' });
    }

    next();
  } catch (error) {
    console.error('Admin check error:', error);
    res.status(500).json({ error: 'Authorization failed' });
  }
};

// 모더레이터 이상 권한 체크
export const requireModerator = async (
  req: AuthRequest,
  res: Response,
  next: NextFunction
) => {
  try {
    if (!req.userId) {
      return res.status(401).json({ error: 'Authentication required' });
    }

    const user = await prisma.user.findUnique({
      where: { id: req.userId },
      select: { role: true }
    });

    if (!user || !['moderator', 'admin', 'super_admin'].includes(user.role)) {
      return res.status(403).json({ error: 'Moderator access required' });
    }

    next();
  } catch (error) {
    console.error('Moderator check error:', error);
    res.status(500).json({ error: 'Authorization failed' });
  }
};

// 최고 관리자 권한 체크
export const requireSuperAdmin = async (
  req: AuthRequest,
  res: Response,
  next: NextFunction
) => {
  try {
    if (!req.userId) {
      return res.status(401).json({ error: 'Authentication required' });
    }

    const user = await prisma.user.findUnique({
      where: { id: req.userId },
      select: { role: true }
    });

    if (!user || user.role !== 'super_admin') {
      return res.status(403).json({ error: 'Super admin access required' });
    }

    next();
  } catch (error) {
    console.error('Super admin check error:', error);
    res.status(500).json({ error: 'Authorization failed' });
  }
};

// 특정 역할 체크
export const requireRole = (allowedRoles: UserRole[]) => {
  return async (req: AuthRequest, res: Response, next: NextFunction) => {
    try {
      if (!req.userId) {
        return res.status(401).json({ error: 'Authentication required' });
      }

      const user = await prisma.user.findUnique({
        where: { id: req.userId },
        select: { role: true }
      });

      if (!user || !allowedRoles.includes(user.role as UserRole)) {
        return res.status(403).json({ 
          error: 'Insufficient permissions',
          required: allowedRoles,
          current: user?.role
        });
      }

      next();
    } catch (error) {
      console.error('Role check error:', error);
      res.status(500).json({ error: 'Authorization failed' });
    }
  };
};
```

---

## 3. 데이터베이스 스키마 확장

### 3.1 User 모델 확장

#### `server/prisma/schema.prisma`

```prisma
model User {
  id                String   @id @default(uuid())
  replitUserId      String   @unique
  email             String   @unique
  displayName       String
  photoURL          String?
  bio               String   @default("")
  
  // 역할 및 권한 (추가)
  role              String   @default("user") // "user", "moderator", "admin", "super_admin"
  isActive          Boolean  @default(true)   // 계정 활성화 여부
  isBanned          Boolean  @default(false)  // 정지 여부
  banReason         String?                   // 정지 사유
  bannedAt          DateTime?                 // 정지 일시
  bannedBy          String?                   // 정지 처리자 ID
  
  // 기존 필드들...
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
  lastLoginAt       DateTime @default(now()) // 마지막 로그인 (추가)
  
  // 관계
  createdRaids      Raid[]        @relation("RaidCreator")
  raidMemberships   RaidMember[]
  applications      Application[]
  givenReviews      Review[]      @relation("ReviewGiver")
  receivedReviews   Review[]      @relation("ReviewReceiver")
  
  // 신고 관련 (추가)
  submittedReports  Report[]      @relation("ReportSubmitter")
  receivedReports   Report[]      @relation("ReportTarget")
  
  // 감사 로그 (추가)
  auditLogs         AuditLog[]
  
  @@index([replitUserId])
  @@index([email])
  @@index([role])
  @@index([isActive])
  @@index([isBanned])
}
```

### 3.2 신고(Report) 모델 추가

```prisma
model Report {
  id              String   @id @default(uuid())
  
  // 신고 대상
  targetType      String   // "user", "raid", "review"
  targetId        String   // 대상 ID
  
  // 신고자
  submitterId     String
  submitter       User     @relation("ReportSubmitter", fields: [submitterId], references: [id])
  
  // 신고 내용
  category        String   // "spam", "harassment", "inappropriate_content", "cheating", "other"
  description     String   @db.Text
  
  // 처리 상태
  status          String   @default("pending") // "pending", "reviewing", "resolved", "rejected"
  resolution      String?  @db.Text // 처리 결과
  resolvedBy      String?  // 처리자 ID
  resolvedAt      DateTime?
  
  // 조치 사항
  actionTaken     String?  // "warning", "ban", "content_removed", "no_action"
  
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  @@index([submitterId])
  @@index([targetType, targetId])
  @@index([status])
  @@index([createdAt])
}
```

### 3.3 감사 로그(AuditLog) 모델 추가

```prisma
model AuditLog {
  id              String   @id @default(uuid())
  
  // 행위자
  userId          String
  user            User     @relation(fields: [userId], references: [id])
  
  // 행위 정보
  action          String   // "user_banned", "competition_created", "report_resolved" 등
  targetType      String?  // "user", "raid", "competition" 등
  targetId        String?  // 대상 ID
  
  // 상세 정보
  details         Json?    // 추가 상세 정보 (JSON)
  ipAddress       String?  // IP 주소
  userAgent       String?  // User Agent
  
  createdAt       DateTime @default(now())
  
  @@index([userId])
  @@index([action])
  @@index([targetType, targetId])
  @@index([createdAt])
}
```

### 3.4 마이그레이션 실행

```bash
cd server
npx prisma migrate dev --name add_admin_features
npx prisma generate
```

---

## 4. 백엔드 API 설계

### 4.1 관리자 라우트 구조

#### `server/src/routes/admin.routes.ts`

```typescript
import { Router } from 'express';
import { authenticateUser } from '../middleware/auth.middleware';
import { requireAdmin, requireModerator, requireSuperAdmin } from '../middleware/admin.middleware';

const router = Router();

// 모든 관리자 라우트는 인증 필요
router.use(authenticateUser);

// 통계 대시보드 (모더레이터 이상)
import * as dashboardController from '../controllers/admin/dashboard.controller';
router.get('/dashboard/stats', requireModerator, dashboardController.getStats);
router.get('/dashboard/charts', requireModerator, dashboardController.getCharts);

// 사용자 관리 (관리자 이상)
import * as userController from '../controllers/admin/user.controller';
router.get('/users', requireAdmin, userController.listUsers);
router.get('/users/:id', requireAdmin, userController.getUser);
router.patch('/users/:id/ban', requireAdmin, userController.banUser);
router.patch('/users/:id/unban', requireAdmin, userController.unbanUser);
router.patch('/users/:id/role', requireSuperAdmin, userController.updateUserRole);
router.delete('/users/:id', requireSuperAdmin, userController.deleteUser);

// 대회 관리 (관리자 이상)
import * as competitionController from '../controllers/admin/competition.controller';
router.post('/competitions', requireAdmin, competitionController.createCompetition);
router.patch('/competitions/:id', requireAdmin, competitionController.updateCompetition);
router.delete('/competitions/:id', requireAdmin, competitionController.deleteCompetition);
router.patch('/competitions/:id/activate', requireAdmin, competitionController.activateCompetition);
router.patch('/competitions/:id/deactivate', requireAdmin, competitionController.deactivateCompetition);

// 원정대 관리 (모더레이터 이상)
import * as raidController from '../controllers/admin/raid.controller';
router.get('/raids', requireModerator, raidController.listAllRaids);
router.delete('/raids/:id', requireModerator, raidController.deleteRaid);
router.patch('/raids/:id/status', requireAdmin, raidController.updateRaidStatus);

// 신고 관리 (모더레이터 이상)
import * as reportController from '../controllers/admin/report.controller';
router.get('/reports', requireModerator, reportController.listReports);
router.get('/reports/:id', requireModerator, reportController.getReport);
router.patch('/reports/:id/resolve', requireModerator, reportController.resolveReport);
router.patch('/reports/:id/reject', requireModerator, reportController.rejectReport);

// 감사 로그 (관리자 이상)
import * as auditController from '../controllers/admin/audit.controller';
router.get('/audit-logs', requireAdmin, auditController.listAuditLogs);

export default router;
```

#### `server/src/index.ts`에 라우트 추가

```typescript
import adminRoutes from './routes/admin.routes';

// ... 기존 라우트들
app.use('/api/admin', adminRoutes);
```

### 4.2 대시보드 컨트롤러

#### `server/src/controllers/admin/dashboard.controller.ts`

```typescript
import { Response } from 'express';
import { AuthRequest } from '../../middleware/auth.middleware';
import { prisma } from '../../index';

// 통계 요약
export const getStats = async (req: AuthRequest, res: Response) => {
  try {
    const [
      totalUsers,
      totalRaids,
      activeRaids,
      totalApplications,
      pendingReports,
      newUsersThisWeek,
      newRaidsThisWeek
    ] = await Promise.all([
      // 전체 사용자 수
      prisma.user.count(),
      
      // 전체 원정대 수
      prisma.raid.count(),
      
      // 활성 원정대 수
      prisma.raid.count({ where: { status: { in: ['recruiting', 'active'] } } }),
      
      // 전체 지원 수
      prisma.application.count(),
      
      // 처리 대기 신고 수
      prisma.report.count({ where: { status: 'pending' } }),
      
      // 이번 주 신규 사용자
      prisma.user.count({
        where: {
          createdAt: {
            gte: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000)
          }
        }
      }),
      
      // 이번 주 신규 원정대
      prisma.raid.count({
        where: {
          createdAt: {
            gte: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000)
          }
        }
      })
    ]);

    // 팀 매칭 성공률 계산
    const acceptedApplications = await prisma.application.count({
      where: { status: 'accepted' }
    });
    const matchSuccessRate = totalApplications > 0 
      ? (acceptedApplications / totalApplications) * 100 
      : 0;

    // 평균 신뢰 점수
    const users = await prisma.user.findMany({
      select: {
        trustCommitment: true,
        trustContribution: true,
        trustCommunication: true,
        trustCollaboration: true,
      }
    });

    const avgTrustScore = users.length > 0
      ? users.reduce((sum, user) => {
          const userAvg = (
            user.trustCommitment +
            user.trustContribution +
            user.trustCommunication +
            user.trustCollaboration
          ) / 4;
          return sum + userAvg;
        }, 0) / users.length
      : 0;

    res.json({
      totalUsers,
      totalRaids,
      activeRaids,
      totalApplications,
      matchSuccessRate: matchSuccessRate.toFixed(2),
      avgTrustScore: avgTrustScore.toFixed(2),
      pendingReports,
      newUsersThisWeek,
      newRaidsThisWeek,
    });
  } catch (error) {
    console.error('Get stats error:', error);
    res.status(500).json({ error: 'Failed to fetch statistics' });
  }
};

// 차트 데이터
export const getCharts = async (req: AuthRequest, res: Response) => {
  try {
    const { period = '30d' } = req.query;
    
    const days = period === '7d' ? 7 : period === '30d' ? 30 : 90;
    const startDate = new Date(Date.now() - days * 24 * 60 * 60 * 1000);

    // 일별 사용자 증가 추이
    const userGrowth = await prisma.$queryRaw<Array<{ date: Date; count: bigint }>>`
      SELECT DATE(created_at) as date, COUNT(*) as count
      FROM "User"
      WHERE created_at >= ${startDate}
      GROUP BY DATE(created_at)
      ORDER BY date ASC
    `;

    // 일별 원정대 생성 추이
    const raidCreation = await prisma.$queryRaw<Array<{ date: Date; count: bigint }>>`
      SELECT DATE(created_at) as date, COUNT(*) as count
      FROM "Raid"
      WHERE created_at >= ${startDate}
      GROUP BY DATE(created_at)
      ORDER BY date ASC
    `;

    // 역할별 사용자 분포
    const roleDistribution = await prisma.$queryRaw<Array<{ role: string; count: bigint }>>`
      SELECT role, COUNT(*) as count
      FROM "User"
      GROUP BY role
    `;

    // 원정대 목표별 분포
    const goalDistribution = await prisma.$queryRaw<Array<{ goal: string; count: bigint }>>`
      SELECT goal, COUNT(*) as count
      FROM "Raid"
      GROUP BY goal
    `;

    res.json({
      userGrowth: userGrowth.map(item => ({
        date: item.date,
        count: Number(item.count)
      })),
      raidCreation: raidCreation.map(item => ({
        date: item.date,
        count: Number(item.count)
      })),
      roleDistribution: roleDistribution.map(item => ({
        role: item.role,
        count: Number(item.count)
      })),
      goalDistribution: goalDistribution.map(item => ({
        goal: item.goal,
        count: Number(item.count)
      })),
    });
  } catch (error) {
    console.error('Get charts error:', error);
    res.status(500).json({ error: 'Failed to fetch chart data' });
  }
};
```

### 4.3 사용자 관리 컨트롤러

#### `server/src/controllers/admin/user.controller.ts`

```typescript
import { Response } from 'express';
import { AuthRequest } from '../../middleware/auth.middleware';
import { prisma } from '../../index';
import { createAuditLog } from '../../utils/auditLog';

// 사용자 목록 조회
export const listUsers = async (req: AuthRequest, res: Response) => {
  try {
    const {
      search,
      role,
      isActive,
      isBanned,
      sortBy = 'createdAt',
      order = 'desc',
      page = '1',
      limit = '20'
    } = req.query;

    const where: any = {};

    // 검색 필터
    if (search) {
      where.OR = [
        { displayName: { contains: search as string, mode: 'insensitive' } },
        { email: { contains: search as string, mode: 'insensitive' } },
      ];
    }

    if (role) where.role = role;
    if (isActive !== undefined) where.isActive = isActive === 'true';
    if (isBanned !== undefined) where.isBanned = isBanned === 'true';

    const skip = (parseInt(page as string) - 1) * parseInt(limit as string);
    const take = parseInt(limit as string);

    const [users, total] = await Promise.all([
      prisma.user.findMany({
        where,
        select: {
          id: true,
          displayName: true,
          email: true,
          photoURL: true,
          role: true,
          isActive: true,
          isBanned: true,
          banReason: true,
          bannedAt: true,
          createdAt: true,
          lastLoginAt: true,
          _count: {
            select: {
              createdRaids: true,
              raidMemberships: true,
              applications: true,
            }
          }
        },
        orderBy: { [sortBy as string]: order },
        skip,
        take,
      }),
      prisma.user.count({ where })
    ]);

    res.json({
      users,
      pagination: {
        page: parseInt(page as string),
        limit: parseInt(limit as string),
        total,
        totalPages: Math.ceil(total / parseInt(limit as string))
      }
    });
  } catch (error) {
    console.error('List users error:', error);
    res.status(500).json({ error: 'Failed to fetch users' });
  }
};

// 특정 사용자 상세 조회
export const getUser = async (req: AuthRequest, res: Response) => {
  try {
    const { id } = req.params;

    const user = await prisma.user.findUnique({
      where: { id },
      include: {
        createdRaids: {
          select: {
            id: true,
            name: true,
            status: true,
            createdAt: true,
          }
        },
        raidMemberships: {
          include: {
            raid: {
              select: {
                id: true,
                name: true,
                status: true,
              }
            }
          }
        },
        applications: {
          select: {
            id: true,
            status: true,
            appliedAt: true,
            raid: {
              select: {
                id: true,
                name: true,
              }
            }
          }
        },
        receivedReports: {
          where: { status: { in: ['pending', 'reviewing'] } },
          select: {
            id: true,
            category: true,
            status: true,
            createdAt: true,
          }
        }
      }
    });

    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }

    res.json(user);
  } catch (error) {
    console.error('Get user error:', error);
    res.status(500).json({ error: 'Failed to fetch user' });
  }
};

// 사용자 정지
export const banUser = async (req: AuthRequest, res: Response) => {
  try {
    const { id } = req.params;
    const { reason, duration } = req.body; // duration: 'permanent' | days (숫자)

    if (!reason) {
      return res.status(400).json({ error: 'Ban reason is required' });
    }

    const bannedUntil = duration === 'permanent' 
      ? null 
      : new Date(Date.now() + duration * 24 * 60 * 60 * 1000);

    const user = await prisma.user.update({
      where: { id },
      data: {
        isBanned: true,
        isActive: false,
        banReason: reason,
        bannedAt: new Date(),
        bannedBy: req.userId,
      }
    });

    // 감사 로그 기록
    await createAuditLog({
      userId: req.userId!,
      action: 'user_banned',
      targetType: 'user',
      targetId: id,
      details: {
        reason,
        duration: duration === 'permanent' ? 'permanent' : `${duration} days`,
        bannedUntil,
      },
      ipAddress: req.ip,
      userAgent: req.get('user-agent'),
    });

    res.json({
      message: 'User banned successfully',
      user
    });
  } catch (error) {
    console.error('Ban user error:', error);
    res.status(500).json({ error: 'Failed to ban user' });
  }
};

// 사용자 정지 해제
export const unbanUser = async (req: AuthRequest, res: Response) => {
  try {
    const { id } = req.params;

    const user = await prisma.user.update({
      where: { id },
      data: {
        isBanned: false,
        isActive: true,
        banReason: null,
        bannedAt: null,
        bannedBy: null,
      }
    });

    // 감사 로그 기록
    await createAuditLog({
      userId: req.userId!,
      action: 'user_unbanned',
      targetType: 'user',
      targetId: id,
      ipAddress: req.ip,
      userAgent: req.get('user-agent'),
    });

    res.json({
      message: 'User unbanned successfully',
      user
    });
  } catch (error) {
    console.error('Unban user error:', error);
    res.status(500).json({ error: 'Failed to unban user' });
  }
};

// 사용자 역할 변경 (Super Admin만)
export const updateUserRole = async (req: AuthRequest, res: Response) => {
  try {
    const { id } = req.params;
    const { role } = req.body;

    const validRoles = ['user', 'moderator', 'admin', 'super_admin'];
    if (!validRoles.includes(role)) {
      return res.status(400).json({ error: 'Invalid role' });
    }

    const user = await prisma.user.update({
      where: { id },
      data: { role }
    });

    // 감사 로그 기록
    await createAuditLog({
      userId: req.userId!,
      action: 'user_role_updated',
      targetType: 'user',
      targetId: id,
      details: { newRole: role },
      ipAddress: req.ip,
      userAgent: req.get('user-agent'),
    });

    res.json({
      message: 'User role updated successfully',
      user
    });
  } catch (error) {
    console.error('Update user role error:', error);
    res.status(500).json({ error: 'Failed to update user role' });
  }
};

// 사용자 삭제 (Super Admin만)
export const deleteUser = async (req: AuthRequest, res: Response) => {
  try {
    const { id } = req.params;

    // 사용자 삭제 (Cascade로 관련 데이터도 삭제)
    await prisma.user.delete({
      where: { id }
    });

    // 감사 로그 기록
    await createAuditLog({
      userId: req.userId!,
      action: 'user_deleted',
      targetType: 'user',
      targetId: id,
      ipAddress: req.ip,
      userAgent: req.get('user-agent'),
    });

    res.json({ message: 'User deleted successfully' });
  } catch (error) {
    console.error('Delete user error:', error);
    res.status(500).json({ error: 'Failed to delete user' });
  }
};
```

### 4.4 신고 관리 컨트롤러

#### `server/src/controllers/admin/report.controller.ts`

```typescript
import { Response } from 'express';
import { AuthRequest } from '../../middleware/auth.middleware';
import { prisma } from '../../index';
import { createAuditLog } from '../../utils/auditLog';

// 신고 목록 조회
export const listReports = async (req: AuthRequest, res: Response) => {
  try {
    const {
      status,
      category,
      targetType,
      sortBy = 'createdAt',
      order = 'desc',
      page = '1',
      limit = '20'
    } = req.query;

    const where: any = {};

    if (status) where.status = status;
    if (category) where.category = category;
    if (targetType) where.targetType = targetType;

    const skip = (parseInt(page as string) - 1) * parseInt(limit as string);
    const take = parseInt(limit as string);

    const [reports, total] = await Promise.all([
      prisma.report.findMany({
        where,
        include: {
          submitter: {
            select: {
              id: true,
              displayName: true,
              email: true,
              photoURL: true,
            }
          }
        },
        orderBy: { [sortBy as string]: order },
        skip,
        take,
      }),
      prisma.report.count({ where })
    ]);

    res.json({
      reports,
      pagination: {
        page: parseInt(page as string),
        limit: parseInt(limit as string),
        total,
        totalPages: Math.ceil(total / parseInt(limit as string))
      }
    });
  } catch (error) {
    console.error('List reports error:', error);
    res.status(500).json({ error: 'Failed to fetch reports' });
  }
};

// 특정 신고 상세 조회
export const getReport = async (req: AuthRequest, res: Response) => {
  try {
    const { id } = req.params;

    const report = await prisma.report.findUnique({
      where: { id },
      include: {
        submitter: {
          select: {
            id: true,
            displayName: true,
            email: true,
            photoURL: true,
          }
        }
      }
    });

    if (!report) {
      return res.status(404).json({ error: 'Report not found' });
    }

    // 신고 대상 정보도 함께 조회
    let targetData = null;
    if (report.targetType === 'user') {
      targetData = await prisma.user.findUnique({
        where: { id: report.targetId },
        select: {
          id: true,
          displayName: true,
          email: true,
          photoURL: true,
          isBanned: true,
        }
      });
    } else if (report.targetType === 'raid') {
      targetData = await prisma.raid.findUnique({
        where: { id: report.targetId },
        select: {
          id: true,
          name: true,
          description: true,
          status: true,
          creator: {
            select: {
              id: true,
              displayName: true,
            }
          }
        }
      });
    }

    res.json({
      ...report,
      targetData
    });
  } catch (error) {
    console.error('Get report error:', error);
    res.status(500).json({ error: 'Failed to fetch report' });
  }
};

// 신고 처리 (승인)
export const resolveReport = async (req: AuthRequest, res: Response) => {
  try {
    const { id } = req.params;
    const { resolution, actionTaken } = req.body;

    if (!resolution || !actionTaken) {
      return res.status(400).json({ error: 'Resolution and action are required' });
    }

    const report = await prisma.report.update({
      where: { id },
      data: {
        status: 'resolved',
        resolution,
        actionTaken,
        resolvedBy: req.userId,
        resolvedAt: new Date(),
      },
      include: {
        submitter: true
      }
    });

    // 조치 사항 실행
    if (actionTaken === 'ban' && report.targetType === 'user') {
      await prisma.user.update({
        where: { id: report.targetId },
        data: {
          isBanned: true,
          isActive: false,
          banReason: `신고 처리: ${resolution}`,
          bannedAt: new Date(),
          bannedBy: req.userId,
        }
      });
    } else if (actionTaken === 'content_removed' && report.targetType === 'raid') {
      await prisma.raid.delete({
        where: { id: report.targetId }
      });
    }

    // 감사 로그 기록
    await createAuditLog({
      userId: req.userId!,
      action: 'report_resolved',
      targetType: 'report',
      targetId: id,
      details: {
        reportTargetType: report.targetType,
        reportTargetId: report.targetId,
        actionTaken,
        resolution,
      },
      ipAddress: req.ip,
      userAgent: req.get('user-agent'),
    });

    res.json({
      message: 'Report resolved successfully',
      report
    });
  } catch (error) {
    console.error('Resolve report error:', error);
    res.status(500).json({ error: 'Failed to resolve report' });
  }
};

// 신고 거부
export const rejectReport = async (req: AuthRequest, res: Response) => {
  try {
    const { id } = req.params;
    const { reason } = req.body;

    const report = await prisma.report.update({
      where: { id },
      data: {
        status: 'rejected',
        resolution: reason || '신고 내용이 타당하지 않음',
        resolvedBy: req.userId,
        resolvedAt: new Date(),
      }
    });

    // 감사 로그 기록
    await createAuditLog({
      userId: req.userId!,
      action: 'report_rejected',
      targetType: 'report',
      targetId: id,
      details: { reason },
      ipAddress: req.ip,
      userAgent: req.get('user-agent'),
    });

    res.json({
      message: 'Report rejected successfully',
      report
    });
  } catch (error) {
    console.error('Reject report error:', error);
    res.status(500).json({ error: 'Failed to reject report' });
  }
};
```

### 4.5 감사 로그 유틸리티

#### `server/src/utils/auditLog.ts`

```typescript
import { prisma } from '../index';

interface CreateAuditLogParams {
  userId: string;
  action: string;
  targetType?: string;
  targetId?: string;
  details?: any;
  ipAddress?: string;
  userAgent?: string;
}

export async function createAuditLog(params: CreateAuditLogParams) {
  try {
    await prisma.auditLog.create({
      data: {
        userId: params.userId,
        action: params.action,
        targetType: params.targetType,
        targetId: params.targetId,
        details: params.details,
        ipAddress: params.ipAddress,
        userAgent: params.userAgent,
      }
    });
  } catch (error) {
    console.error('Failed to create audit log:', error);
    // 감사 로그 실패는 메인 작업을 방해하지 않음
  }
}
```

---

## 5. 프론트엔드 페이지 구조

### 5.1 관리자 라우트 설정

#### `client/src/App.tsx`

```typescript
import React from 'react';
import { HashRouter, Routes, Route, Navigate } from 'react-router-dom';
import { DataProvider } from './contexts/DataContext';
import { AuthProvider, useAuth } from './contexts/AuthContext';

// 일반 사용자 페이지
import Header from './components/Header';
import HomePage from './pages/HomePage';
import RaidDetailPage from './pages/RaidDetailPage';
import ProfilePage from './pages/ProfilePage';
import CreateRaidPage from './pages/CreateRaidPage';
import DashboardPage from './pages/DashboardPage';

// 관리자 페이지
import AdminLayout from './components/admin/AdminLayout';
import AdminDashboard from './pages/admin/AdminDashboard';
import UserManagement from './pages/admin/UserManagement';
import CompetitionManagement from './pages/admin/CompetitionManagement';
import RaidManagement from './pages/admin/RaidManagement';
import ReportManagement from './pages/admin/ReportManagement';
import AuditLogs from './pages/admin/AuditLogs';

// 관리자 권한 체크 컴포넌트
const AdminRoute: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { user } = useAuth();
  
  if (!user) {
    return <Navigate to="/" replace />;
  }
  
  if (!['moderator', 'admin', 'super_admin'].includes(user.role)) {
    return <Navigate to="/" replace />;
  }
  
  return <>{children}</>;
};

const App: React.FC = () => {
  return (
    <AuthProvider>
      <DataProvider>
        <HashRouter>
          <Routes>
            {/* 일반 사용자 라우트 */}
            <Route path="/" element={
              <div className="min-h-screen bg-neutral-900">
                <Header />
                <main className="container mx-auto px-4 py-8">
                  <HomePage />
                </main>
              </div>
            } />
            <Route path="/raid/:raidId" element={
              <div className="min-h-screen bg-neutral-900">
                <Header />
                <main className="container mx-auto px-4 py-8">
                  <RaidDetailPage />
                </main>
              </div>
            } />
            <Route path="/profile/:userId" element={
              <div className="min-h-screen bg-neutral-900">
                <Header />
                <main className="container mx-auto px-4 py-8">
                  <ProfilePage />
                </main>
              </div>
            } />
            <Route path="/create-raid" element={
              <div className="min-h-screen bg-neutral-900">
                <Header />
                <main className="container mx-auto px-4 py-8">
                  <CreateRaidPage />
                </main>
              </div>
            } />
            <Route path="/dashboard" element={
              <div className="min-h-screen bg-neutral-900">
                <Header />
                <main className="container mx-auto px-4 py-8">
                  <DashboardPage />
                </main>
              </div>
            } />

            {/* 관리자 라우트 */}
            <Route path="/admin" element={
              <AdminRoute>
                <AdminLayout />
              </AdminRoute>
            }>
              <Route index element={<Navigate to="/admin/dashboard" replace />} />
              <Route path="dashboard" element={<AdminDashboard />} />
              <Route path="users" element={<UserManagement />} />
              <Route path="competitions" element={<CompetitionManagement />} />
              <Route path="raids" element={<RaidManagement />} />
              <Route path="reports" element={<ReportManagement />} />
              <Route path="audit-logs" element={<AuditLogs />} />
            </Route>
          </Routes>
        </HashRouter>
      </DataProvider>
    </AuthProvider>
  );
};

export default App;
```

### 5.2 관리자 레이아웃

#### `client/src/components/admin/AdminLayout.tsx`

```typescript
import React from 'react';
import { Outlet, Link, useLocation } from 'react-router-dom';
import { 
  LayoutDashboard, 
  Users, 
  Trophy, 
  Shield, 
  Flag, 
  FileText,
  LogOut 
} from 'lucide-react';
import { useAuth } from '../../contexts/AuthContext';

const AdminLayout: React.FC = () => {
  const location = useLocation();
  const { user, logout } = useAuth();

  const menuItems = [
    { path: '/admin/dashboard', icon: LayoutDashboard, label: '대시보드' },
    { path: '/admin/users', icon: Users, label: '사용자 관리' },
    { path: '/admin/competitions', icon: Trophy, label: '대회 관리' },
    { path: '/admin/raids', icon: Shield, label: '원정대 관리' },
    { path: '/admin/reports', icon: Flag, label: '신고 관리' },
    { path: '/admin/audit-logs', icon: FileText, label: '감사 로그' },
  ];

  return (
    <div className="min-h-screen bg-neutral-900 flex">
      {/* 사이드바 */}
      <aside className="w-64 bg-neutral-800 border-r border-neutral-700">
        {/* 로고 */}
        <div className="p-6 border-b border-neutral-700">
          <h1 className="text-xl font-bold text-white">RAID 관리자</h1>
          <p className="text-sm text-neutral-400 mt-1">{user?.displayName}</p>
          <span className="inline-block mt-2 px-2 py-1 text-xs font-semibold bg-primary-500 text-white rounded">
            {user?.role === 'super_admin' ? '최고 관리자' : 
             user?.role === 'admin' ? '관리자' : '모더레이터'}
          </span>
        </div>

        {/* 메뉴 */}
        <nav className="p-4">
          <ul className="space-y-2">
            {menuItems.map((item) => {
              const Icon = item.icon;
              const isActive = location.pathname === item.path;
              
              return (
                <li key={item.path}>
                  <Link
                    to={item.path}
                    className={`flex items-center gap-3 px-4 py-3 rounded-lg transition-colors ${
                      isActive
                        ? 'bg-primary-500 text-white'
                        : 'text-neutral-300 hover:bg-neutral-700'
                    }`}
                  >
                    <Icon className="w-5 h-5" />
                    <span>{item.label}</span>
                  </Link>
                </li>
              );
            })}
          </ul>
        </nav>

        {/* 하단 버튼 */}
        <div className="absolute bottom-0 w-64 p-4 border-t border-neutral-700">
          <Link
            to="/"
            className="flex items-center gap-3 px-4 py-3 rounded-lg text-neutral-300 hover:bg-neutral-700 transition-colors mb-2"
          >
            <LayoutDashboard className="w-5 h-5" />
            <span>사용자 페이지로</span>
          </Link>
          <button
            onClick={logout}
            className="w-full flex items-center gap-3 px-4 py-3 rounded-lg text-red-400 hover:bg-neutral-700 transition-colors"
          >
            <LogOut className="w-5 h-5" />
            <span>로그아웃</span>
          </button>
        </div>
      </aside>

      {/* 메인 콘텐츠 */}
      <main className="flex-1 overflow-y-auto">
        <div className="container mx-auto px-8 py-6">
          <Outlet />
        </div>
      </main>
    </div>
  );
};

export default AdminLayout;
```

---

## 6. 주요 관리 기능 구현

### 6.1 통계 대시보드

#### `client/src/pages/admin/AdminDashboard.tsx`

```typescript
import React, { useEffect, useState } from 'react';
import { Users, Shield, TrendingUp, AlertCircle, CheckCircle, Clock } from 'lucide-react';
import { Line, Bar, Pie } from 'react-chartjs-2';
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  BarElement,
  ArcElement,
  Title,
  Tooltip,
  Legend,
} from 'chart.js';

ChartJS.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  BarElement,
  ArcElement,
  Title,
  Tooltip,
  Legend
);

interface Stats {
  totalUsers: number;
  totalRaids: number;
  activeRaids: number;
  totalApplications: number;
  matchSuccessRate: string;
  avgTrustScore: string;
  pendingReports: number;
  newUsersThisWeek: number;
  newRaidsThisWeek: number;
}

interface ChartData {
  userGrowth: Array<{ date: string; count: number }>;
  raidCreation: Array<{ date: string; count: number }>;
  roleDistribution: Array<{ role: string; count: number }>;
  goalDistribution: Array<{ goal: string; count: number }>;
}

const AdminDashboard: React.FC = () => {
  const [stats, setStats] = useState<Stats | null>(null);
  const [chartData, setChartData] = useState<ChartData | null>(null);
  const [loading, setLoading] = useState(true);
  const [period, setPeriod] = useState('30d');

  useEffect(() => {
    fetchStats();
    fetchChartData();
  }, [period]);

  const fetchStats = async () => {
    try {
      const response = await fetch('/api/admin/dashboard/stats');
      const data = await response.json();
      setStats(data);
    } catch (error) {
      console.error('Failed to fetch stats:', error);
    }
  };

  const fetchChartData = async () => {
    try {
      const response = await fetch(`/api/admin/dashboard/charts?period=${period}`);
      const data = await response.json();
      setChartData(data);
    } catch (error) {
      console.error('Failed to fetch chart data:', error);
    } finally {
      setLoading(false);
    }
  };

  if (loading || !stats || !chartData) {
    return <div className="text-center py-20 text-white">로딩 중...</div>;
  }

  // KPI 카드 데이터
  const kpiCards = [
    {
      title: '전체 사용자',
      value: stats.totalUsers.toLocaleString(),
      icon: Users,
      color: 'bg-blue-500',
      trend: `+${stats.newUsersThisWeek} 이번 주`,
    },
    {
      title: '활성 원정대',
      value: stats.activeRaids.toLocaleString(),
      icon: Shield,
      color: 'bg-green-500',
      trend: `전체 ${stats.totalRaids}개`,
    },
    {
      title: '매칭 성공률',
      value: `${stats.matchSuccessRate}%`,
      icon: TrendingUp,
      color: 'bg-purple-500',
      trend: `${stats.totalApplications} 지원`,
    },
    {
      title: '처리 대기 신고',
      value: stats.pendingReports.toLocaleString(),
      icon: AlertCircle,
      color: 'bg-red-500',
      trend: '빠른 처리 필요',
    },
  ];

  // 사용자 증가 차트 데이터
  const userGrowthChartData = {
    labels: chartData.userGrowth.map(item => 
      new Date(item.date).toLocaleDateString('ko-KR', { month: 'short', day: 'numeric' })
    ),
    datasets: [
      {
        label: '신규 사용자',
        data: chartData.userGrowth.map(item => item.count),
        borderColor: 'rgb(59, 130, 246)',
        backgroundColor: 'rgba(59, 130, 246, 0.1)',
        fill: true,
      },
    ],
  };

  // 역할별 분포 차트 데이터
  const roleDistributionChartData = {
    labels: chartData.roleDistribution.map(item => item.role),
    datasets: [
      {
        data: chartData.roleDistribution.map(item => item.count),
        backgroundColor: [
          'rgba(59, 130, 246, 0.8)',
          'rgba(16, 185, 129, 0.8)',
          'rgba(245, 158, 11, 0.8)',
          'rgba(239, 68, 68, 0.8)',
        ],
      },
    ],
  };

  return (
    <div>
      {/* 헤더 */}
      <div className="mb-8">
        <h1 className="text-3xl font-bold text-white mb-2">대시보드</h1>
        <p className="text-neutral-400">RAID 시스템 전체 현황을 한눈에 확인하세요</p>
      </div>

      {/* KPI 카드 */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
        {kpiCards.map((card, index) => {
          const Icon = card.icon;
          return (
            <div key={index} className="bg-neutral-800 rounded-xl p-6 border border-neutral-700">
              <div className="flex items-center justify-between mb-4">
                <div className={`${card.color} p-3 rounded-lg`}>
                  <Icon className="w-6 h-6 text-white" />
                </div>
              </div>
              <h3 className="text-neutral-400 text-sm font-medium mb-1">{card.title}</h3>
              <p className="text-white text-3xl font-bold mb-2">{card.value}</p>
              <p className="text-neutral-500 text-sm">{card.trend}</p>
            </div>
          );
        })}
      </div>

      {/* 기간 선택 */}
      <div className="mb-6">
        <div className="flex gap-2">
          {['7d', '30d', '90d'].map((p) => (
            <button
              key={p}
              onClick={() => setPeriod(p)}
              className={`px-4 py-2 rounded-lg transition-colors ${
                period === p
                  ? 'bg-primary-500 text-white'
                  : 'bg-neutral-800 text-neutral-300 hover:bg-neutral-700'
              }`}
            >
              {p === '7d' ? '7일' : p === '30d' ? '30일' : '90일'}
            </button>
          ))}
        </div>
      </div>

      {/* 차트 그리드 */}
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        {/* 사용자 증가 추이 */}
        <div className="bg-neutral-800 rounded-xl p-6 border border-neutral-700">
          <h3 className="text-white text-lg font-semibold mb-4">사용자 증가 추이</h3>
          <Line 
            data={userGrowthChartData}
            options={{
              responsive: true,
              plugins: {
                legend: { display: false },
              },
              scales: {
                x: {
                  grid: { color: 'rgba(255, 255, 255, 0.1)' },
                  ticks: { color: '#a3a3a3' },
                },
                y: {
                  grid: { color: 'rgba(255, 255, 255, 0.1)' },
                  ticks: { color: '#a3a3a3' },
                },
              },
            }}
          />
        </div>

        {/* 역할별 사용자 분포 */}
        <div className="bg-neutral-800 rounded-xl p-6 border border-neutral-700">
          <h3 className="text-white text-lg font-semibold mb-4">역할별 사용자 분포</h3>
          <Pie 
            data={roleDistributionChartData}
            options={{
              responsive: true,
              plugins: {
                legend: {
                  position: 'bottom',
                  labels: { color: '#a3a3a3' },
                },
              },
            }}
          />
        </div>

        {/* 원정대 생성 추이 */}
        <div className="bg-neutral-800 rounded-xl p-6 border border-neutral-700">
          <h3 className="text-white text-lg font-semibold mb-4">원정대 생성 추이</h3>
          <Bar 
            data={{
              labels: chartData.raidCreation.map(item => 
                new Date(item.date).toLocaleDateString('ko-KR', { month: 'short', day: 'numeric' })
              ),
              datasets: [
                {
                  label: '신규 원정대',
                  data: chartData.raidCreation.map(item => item.count),
                  backgroundColor: 'rgba(16, 185, 129, 0.8)',
                },
              ],
            }}
            options={{
              responsive: true,
              plugins: {
                legend: { display: false },
              },
              scales: {
                x: {
                  grid: { color: 'rgba(255, 255, 255, 0.1)' },
                  ticks: { color: '#a3a3a3' },
                },
                y: {
                  grid: { color: 'rgba(255, 255, 255, 0.1)' },
                  ticks: { color: '#a3a3a3' },
                },
              },
            }}
          />
        </div>

        {/* 원정대 목표별 분포 */}
        <div className="bg-neutral-800 rounded-xl p-6 border border-neutral-700">
          <h3 className="text-white text-lg font-semibold mb-4">원정대 목표별 분포</h3>
          <Bar 
            data={{
              labels: chartData.goalDistribution.map(item => item.goal),
              datasets: [
                {
                  label: '원정대 수',
                  data: chartData.goalDistribution.map(item => item.count),
                  backgroundColor: [
                    'rgba(245, 158, 11, 0.8)',
                    'rgba(239, 68, 68, 0.8)',
                    'rgba(59, 130, 246, 0.8)',
                  ],
                },
              ],
            }}
            options={{
              responsive: true,
              plugins: {
                legend: { display: false },
              },
              scales: {
                x: {
                  grid: { color: 'rgba(255, 255, 255, 0.1)' },
                  ticks: { color: '#a3a3a3' },
                },
                y: {
                  grid: { color: 'rgba(255, 255, 255, 0.1)' },
                  ticks: { color: '#a3a3a3' },
                },
              },
            }}
          />
        </div>
      </div>

      {/* 추가 통계 */}
      <div className="mt-8 grid grid-cols-1 md:grid-cols-3 gap-6">
        <div className="bg-neutral-800 rounded-xl p-6 border border-neutral-700">
          <div className="flex items-center gap-3 mb-3">
            <CheckCircle className="w-6 h-6 text-green-500" />
            <h3 className="text-white font-semibold">평균 신뢰 점수</h3>
          </div>
          <p className="text-3xl font-bold text-white">{stats.avgTrustScore}</p>
          <p className="text-neutral-400 text-sm mt-2">5.0 만점</p>
        </div>

        <div className="bg-neutral-800 rounded-xl p-6 border border-neutral-700">
          <div className="flex items-center gap-3 mb-3">
            <TrendingUp className="w-6 h-6 text-blue-500" />
            <h3 className="text-white font-semibold">이번 주 신규 원정대</h3>
          </div>
          <p className="text-3xl font-bold text-white">{stats.newRaidsThisWeek}</p>
          <p className="text-neutral-400 text-sm mt-2">최근 7일</p>
        </div>

        <div className="bg-neutral-800 rounded-xl p-6 border border-neutral-700">
          <div className="flex items-center gap-3 mb-3">
            <Clock className="w-6 h-6 text-yellow-500" />
            <h3 className="text-white font-semibold">평균 응답 시간</h3>
          </div>
          <p className="text-3xl font-bold text-white">2.4시간</p>
          <p className="text-neutral-400 text-sm mt-2">지원 → 수락/거절</p>
        </div>
      </div>
    </div>
  );
};

export default AdminDashboard;
```

### 6.2 사용자 관리 페이지

#### `client/src/pages/admin/UserManagement.tsx`

```typescript
import React, { useEffect, useState } from 'react';
import { Search, Ban, CheckCircle, Shield, Trash2, Edit } from 'lucide-react';

interface User {
  id: string;
  displayName: string;
  email: string;
  photoURL: string;
  role: string;
  isActive: boolean;
  isBanned: boolean;
  banReason: string | null;
  bannedAt: string | null;
  createdAt: string;
  lastLoginAt: string;
  _count: {
    createdRaids: number;
    raidMemberships: number;
    applications: number;
  };
}

const UserManagement: React.FC = () => {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [search, setSearch] = useState('');
  const [filterRole, setFilterRole] = useState('');
  const [filterStatus, setFilterStatus] = useState('');
  const [selectedUser, setSelectedUser] = useState<User | null>(null);
  const [showBanModal, setShowBanModal] = useState(false);
  const [banReason, setBanReason] = useState('');
  const [banDuration, setBanDuration] = useState<'permanent' | number>('permanent');

  useEffect(() => {
    fetchUsers();
  }, [search, filterRole, filterStatus]);

  const fetchUsers = async () => {
    try {
      const params = new URLSearchParams();
      if (search) params.append('search', search);
      if (filterRole) params.append('role', filterRole);
      if (filterStatus === 'active') params.append('isActive', 'true');
      if (filterStatus === 'banned') params.append('isBanned', 'true');

      const response = await fetch(`/api/admin/users?${params}`);
      const data = await response.json();
      setUsers(data.users);
    } catch (error) {
      console.error('Failed to fetch users:', error);
    } finally {
      setLoading(false);
    }
  };

  const handleBanUser = async () => {
    if (!selectedUser || !banReason) return;

    try {
      const response = await fetch(`/api/admin/users/${selectedUser.id}/ban`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ reason: banReason, duration: banDuration }),
      });

      if (response.ok) {
        alert('사용자가 정지되었습니다.');
        setShowBanModal(false);
        fetchUsers();
      }
    } catch (error) {
      console.error('Failed to ban user:', error);
    }
  };

  const handleUnbanUser = async (userId: string) => {
    if (!confirm('이 사용자의 정지를 해제하시겠습니까?')) return;

    try {
      const response = await fetch(`/api/admin/users/${userId}/unban`, {
        method: 'PATCH',
      });

      if (response.ok) {
        alert('정지가 해제되었습니다.');
        fetchUsers();
      }
    } catch (error) {
      console.error('Failed to unban user:', error);
    }
  };

  const getRoleBadgeColor = (role: string) => {
    switch (role) {
      case 'super_admin': return 'bg-red-500';
      case 'admin': return 'bg-orange-500';
      case 'moderator': return 'bg-yellow-500';
      default: return 'bg-neutral-600';
    }
  };

  if (loading) {
    return <div className="text-center py-20 text-white">로딩 중...</div>;
  }

  return (
    <div>
      {/* 헤더 */}
      <div className="mb-8">
        <h1 className="text-3xl font-bold text-white mb-2">사용자 관리</h1>
        <p className="text-neutral-400">전체 사용자를 관리하고 권한을 부여합니다</p>
      </div>

      {/* 검색 및 필터 */}
      <div className="bg-neutral-800 rounded-xl p-6 border border-neutral-700 mb-6">
        <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
          {/* 검색 */}
          <div className="relative">
            <Search className="absolute left-3 top-3 w-5 h-5 text-neutral-400" />
            <input
              type="text"
              placeholder="이름 또는 이메일 검색..."
              value={search}
              onChange={(e) => setSearch(e.target.value)}
              className="w-full pl-10 pr-4 py-2 bg-neutral-700 border border-neutral-600 rounded-lg text-white focus:outline-none focus:border-primary-500"
            />
          </div>

          {/* 역할 필터 */}
          <select
            value={filterRole}
            onChange={(e) => setFilterRole(e.target.value)}
            className="px-4 py-2 bg-neutral-700 border border-neutral-600 rounded-lg text-white focus:outline-none focus:border-primary-500"
          >
            <option value="">전체 역할</option>
            <option value="user">일반 사용자</option>
            <option value="moderator">모더레이터</option>
            <option value="admin">관리자</option>
            <option value="super_admin">최고 관리자</option>
          </select>

          {/* 상태 필터 */}
          <select
            value={filterStatus}
            onChange={(e) => setFilterStatus(e.target.value)}
            className="px-4 py-2 bg-neutral-700 border border-neutral-600 rounded-lg text-white focus:outline-none focus:border-primary-500"
          >
            <option value="">전체 상태</option>
            <option value="active">활성</option>
            <option value="banned">정지</option>
          </select>
        </div>
      </div>

      {/* 사용자 테이블 */}
      <div className="bg-neutral-800 rounded-xl border border-neutral-700 overflow-hidden">
        <table className="w-full">
          <thead className="bg-neutral-700">
            <tr>
              <th className="px-6 py-4 text-left text-sm font-semibold text-neutral-300">사용자</th>
              <th className="px-6 py-4 text-left text-sm font-semibold text-neutral-300">역할</th>
              <th className="px-6 py-4 text-left text-sm font-semibold text-neutral-300">상태</th>
              <th className="px-6 py-4 text-left text-sm font-semibold text-neutral-300">활동</th>
              <th className="px-6 py-4 text-left text-sm font-semibold text-neutral-300">가입일</th>
              <th className="px-6 py-4 text-right text-sm font-semibold text-neutral-300">작업</th>
            </tr>
          </thead>
          <tbody className="divide-y divide-neutral-700">
            {users.map((user) => (
              <tr key={user.id} className="hover:bg-neutral-750">
                <td className="px-6 py-4">
                  <div className="flex items-center gap-3">
                    <img
                      src={user.photoURL || 'https://via.placeholder.com/40'}
                      alt={user.displayName}
                      className="w-10 h-10 rounded-full"
                    />
                    <div>
                      <p className="text-white font-medium">{user.displayName}</p>
                      <p className="text-neutral-400 text-sm">{user.email}</p>
                    </div>
                  </div>
                </td>
                <td className="px-6 py-4">
                  <span className={`px-3 py-1 ${getRoleBadgeColor(user.role)} text-white text-xs font-semibold rounded-full`}>
                    {user.role === 'super_admin' ? '최고 관리자' :
                     user.role === 'admin' ? '관리자' :
                     user.role === 'moderator' ? '모더레이터' : '사용자'}
                  </span>
                </td>
                <td className="px-6 py-4">
                  {user.isBanned ? (
                    <div>
                      <span className="px-3 py-1 bg-red-500 text-white text-xs font-semibold rounded-full">정지</span>
                      {user.banReason && (
                        <p className="text-neutral-400 text-xs mt-1">{user.banReason}</p>
                      )}
                    </div>
                  ) : (
                    <span className="px-3 py-1 bg-green-500 text-white text-xs font-semibold rounded-full">활성</span>
                  )}
                </td>
                <td className="px-6 py-4">
                  <div className="text-sm text-neutral-300">
                    <p>원정대: {user._count.createdRaids}개</p>
                    <p>참여: {user._count.raidMemberships}개</p>
                  </div>
                </td>
                <td className="px-6 py-4">
                  <p className="text-neutral-300 text-sm">
                    {new Date(user.createdAt).toLocaleDateString('ko-KR')}
                  </p>
                </td>
                <td className="px-6 py-4">
                  <div className="flex justify-end gap-2">
                    {user.isBanned ? (
                      <button
                        onClick={() => handleUnbanUser(user.id)}
                        className="p-2 text-green-400 hover:bg-neutral-700 rounded-lg transition-colors"
                        title="정지 해제"
                      >
                        <CheckCircle className="w-5 h-5" />
                      </button>
                    ) : (
                      <button
                        onClick={() => {
                          setSelectedUser(user);
                          setShowBanModal(true);
                        }}
                        className="p-2 text-red-400 hover:bg-neutral-700 rounded-lg transition-colors"
                        title="정지"
                      >
                        <Ban className="w-5 h-5" />
                      </button>
                    )}
                    <button
                      className="p-2 text-blue-400 hover:bg-neutral-700 rounded-lg transition-colors"
                      title="역할 변경"
                    >
                      <Shield className="w-5 h-5" />
                    </button>
                  </div>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>

      {/* 정지 모달 */}
      {showBanModal && selectedUser && (
        <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
          <div className="bg-neutral-800 rounded-xl p-6 w-full max-w-md border border-neutral-700">
            <h3 className="text-xl font-bold text-white mb-4">사용자 정지</h3>
            
            <div className="mb-4">
              <p className="text-neutral-300 mb-2">
                <strong>{selectedUser.displayName}</strong>을(를) 정지하시겠습니까?
              </p>
            </div>

            <div className="mb-4">
              <label className="block text-sm font-medium text-neutral-300 mb-2">
                정지 사유 *
              </label>
              <textarea
                value={banReason}
                onChange={(e) => setBanReason(e.target.value)}
                rows={3}
                className="w-full px-4 py-2 bg-neutral-700 border border-neutral-600 rounded-lg text-white focus:outline-none focus:border-primary-500"
                placeholder="정지 사유를 입력하세요..."
              />
            </div>

            <div className="mb-6">
              <label className="block text-sm font-medium text-neutral-300 mb-2">
                정지 기간
              </label>
              <select
                value={banDuration}
                onChange={(e) => setBanDuration(e.target.value === 'permanent' ? 'permanent' : parseInt(e.target.value))}
                className="w-full px-4 py-2 bg-neutral-700 border border-neutral-600 rounded-lg text-white focus:outline-none focus:border-primary-500"
              >
                <option value="permanent">영구 정지</option>
                <option value="7">7일</option>
                <option value="14">14일</option>
                <option value="30">30일</option>
              </select>
            </div>

            <div className="flex justify-end gap-3">
              <button
                onClick={() => {
                  setShowBanModal(false);
                  setBanReason('');
                }}
                className="px-4 py-2 bg-neutral-700 text-white rounded-lg hover:bg-neutral-600 transition-colors"
              >
                취소
              </button>
              <button
                onClick={handleBanUser}
                disabled={!banReason}
                className="px-4 py-2 bg-red-500 text-white rounded-lg hover:bg-red-600 transition-colors disabled:opacity-50 disabled:cursor-not-allowed"
              >
                정지
              </button>
            </div>
          </div>
        </div>
      )}
    </div>
  );
};

export default UserManagement;
```

---

## 7. 통계 및 분석 대시보드

### 7.1 통계 계산 유틸리티

#### `server/src/utils/statistics.ts`

```typescript
import { prisma } from '../index';

/**
 * 특정 기간의 사용자 증가율 계산
 */
export async function calculateUserGrowthRate(days: number = 30): Promise<number> {
  const startDate = new Date(Date.now() - days * 24 * 60 * 60 * 1000);
  const previousPeriodStart = new Date(startDate.getTime() - days * 24 * 60 * 60 * 1000);

  const [currentPeriod, previousPeriod] = await Promise.all([
    prisma.user.count({
      where: { createdAt: { gte: startDate } }
    }),
    prisma.user.count({
      where: {
        createdAt: {
          gte: previousPeriodStart,
          lt: startDate
        }
      }
    })
  ]);

  if (previousPeriod === 0) return 100;
  return ((currentPeriod - previousPeriod) / previousPeriod) * 100;
}

/**
 * 팀 구성 성공률 분석
 */
export async function analyzeTeamFormationSuccess() {
  const [totalRaids, completedRaids, activeRaids] = await Promise.all([
    prisma.raid.count(),
    prisma.raid.count({ where: { status: 'completed' } }),
    prisma.raid.count({ where: { status: 'active' } }),
  ]);

  return {
    total: totalRaids,
    completed: completedRaids,
    active: activeRaids,
    successRate: totalRaids > 0 ? (completedRaids / totalRaids) * 100 : 0,
  };
}

/**
 * 평균 매칭 시간 계산 (지원 → 수락)
 */
export async function calculateAverageMatchingTime(): Promise<number> {
  const acceptedApplications = await prisma.application.findMany({
    where: {
      status: 'accepted',
      respondedAt: { not: null }
    },
    select: {
      appliedAt: true,
      respondedAt: true,
    }
  });

  if (acceptedApplications.length === 0) return 0;

  const totalTime = acceptedApplications.reduce((sum, app) => {
    if (!app.respondedAt) return sum;
    const diff = new Date(app.respondedAt).getTime() - new Date(app.appliedAt).getTime();
    return sum + diff;
  }, 0);

  // 밀리초를 시간으로 변환
  return totalTime / acceptedApplications.length / (1000 * 60 * 60);
}

/**
 * 인기 있는 역할 분석
 */
export async function analyzePopularRoles() {
  const applications = await prisma.application.groupBy({
    by: ['appliedRole'],
    _count: {
      appliedRole: true
    },
    orderBy: {
      _count: {
        appliedRole: 'desc'
      }
    },
    take: 10
  });

  return applications.map(item => ({
    role: item.appliedRole,
    count: item._count.appliedRole
  }));
}
```

---

## 8. 보안 및 감사 로그

### 8.1 감사 로그 조회 페이지

#### `client/src/pages/admin/AuditLogs.tsx`

```typescript
import React, { useEffect, useState } from 'react';
import { FileText, Filter } from 'lucide-react';

interface AuditLog {
  id: string;
  userId: string;
  user: {
    displayName: string;
    email: string;
  };
  action: string;
  targetType: string | null;
  targetId: string | null;
  details: any;
  ipAddress: string | null;
  createdAt: string;
}

const AuditLogs: React.FC = () => {
  const [logs, setLogs] = useState<AuditLog[]>([]);
  const [loading, setLoading] = useState(true);
  const [filterAction, setFilterAction] = useState('');
  const [page, setPage] = useState(1);
  const [total, setTotal] = useState(0);

  useEffect(() => {
    fetchLogs();
  }, [filterAction, page]);

  const fetchLogs = async () => {
    try {
      const params = new URLSearchParams();
      if (filterAction) params.append('action', filterAction);
      params.append('page', page.toString());
      params.append('limit', '50');

      const response = await fetch(`/api/admin/audit-logs?${params}`);
      const data = await response.json();
      setLogs(data.logs);
      setTotal(data.pagination.total);
    } catch (error) {
      console.error('Failed to fetch audit logs:', error);
    } finally {
      setLoading(false);
    }
  };

  const getActionLabel = (action: string) => {
    const labels: Record<string, string> = {
      'user_banned': '사용자 정지',
      'user_unbanned': '정지 해제',
      'user_role_updated': '역할 변경',
      'user_deleted': '사용자 삭제',
      'competition_created': '대회 생성',
      'competition_updated': '대회 수정',
      'competition_deleted': '대회 삭제',
      'raid_deleted': '원정대 삭제',
      'report_resolved': '신고 처리',
      'report_rejected': '신고 거부',
    };
    return labels[action] || action;
  };

  const getActionColor = (action: string) => {
    if (action.includes('deleted') || action.includes('banned')) return 'text-red-400';
    if (action.includes('created') || action.includes('unbanned')) return 'text-green-400';
    if (action.includes('updated')) return 'text-yellow-400';
    return 'text-blue-400';
  };

  if (loading) {
    return <div className="text-center py-20 text-white">로딩 중...</div>;
  }

  return (
    <div>
      {/* 헤더 */}
      <div className="mb-8">
        <h1 className="text-3xl font-bold text-white mb-2">감사 로그</h1>
        <p className="text-neutral-400">관리자 활동 기록을 조회합니다</p>
      </div>

      {/* 필터 */}
      <div className="bg-neutral-800 rounded-xl p-6 border border-neutral-700 mb-6">
        <div className="flex items-center gap-4">
          <Filter className="w-5 h-5 text-neutral-400" />
          <select
            value={filterAction}
            onChange={(e) => setFilterAction(e.target.value)}
            className="px-4 py-2 bg-neutral-700 border border-neutral-600 rounded-lg text-white focus:outline-none focus:border-primary-500"
          >
            <option value="">전체 활동</option>
            <option value="user_banned">사용자 정지</option>
            <option value="user_unbanned">정지 해제</option>
            <option value="competition_created">대회 생성</option>
            <option value="raid_deleted">원정대 삭제</option>
            <option value="report_resolved">신고 처리</option>
          </select>
          <span className="text-neutral-400 text-sm">
            총 {total.toLocaleString()}건의 로그
          </span>
        </div>
      </div>

      {/* 로그 테이블 */}
      <div className="bg-neutral-800 rounded-xl border border-neutral-700 overflow-hidden">
        <table className="w-full">
          <thead className="bg-neutral-700">
            <tr>
              <th className="px-6 py-4 text-left text-sm font-semibold text-neutral-300">시간</th>
              <th className="px-6 py-4 text-left text-sm font-semibold text-neutral-300">관리자</th>
              <th className="px-6 py-4 text-left text-sm font-semibold text-neutral-300">활동</th>
              <th className="px-6 py-4 text-left text-sm font-semibold text-neutral-300">대상</th>
              <th className="px-6 py-4 text-left text-sm font-semibold text-neutral-300">IP 주소</th>
            </tr>
          </thead>
          <tbody className="divide-y divide-neutral-700">
            {logs.map((log) => (
              <tr key={log.id} className="hover:bg-neutral-750">
                <td className="px-6 py-4">
                  <p className="text-neutral-300 text-sm">
                    {new Date(log.createdAt).toLocaleString('ko-KR')}
                  </p>
                </td>
                <td className="px-6 py-4">
                  <div>
                    <p className="text-white font-medium">{log.user.displayName}</p>
                    <p className="text-neutral-400 text-xs">{log.user.email}</p>
                  </div>
                </td>
                <td className="px-6 py-4">
                  <span className={`font-medium ${getActionColor(log.action)}`}>
                    {getActionLabel(log.action)}
                  </span>
                </td>
                <td className="px-6 py-4">
                  <div className="text-sm text-neutral-300">
                    <p>{log.targetType || '-'}</p>
                    {log.targetId && (
                      <p className="text-neutral-500 text-xs font-mono">{log.targetId}</p>
                    )}
                  </div>
                </td>
                <td className="px-6 py-4">
                  <p className="text-neutral-400 text-sm font-mono">
                    {log.ipAddress || '-'}
                  </p>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>

      {/* 페이지네이션 */}
      <div className="mt-6 flex justify-center gap-2">
        <button
          onClick={() => setPage(page - 1)}
          disabled={page === 1}
          className="px-4 py-2 bg-neutral-700 text-white rounded-lg hover:bg-neutral-600 disabled:opacity-50 disabled:cursor-not-allowed"
        >
          이전
        </button>
        <span className="px-4 py-2 text-white">
          {page} / {Math.ceil(total / 50)}
        </span>
        <button
          onClick={() => setPage(page + 1)}
          disabled={page >= Math.ceil(total / 50)}
          className="px-4 py-2 bg-neutral-700 text-white rounded-lg hover:bg-neutral-600 disabled:opacity-50 disabled:cursor-not-allowed"
        >
          다음
        </button>
      </div>
    </div>
  );
};

export default AuditLogs;
```

---

## 9. 체크리스트

### 9.1 관리자 시스템 구현 체크리스트

#### 백엔드
- [ ] User 모델에 role, isBanned 필드 추가
- [ ] Report 모델 생성
- [ ] AuditLog 모델 생성
- [ ] 권한 체크 미들웨어 구현
- [ ] 관리자 API 라우트 구현
- [ ] 대시보드 통계 API 구현
- [ ] 사용자 관리 API 구현
- [ ] 신고 관리 API 구현
- [ ] 감사 로그 유틸리티 구현

#### 프론트엔드
- [ ] AdminLayout 컴포넌트 생성
- [ ] AdminRoute 권한 체크 구현
- [ ] AdminDashboard 페이지 구현
- [ ] UserManagement 페이지 구현
- [ ] CompetitionManagement 페이지 구현
- [ ] RaidManagement 페이지 구현
- [ ] ReportManagement 페이지 구현
- [ ] AuditLogs 페이지 구현
- [ ] Chart.js 설치 및 차트 구현

#### 보안
- [ ] Super Admin 계정 초기화
- [ ] 모든 관리자 API에 권한 체크 적용
- [ ] IP 주소 및 User Agent 로깅
- [ ] Rate Limiting 강화
- [ ] CSRF 보호 구현

---

## 10. 배포 가이드

### 10.1 Super Admin 초기화

프로덕션 배포 전 최초 Super Admin 계정을 생성해야 합니다.

#### 방법 1: 데이터베이스 직접 수정
```sql
-- Prisma Studio에서 실행
UPDATE "User" 
SET role = 'super_admin' 
WHERE email = 'admin@dacon-raid.com';
```

#### 방법 2: 초기화 스크립트
```typescript
// server/scripts/init-admin.ts
import { prisma } from '../src/index';

async function initSuperAdmin() {
  const email = process.env.SUPER_ADMIN_EMAIL;
  
  if (!email) {
    console.error('SUPER_ADMIN_EMAIL is not set');
    process.exit(1);
  }

  const user = await prisma.user.update({
    where: { email },
    data: { role: 'super_admin' }
  });

  console.log(`Super admin initialized: ${user.email}`);
}

initSuperAdmin();
```

실행:
```bash
cd server
SUPER_ADMIN_EMAIL=admin@dacon-raid.com npm run init-admin
```

---

## 11. 참고 자료

### 11.1 차트 라이브러리 설치

```bash
cd client
npm install chart.js react-chartjs-2
```

### 11.2 추천 도구

- **Prisma Studio**: 데이터베이스 GUI
- **Postman**: API 테스트
- **Sentry**: 에러 모니터링 (프로덕션)

---

## 12. 마무리

이 가이드를 따라 구현하면 다음과 같은 **완전한 관리자 대시보드**를 구축할 수 있습니다:

✅ **통계 대시보드** - 실시간 KPI 및 차트  
✅ **사용자 관리** - 정지, 권한 부여, 삭제  
✅ **대회 관리** - CRUD 및 활성화 관리  
✅ **원정대 관리** - 모니터링 및 삭제  
✅ **신고 관리** - 처리 및 조치  
✅ **감사 로그** - 모든 관리자 활동 기록

---

**문서 버전**: 1.0  
**작성일**: 2025년 11월 1일  
**다음 업데이트**: 사용자 피드백 반영 후
