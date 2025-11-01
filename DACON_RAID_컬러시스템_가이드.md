# 🎨 DACON: RAID - 컬러 시스템 가이드

**버전**: 2.0  
**업데이트**: 2025년 11월 1일  
**테마**: Light Theme (밝은 테마)

---

## 📋 색상 팔레트

### 기본 색상
```css
/* Primary (노란색) - 메인 브랜드 색상 */
--primary: #fcd34d;
--primary-hover: #fbbf24;
--primary-active: #f59e0b;

/* Secondary (파란색) - 활성/강조 색상 */
--secondary: #3b82f6;
--secondary-hover: #2563eb;
--secondary-active: #1d4ed8;

/* Background (밝은 회색) */
--background: #f8f9fa;
--background-secondary: #e9ecef;
--background-card: #ffffff;

/* Text Colors */
--text-primary: #212529;
--text-secondary: #6c757d;
--text-muted: #adb5bd;

/* Border & Shadow */
--border: #000000;
--shadow: rgba(0, 0, 0, 0.1);
--shadow-strong: rgba(0, 0, 0, 0.25);

/* Status Colors */
--success: #10b981;
--warning: #f59e0b;
--error: #ef4444;
--info: #3b82f6;
```

---

## 1. Tailwind CSS 설정

### 1.1 `tailwind.config.js` 업데이트

#### `client/tailwind.config.js`

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
        // Primary (노란색) - 메인 브랜드 색상
        primary: {
          50: '#fffbeb',
          100: '#fef3c7',
          200: '#fde68a',
          300: '#fcd34d',  // 메인 컬러
          400: '#fbbf24',
          500: '#f59e0b',
          600: '#d97706',
          700: '#b45309',
          800: '#92400e',
          900: '#78350f',
        },
        // Secondary (파란색) - 활성/강조 색상
        secondary: {
          50: '#eff6ff',
          100: '#dbeafe',
          200: '#bfdbfe',
          300: '#93c5fd',
          400: '#60a5fa',
          500: '#3b82f6',  // 메인 컬러
          600: '#2563eb',
          700: '#1d4ed8',
          800: '#1e40af',
          900: '#1e3a8a',
        },
        // Background
        background: {
          DEFAULT: '#f8f9fa',
          secondary: '#e9ecef',
          card: '#ffffff',
        },
        // Text
        text: {
          primary: '#212529',
          secondary: '#6c757d',
          muted: '#adb5bd',
        },
        // Border
        border: {
          DEFAULT: '#000000',
          light: '#dee2e6',
        },
        // Status
        success: {
          DEFAULT: '#10b981',
          light: '#d1fae5',
          dark: '#065f46',
        },
        warning: {
          DEFAULT: '#f59e0b',
          light: '#fef3c7',
          dark: '#92400e',
        },
        error: {
          DEFAULT: '#ef4444',
          light: '#fee2e2',
          dark: '#991b1b',
        },
      },
      boxShadow: {
        'DEFAULT': '0 1px 3px 0 rgba(0, 0, 0, 0.1)',
        'md': '0 4px 6px -1px rgba(0, 0, 0, 0.1)',
        'lg': '0 10px 15px -3px rgba(0, 0, 0, 0.1)',
        'xl': '0 20px 25px -5px rgba(0, 0, 0, 0.1)',
        'strong': '0 10px 25px rgba(0, 0, 0, 0.25)',
      },
    },
  },
  plugins: [],
}
```

---

## 2. 전역 스타일

### 2.1 `client/src/index.css`

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  body {
    @apply bg-background text-text-primary;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
      'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
      sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }

  * {
    box-sizing: border-box;
  }
}

@layer components {
  /* 버튼 스타일 */
  .btn-primary {
    @apply bg-primary-300 hover:bg-primary-400 active:bg-primary-500 
           text-text-primary font-semibold py-2 px-4 rounded-lg 
           transition-all duration-200 border border-border
           shadow hover:shadow-md;
  }
  
  .btn-secondary {
    @apply bg-secondary-500 hover:bg-secondary-600 active:bg-secondary-700 
           text-white font-semibold py-2 px-4 rounded-lg 
           transition-all duration-200 border border-border
           shadow hover:shadow-md;
  }
  
  .btn-outline {
    @apply bg-transparent hover:bg-background-secondary 
           text-text-primary font-semibold py-2 px-4 rounded-lg 
           transition-all duration-200 border-2 border-border
           hover:shadow;
  }

  .btn-ghost {
    @apply bg-transparent hover:bg-background-secondary 
           text-text-primary font-semibold py-2 px-4 rounded-lg 
           transition-all duration-200;
  }
  
  /* 카드 스타일 */
  .card {
    @apply bg-background-card rounded-xl p-6 
           border-2 border-border shadow-md;
  }

  .card-hover {
    @apply bg-background-card rounded-xl p-6 
           border-2 border-border shadow-md
           transition-all duration-200
           hover:shadow-lg hover:border-secondary-500;
  }
  
  /* 입력 필드 스타일 */
  .input {
    @apply w-full px-4 py-2 
           bg-background-card border-2 border-border rounded-lg 
           text-text-primary placeholder-text-muted
           focus:outline-none focus:border-secondary-500 focus:ring-2 focus:ring-secondary-500/20
           transition-all duration-200;
  }

  .input-error {
    @apply border-error focus:border-error focus:ring-error/20;
  }
  
  /* 선택 필드 스타일 */
  .select {
    @apply w-full px-4 py-2 
           bg-background-card border-2 border-border rounded-lg 
           text-text-primary
           focus:outline-none focus:border-secondary-500 focus:ring-2 focus:ring-secondary-500/20
           transition-all duration-200;
  }
  
  /* 텍스트 영역 스타일 */
  .textarea {
    @apply w-full px-4 py-2 
           bg-background-card border-2 border-border rounded-lg 
           text-text-primary placeholder-text-muted
           focus:outline-none focus:border-secondary-500 focus:ring-2 focus:ring-secondary-500/20
           transition-all duration-200
           resize-vertical;
  }

  /* 뱃지 스타일 */
  .badge {
    @apply inline-block px-3 py-1 text-sm font-semibold rounded-full;
  }

  .badge-primary {
    @apply bg-primary-300 text-text-primary border border-border;
  }

  .badge-secondary {
    @apply bg-secondary-500 text-white;
  }

  .badge-success {
    @apply bg-success text-white;
  }

  .badge-warning {
    @apply bg-warning text-white;
  }

  .badge-error {
    @apply bg-error text-white;
  }

  /* 링크 스타일 */
  .link {
    @apply text-secondary-500 hover:text-secondary-600 
           underline transition-colors duration-200;
  }

  /* 구분선 */
  .divider {
    @apply border-t-2 border-border my-6;
  }

  /* 스크롤바 커스터마이징 */
  .scrollbar::-webkit-scrollbar {
    width: 8px;
    height: 8px;
  }

  .scrollbar::-webkit-scrollbar-track {
    @apply bg-background-secondary rounded-full;
  }

  .scrollbar::-webkit-scrollbar-thumb {
    @apply bg-text-muted hover:bg-text-secondary rounded-full;
  }
}

@layer utilities {
  /* 텍스트 유틸리티 */
  .text-balance {
    text-wrap: balance;
  }

  /* 그라데이션 */
  .gradient-primary {
    background: linear-gradient(135deg, #fcd34d 0%, #f59e0b 100%);
  }

  .gradient-secondary {
    background: linear-gradient(135deg, #3b82f6 0%, #1d4ed8 100%);
  }

  /* 애니메이션 */
  .animate-fade-in {
    animation: fadeIn 0.3s ease-in;
  }

  @keyframes fadeIn {
    from {
      opacity: 0;
      transform: translateY(-10px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  .animate-slide-up {
    animation: slideUp 0.3s ease-out;
  }

  @keyframes slideUp {
    from {
      opacity: 0;
      transform: translateY(20px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }
}
```

---

## 3. 컴포넌트별 색상 변경

### 3.1 Header 컴포넌트

#### `client/src/components/Header.tsx`

```typescript
import React from 'react';
import { Link } from 'react-router-dom';
import { Shield, Plus, User, LogOut } from 'lucide-react';
import { useAuth } from '../contexts/AuthContext';

const Header: React.FC = () => {
  const { user, logout } = useAuth();

  return (
    <header className="bg-background-card border-b-2 border-border shadow-md">
      <div className="container mx-auto px-4">
        <div className="flex items-center justify-between h-16">
          {/* 로고 */}
          <Link to="/" className="flex items-center gap-2 hover:opacity-80 transition-opacity">
            <Shield className="w-8 h-8 text-primary-300" />
            <span className="text-2xl font-bold gradient-primary bg-clip-text text-transparent">
              DACON: RAID
            </span>
          </Link>

          {/* 네비게이션 */}
          <nav className="hidden md:flex items-center gap-6">
            <Link 
              to="/" 
              className="text-text-primary hover:text-secondary-500 font-medium transition-colors"
            >
              원정대 찾기
            </Link>
            <Link 
              to="/dashboard" 
              className="text-text-primary hover:text-secondary-500 font-medium transition-colors"
            >
              내 원정대
            </Link>
          </nav>

          {/* 사용자 메뉴 */}
          <div className="flex items-center gap-3">
            {user ? (
              <>
                <Link to="/create-raid" className="btn-primary">
                  <Plus className="w-5 h-5 inline mr-2" />
                  원정대 만들기
                </Link>
                <Link 
                  to={`/profile/${user.id}`}
                  className="flex items-center gap-2 px-4 py-2 hover:bg-background-secondary rounded-lg transition-colors"
                >
                  <img 
                    src={user.photoURL || 'https://via.placeholder.com/32'} 
                    alt={user.displayName}
                    className="w-8 h-8 rounded-full border-2 border-border"
                  />
                  <span className="text-text-primary font-medium">{user.displayName}</span>
                </Link>
                {['moderator', 'admin', 'super_admin'].includes(user.role) && (
                  <Link 
                    to="/admin" 
                    className="btn-secondary"
                  >
                    관리자
                  </Link>
                )}
                <button
                  onClick={logout}
                  className="p-2 text-text-secondary hover:text-error hover:bg-background-secondary rounded-lg transition-colors"
                  title="로그아웃"
                >
                  <LogOut className="w-5 h-5" />
                </button>
              </>
            ) : (
              <button className="btn-primary">
                로그인
              </button>
            )}
          </div>
        </div>
      </div>
    </header>
  );
};

export default Header;
```

### 3.2 RaidCard 컴포넌트

#### `client/src/components/RaidCard.tsx`

```typescript
import React from 'react';
import { Link } from 'react-router-dom';
import { Users, Clock, Target, Calendar } from 'lucide-react';
import { Raid } from '../types';

interface RaidCardProps {
  raid: Raid;
}

const RaidCard: React.FC<RaidCardProps> = ({ raid }) => {
  const totalSlots = raid.slots.reduce((sum, slot) => sum + slot.count, 0);
  const filledSlots = raid.slots.reduce((sum, slot) => sum + slot.filled, 0);
  const progress = (filledSlots / totalSlots) * 100;

  const getGoalColor = (goal: string) => {
    switch (goal) {
      case '학습 중심': return 'badge-success';
      case '상위권 목표': return 'badge-warning';
      case '프로덕트 완성': return 'badge-secondary';
      default: return 'badge-primary';
    }
  };

  return (
    <Link to={`/raid/${raid.id}`} className="block">
      <div className="card-hover animate-fade-in">
        {/* 헤더 */}
        <div className="flex items-start justify-between mb-4">
          <div className="flex-1">
            <h3 className="text-xl font-bold text-text-primary mb-2 hover:text-secondary-500 transition-colors">
              {raid.name}
            </h3>
            <p className="text-text-secondary text-sm line-clamp-2">
              {raid.description}
            </p>
          </div>
          <span className={`badge ${getGoalColor(raid.goal)} ml-4`}>
            {raid.goal}
          </span>
        </div>

        {/* 대회 정보 */}
        {raid.competition && (
          <div className="flex items-center gap-2 mb-4 px-3 py-2 bg-background-secondary rounded-lg border border-border-light">
            <Calendar className="w-4 h-4 text-secondary-500" />
            <span className="text-sm font-medium text-text-primary">
              {raid.competition.name}
            </span>
            <span className="text-xs px-2 py-1 bg-background-card rounded text-text-secondary border border-border-light">
              {raid.competition.type}
            </span>
          </div>
        )}

        {/* 역할 슬롯 */}
        <div className="mb-4">
          <div className="flex flex-wrap gap-2">
            {raid.slots.map((slot) => (
              <div 
                key={slot.slotId}
                className="px-3 py-1 bg-background-card rounded-lg border-2 border-border text-sm"
              >
                <span className="font-semibold text-text-primary">{slot.role}</span>
                <span className="text-text-muted mx-1">·</span>
                <span className={`font-medium ${slot.filled >= slot.count ? 'text-error' : 'text-success'}`}>
                  {slot.filled}/{slot.count}
                </span>
              </div>
            ))}
          </div>
        </div>

        {/* 진행률 바 */}
        <div className="mb-4">
          <div className="flex justify-between text-sm mb-2">
            <span className="text-text-secondary">팀 구성</span>
            <span className="font-semibold text-text-primary">
              {filledSlots}/{totalSlots} 명
            </span>
          </div>
          <div className="w-full bg-background-secondary rounded-full h-2 border border-border-light">
            <div
              className="h-2 rounded-full transition-all duration-300"
              style={{
                width: `${progress}%`,
                background: progress === 100 
                  ? 'linear-gradient(90deg, #10b981 0%, #059669 100%)'
                  : 'linear-gradient(90deg, #fcd34d 0%, #f59e0b 100%)'
              }}
            />
          </div>
        </div>

        {/* 하단 정보 */}
        <div className="flex items-center justify-between pt-4 border-t-2 border-border-light">
          <div className="flex items-center gap-4 text-sm text-text-secondary">
            <div className="flex items-center gap-1">
              <Clock className="w-4 h-4" />
              <span>{raid.expectedHoursPerWeek}</span>
            </div>
            <div className="flex items-center gap-1">
              <Users className="w-4 h-4" />
              <span>{raid.collaborationMethod}</span>
            </div>
          </div>
          
          {raid.creator && (
            <div className="flex items-center gap-2">
              <img
                src={raid.creator.photoURL || 'https://via.placeholder.com/24'}
                alt={raid.creator.displayName}
                className="w-6 h-6 rounded-full border border-border"
              />
              <span className="text-sm font-medium text-text-primary">
                {raid.creator.displayName}
              </span>
            </div>
          )}
        </div>
      </div>
    </Link>
  );
};

export default RaidCard;
```

### 3.3 기본 UI 컴포넌트

#### `client/src/components/ui/Button.tsx`

```typescript
import React from 'react';
import { LucideIcon } from 'lucide-react';

interface ButtonProps {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  icon?: LucideIcon;
  disabled?: boolean;
  onClick?: () => void;
  type?: 'button' | 'submit' | 'reset';
  className?: string;
}

const Button: React.FC<ButtonProps> = ({
  children,
  variant = 'primary',
  size = 'md',
  icon: Icon,
  disabled = false,
  onClick,
  type = 'button',
  className = '',
}) => {
  const baseClasses = 'font-semibold rounded-lg transition-all duration-200 inline-flex items-center justify-center gap-2';
  
  const variantClasses = {
    primary: 'btn-primary',
    secondary: 'btn-secondary',
    outline: 'btn-outline',
    ghost: 'btn-ghost',
  };

  const sizeClasses = {
    sm: 'text-sm py-1.5 px-3',
    md: 'text-base py-2 px-4',
    lg: 'text-lg py-3 px-6',
  };

  const disabledClasses = disabled 
    ? 'opacity-50 cursor-not-allowed' 
    : 'cursor-pointer';

  return (
    <button
      type={type}
      onClick={onClick}
      disabled={disabled}
      className={`${baseClasses} ${variantClasses[variant]} ${sizeClasses[size]} ${disabledClasses} ${className}`}
    >
      {Icon && <Icon className="w-5 h-5" />}
      {children}
    </button>
  );
};

export default Button;
```

#### `client/src/components/ui/Card.tsx`

```typescript
import React from 'react';

interface CardProps {
  children: React.ReactNode;
  hover?: boolean;
  className?: string;
  onClick?: () => void;
}

const Card: React.FC<CardProps> = ({ 
  children, 
  hover = false, 
  className = '',
  onClick 
}) => {
  const baseClass = hover ? 'card-hover' : 'card';
  const clickableClass = onClick ? 'cursor-pointer' : '';

  return (
    <div 
      className={`${baseClass} ${clickableClass} ${className}`}
      onClick={onClick}
    >
      {children}
    </div>
  );
};

export default Card;
```

#### `client/src/components/ui/Badge.tsx`

```typescript
import React from 'react';

interface BadgeProps {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary' | 'success' | 'warning' | 'error';
  className?: string;
}

const Badge: React.FC<BadgeProps> = ({ 
  children, 
  variant = 'primary',
  className = '' 
}) => {
  const variantClasses = {
    primary: 'badge-primary',
    secondary: 'badge-secondary',
    success: 'badge-success',
    warning: 'badge-warning',
    error: 'badge-error',
  };

  return (
    <span className={`badge ${variantClasses[variant]} ${className}`}>
      {children}
    </span>
  );
};

export default Badge;
```

---

## 4. 관리자 페이지 색상

### 4.1 AdminLayout

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
  LogOut,
  Home
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
    <div className="min-h-screen bg-background flex">
      {/* 사이드바 */}
      <aside className="w-64 bg-background-card border-r-2 border-border shadow-lg">
        {/* 로고 */}
        <div className="p-6 border-b-2 border-border">
          <h1 className="text-xl font-bold gradient-primary bg-clip-text text-transparent">
            RAID 관리자
          </h1>
          <p className="text-sm text-text-secondary mt-1">{user?.displayName}</p>
          <span className="inline-block mt-2 px-3 py-1 text-xs font-semibold bg-primary-300 text-text-primary rounded-full border border-border">
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
                    className={`flex items-center gap-3 px-4 py-3 rounded-lg transition-all ${
                      isActive
                        ? 'bg-secondary-500 text-white shadow-md'
                        : 'text-text-primary hover:bg-background-secondary border border-transparent hover:border-border'
                    }`}
                  >
                    <Icon className="w-5 h-5" />
                    <span className="font-medium">{item.label}</span>
                  </Link>
                </li>
              );
            })}
          </ul>
        </nav>

        {/* 하단 버튼 */}
        <div className="absolute bottom-0 w-64 p-4 border-t-2 border-border bg-background-card">
          <Link
            to="/"
            className="flex items-center gap-3 px-4 py-3 rounded-lg text-text-primary hover:bg-background-secondary border border-border hover:border-secondary-500 transition-all mb-2"
          >
            <Home className="w-5 h-5" />
            <span className="font-medium">사용자 페이지로</span>
          </Link>
          <button
            onClick={logout}
            className="w-full flex items-center gap-3 px-4 py-3 rounded-lg text-error hover:bg-error/10 border border-error transition-all"
          >
            <LogOut className="w-5 h-5" />
            <span className="font-medium">로그아웃</span>
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

## 5. 색상 사용 가이드라인

### 5.1 버튼 사용

```typescript
// Primary 버튼 - 주요 액션 (원정대 만들기, 제출 등)
<button className="btn-primary">원정대 만들기</button>

// Secondary 버튼 - 보조 액션 (활성화, 강조)
<button className="btn-secondary">지원하기</button>

// Outline 버튼 - 덜 중요한 액션
<button className="btn-outline">취소</button>

// Ghost 버튼 - 최소한의 강조
<button className="btn-ghost">더보기</button>
```

### 5.2 상태 색상

```typescript
// 성공 - 완료, 수락
<span className="text-success">수락됨</span>

// 경고 - 주의 필요
<span className="text-warning">대기 중</span>

// 에러 - 거부, 삭제
<span className="text-error">거부됨</span>

// 정보 - 일반 정보
<span className="text-secondary-500">안내</span>
```

### 5.3 배경 색상

```typescript
// 메인 배경
<div className="bg-background">...</div>

// 보조 배경
<div className="bg-background-secondary">...</div>

// 카드 배경
<div className="bg-background-card">...</div>
```

---

## 6. 적용 체크리스트

### 6.1 설정 파일
- [ ] `tailwind.config.js` 업데이트
- [ ] `index.css` 전역 스타일 적용
- [ ] `postcss.config.js` 확인

### 6.2 공통 컴포넌트
- [ ] Header 색상 변경
- [ ] Footer 색상 변경 (있는 경우)
- [ ] RaidCard 색상 변경
- [ ] Button 컴포넌트 업데이트
- [ ] Card 컴포넌트 업데이트
- [ ] Badge 컴포넌트 업데이트

### 6.3 페이지
- [ ] HomePage 색상 적용
- [ ] RaidDetailPage 색상 적용
- [ ] ProfilePage 색상 적용
- [ ] CreateRaidPage 색상 적용
- [ ] DashboardPage 색상 적용

### 6.4 관리자 페이지
- [ ] AdminLayout 색상 변경
- [ ] AdminDashboard 색상 변경
- [ ] UserManagement 색상 변경
- [ ] 기타 관리 페이지 색상 변경

---

## 7. 빠른 적용 방법

### Step 1: Tailwind 설정 교체
```bash
# client 폴더에서
cp tailwind.config.js tailwind.config.js.backup
# 위의 새 설정으로 교체
```

### Step 2: 전역 스타일 교체
```bash
# client/src/index.css 교체
# 위의 새 스타일로 교체
```

### Step 3: 개발 서버 재시작
```bash
npm run dev
```

### Step 4: 브라우저에서 확인
- 모든 페이지를 순회하며 색상 확인
- 버튼 호버/액티브 상태 확인
- 카드 호버 효과 확인

---

## 8. 색상 미리보기

```html
<!-- 색상 팔레트 미리보기 페이지 -->
<!-- client/src/pages/ColorPreview.tsx 생성 -->

<div className="container mx-auto p-8">
  <h1 className="text-3xl font-bold mb-8">DACON: RAID 컬러 시스템</h1>
  
  <!-- Primary Colors -->
  <div className="mb-8">
    <h2 className="text-xl font-semibold mb-4">Primary (노란색)</h2>
    <div className="flex gap-4">
      <div className="w-20 h-20 bg-primary-300 border-2 border-border rounded-lg shadow"></div>
      <div className="w-20 h-20 bg-primary-400 border-2 border-border rounded-lg shadow"></div>
      <div className="w-20 h-20 bg-primary-500 border-2 border-border rounded-lg shadow"></div>
    </div>
  </div>
  
  <!-- Secondary Colors -->
  <div className="mb-8">
    <h2 className="text-xl font-semibold mb-4">Secondary (파란색)</h2>
    <div className="flex gap-4">
      <div className="w-20 h-20 bg-secondary-400 border-2 border-border rounded-lg shadow"></div>
      <div className="w-20 h-20 bg-secondary-500 border-2 border-border rounded-lg shadow"></div>
      <div className="w-20 h-20 bg-secondary-600 border-2 border-border rounded-lg shadow"></div>
    </div>
  </div>
  
  <!-- Buttons -->
  <div className="mb-8">
    <h2 className="text-xl font-semibold mb-4">Buttons</h2>
    <div className="flex gap-4">
      <button className="btn-primary">Primary</button>
      <button className="btn-secondary">Secondary</button>
      <button className="btn-outline">Outline</button>
      <button className="btn-ghost">Ghost</button>
    </div>
  </div>
</div>
```

---

**문서 버전**: 2.0  
**업데이트**: 밝은 테마로 전환  
**다음 검토**: 사용자 피드백 후
