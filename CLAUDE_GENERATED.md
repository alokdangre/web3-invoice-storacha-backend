# Claude Generated Code

**Task**: Objective: Help design and implement new features based on the current roadmap or project needs. Focus on clean, scalable, and testable code  Claude Tasks:  🧩 Feature Implementation Read the existing codebase and integrate new feature modules where appropriate.  Ensure new code follows existing patterns and conventions.  🗂️ Example Features (replace/add specifics as needed): Add user role-based access control  Implement export to PDF  Add multi-language support  Enable dark mode toggle  🧪 Testing & Documentation Add unit/integration tests for new functionality.  Update README or in-code comments if required.

**Generated on**: Wed Jul 30 20:04:01 UTC 2025

## Generated Content

```
Looking at this Ionic React TypeScript codebase for government billing/invoicing, I'll implement the requested features. Based on the existing structure, I can see this is a spreadsheet-based invoicing application using SocialCalc. Let me implement user role-based access control, PDF export, multi-language support, and dark mode toggle.

## FILES_TO_CREATE

### 1. User Management and Authentication
- `src/types/User.ts`
- `src/contexts/AuthContext.tsx`
- `src/contexts/ThemeContext.tsx`
- `src/contexts/LanguageContext.tsx`
- `src/hooks/useAuth.ts`
- `src/hooks/useTheme.ts`
- `src/hooks/useLanguage.ts`
- `src/services/AuthService.ts`
- `src/services/PdfExportService.ts`
- `src/components/LoginForm/LoginForm.tsx`
- `src/components/UserManagement/UserManagement.tsx`
- `src/components/LanguageSelector/LanguageSelector.tsx`
- `src/components/ThemeToggle/ThemeToggle.tsx`
- `src/utils/rbac.ts`
- `src/locales/en.json`
- `src/locales/es.json`
- `src/locales/fr.json`

### 2. Enhanced Components
- `src/components/ProtectedRoute/ProtectedRoute.tsx`
- `src/components/ExportMenu/ExportMenu.tsx`

## FILES_TO_MODIFY

- `src/App/App.js` (convert to TypeScript and add new features)
- `src/index.js` (convert to TypeScript and add providers)
- `package.json` (add new dependencies)

## CODE_CHANGES

### FILES_TO_CREATE

**src/types/User.ts**
```typescript
export enum UserRole {
  ADMIN = 'admin',
  MANAGER = 'manager',
  USER = 'user',
  VIEWER = 'viewer'
}

export interface User {
  id: string;
  username: string;
  email: string;
  role: UserRole;
  permissions: Permission[];
  createdAt: string;
  lastLogin?: string;
  isActive: boolean;
}

export interface Permission {
  resource: string;
  actions: string[];
}

export interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
}

export interface LoginCredentials {
  username: string;
  password: string;
}
```

**src/contexts/AuthContext.tsx**
```typescript
import React, { createContext, useContext, useReducer, useEffect } from 'react';
import { User, AuthState, LoginCredentials } from '../types/User';
import { AuthService } from '../services/AuthService';

interface AuthContextType extends AuthState {
  login: (credentials: LoginCredentials) => Promise<boolean>;
  logout: () => void;
  hasPermission: (resource: string, action: string) => boolean;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

type AuthAction = 
  | { type: 'LOGIN_START' }
  | { type: 'LOGIN_SUCCESS'; payload: User }
  | { type: 'LOGIN_FAILURE'; payload: string }
  | { type: 'LOGOUT' }
  | { type: 'SET_LOADING'; payload: boolean };

const initialState: AuthState = {
  user: null,
  isAuthenticated: false,
  isLoading: true,
  error: null
};

function authReducer(state: AuthState, action: AuthAction): AuthState {
  switch (action.type) {
    case 'LOGIN_START':
      return { ...state, isLoading: true, error: null };
    case 'LOGIN_SUCCESS':
      return { 
        ...state, 
        user: action.payload, 
        isAuthenticated: true, 
        isLoading: false, 
        error: null 
      };
    case 'LOGIN_FAILURE':
      return { 
        ...state, 
        user: null, 
        isAuthenticated: false, 
        isLoading: false, 
        error: action.payload 
      };
    case 'LOGOUT':
      return { 
        ...state, 
        user: null, 
        isAuthenticated: false, 
        isLoading: false, 
        error: null 
      };
    case 'SET_LOADING':
      return { ...state, isLoading: action.payload };
    default:
      return state;
  }
}

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [state, dispatch] = useReducer(authReducer, initialState);

  useEffect(() => {
    const initAuth = async () => {
      try {
        const user = await AuthService.getCurrentUser();
        if (user) {
          dispatch({ type: 'LOGIN_SUCCESS', payload: user });
        }
      } catch (error) {
        console.error('Auth initialization error:', error);
      } finally {
        dispatch({ type: 'SET_LOADING', payload: false });
      }
    };

    initAuth();
  }, []);

  const login = async (credentials: LoginCredentials): Promise<boolean> => {
    dispatch({ type: 'LOGIN_START' });
    try {
      const user = await AuthService.login(credentials);
      dispatch({ type: 'LOGIN_SUCCESS', payload: user });
      return true;
    } catch (error) {
      dispatch({ type: 'LOGIN_FAILURE', payload: error.message || 'Login failed' });
      return false;
    }
  };

  const logout = async () => {
    try {
      await AuthService.logout();
      dispatch({ type: 'LOGOUT' });
    } catch (error) {
      console.error('Logout error:', error);
      dispatch({ type: 'LOGOUT' });
    }
  };

  const hasPermission = (resource: string, action: string): boolean => {
    if (!state.user) return false;
    
    return state.user.permissions.some(permission => 
      permission.resource === resource && permission.actions.includes(action)
    );
  };

  const value: AuthContextType = {
    ...state,
    login,
    logout,
    hasPermission
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

**src/contexts/ThemeContext.tsx**
```typescript
import React, { createContext, useContext, useState, useEffect } from 'react';

type Theme = 'light' | 'dark';

interface ThemeContextType {
  theme: Theme;
  toggleTheme: () => void;
  setTheme: (theme: Theme) => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [theme, setThemeState] = useState<Theme>(() => {
    const savedTheme = localStorage.getItem('theme') as Theme;
    return savedTheme || 'light';
  });

  useEffect(() => {
    document.documentElement.classList.remove('light', 'dark');
    document.documentElement.classList.add(theme);
    localStorage.setItem('theme', theme);
  }, [theme]);

  const toggleTheme = () => {
    setThemeState(prev => prev === 'light' ? 'dark' : 'light');
  };

  const setTheme = (newTheme: Theme) => {
    setThemeState(newTheme);
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};
```

**src/contexts/LanguageContext.tsx**
```typescript
import React, { createContext, useContext, useState, useEffect } from 'react';

type Language = 'en' | 'es' | 'fr';

interface Translations {
  [key: string]: string | Translations;
}

interface LanguageContextType {
  language: Language;
  setLanguage: (language: Language) => void;
  t: (key: string) => string;
  translations: Translations;
}

const LanguageContext = createContext<LanguageContextType | undefined>(undefined);

const loadTranslations = async (lang: Language): Promise<Translations> => {
  try {
    const translations = await import(`../locales/${lang}.json`);
    return translations.default;
  } catch (error) {
    console.error(`Failed to load translations for ${lang}`, error);
    const fallback = await import('../locales/en.json');
    return fallback.default;
  }
};

export const LanguageProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [language, setLanguageState] = useState<Language>(() => {
    const saved = localStorage.getItem('language') as Language;
    return saved || 'en';
  });
  
  const [translations, setTranslations] = useState<Translations>({});

  useEffect(() => {
    const loadLang = async () => {
      const trans = await loadTranslations(language);
      setTranslations(trans);
    };
    
    loadLang();
    localStorage.setItem('language', language);
  }, [language]);

  const setLanguage = (newLanguage: Language) => {
    setLanguageState(newLanguage);
  };

  const t = (key: string): string => {
    const keys = key.split('.');
    let value: any = translations;
    
    for (const k of keys) {
      value = value?.[k];
      if (value === undefined) break;
    }
    
    return typeof value === 'string' ? value : key;
  };

  return (
    <LanguageContext.Provider value={{ language, setLanguage, t, translations }}>
      {children}
    </LanguageContext.Provider>
  );
};

export const useLanguage = () => {
  const context = useContext(LanguageContext);
  if (context === undefined) {
    throw new Error('useLanguage must be used within a LanguageProvider');
  }
  return context;
};
```

**src/services/AuthService.ts**
```typescript
import { User, LoginCredentials, UserRole } from '../types/User';

// Mock users for development - replace with real API calls
const MOCK_USERS: User[] = [
  {
    id: '1',
    username: 'admin',
    email: 'admin@gov.agency',
    role: UserRole.ADMIN,
    permissions: [
      { resource: 'invoices', actions: ['create', 'read', 'update', 'delete', 'export'] },
      { resource: 'users', actions: ['create', 'read', 'update', 'delete'] },
      { resource: 'settings', actions: ['read', 'update'] }
    ],
    createdAt: '2024-01-01T00:00:00Z',
    isActive: true
  },
  {
    id: '2',
    username: 'manager',
    email: 'manager@gov.agency',
    role: UserRole.MANAGER,
    permissions: [
      { resource: 'invoices', actions: ['create', 'read', 'update', 'export'] },
      { resource: 'users', actions: ['read'] }
    ],
    createdAt: '2024-01-01T00:00:00Z',
    isActive: true
  },
  {
    id: '3',
    username: 'user',
    email: 'user@gov.agency',
    role: UserRole.USER,
    permissions: [
      { resource: 'invoices', actions: ['create', 'read', 'update'] }
    ],
    createdAt: '2024-01-01T00:00:00Z',
    isActive: true
  },
  {
    id: '4',
    username: 'viewer',
    email: 'viewer@gov.agency',
    role: UserRole.VIEWER,
    permissions: [
      { resource: 'invoices', actions: ['read'] }
    ],
    createdAt: '2024-01-01T00:00:00Z',
    isActive: true
  }
];

export class AuthService {
  private static readonly TOKEN_KEY = 'auth_token';
  private static readonly USER_KEY = 'current_user';

  static async login(credentials: LoginCredentials): Promise<User> {
    // Simulate API call delay
    await new Promise(resolve => setTimeout(resolve, 1000));

    // Mock authentication - replace with real API call
    const user = MOCK_USERS.find(u => u.username === credentials.username);
    
    if (!user) {
      throw new Error('Invalid username or password');
    }

    // Mock password validation (in real app, this would be done server-side)
    if (credentials.password !== 'password123') {
      throw new Error('Invalid username or password');
    }

    // Store auth token and user info
    const token = btoa(JSON.stringify({ userId: user.id, timestamp: Date.now() }));
    localStorage.setItem(this.TOKEN_KEY, token);
    localStorage.setItem(this.USER_KEY, JSON.stringify(user));

    const updatedUser = { ...user, lastLogin: new Date().toISOString() };
    localStorage.setItem(this.USER_KEY, JSON.stringify(updatedUser));

    return updatedUser;
  }

  static async logout(): Promise<void> {
    localStorage.removeItem(this.TOKEN_KEY);
    localStorage.removeItem(this.USER_KEY);
  }

  static async getCurrentUser(): Promise<User | null> {
    const token = localStorage.getItem(this.TOKEN_KEY);
    const userStr = localStorage.getItem(this.USER_KEY);

    if (!token || !userStr) {
      return null;
    }

    try {
      const user = JSON.parse(userStr);
      return user;
    } catch (error) {
      console.error('Error parsing user data:', error);
      await this.logout();
      return null;
    }
  }

  static async getAllUsers(): Promise<User[]> {
    // In a real app, this would be an API call with proper authorization
    return MOCK_USERS;
  }

  static async createUser(userData: Partial<User>): Promise<User> {
    // Mock user creation - replace with real API call
    const newUser: User = {
      id: Date.now().toString(),
      username: userData.username!,
      email: userData.email!,
      role: userData.role || UserRole.USER,
      permissions: userData.permissions || [],
      createdAt: new Date().toISOString(),
      isActive: true
    };

    MOCK_USERS.push(newUser);
    return newUser;
  }

  static async updateUser(userId: string, updates: Partial<User>): Promise<User> {
    const index = MOCK_USERS.findIndex(u => u.id === userId);
    if (index === -1) {
      throw new Error('User not found');
    }

    MOCK_USERS[index] = { ...MOCK_USERS[index], ...updates };
    return MOCK_USERS[index];
  }

  static async deleteUser(userId: string): Promise<void> {
    const index = MOCK_USERS.findIndex(u => u.id === userId);
    if (index === -1) {
      throw new Error('User not found');
    }

    MOCK_USERS.splice(index, 1);
  }
}
```

**src/services/PdfExportService.ts**
```typescript
import jsPDF from 'jspdf';
import 'jspdf-autotable';

export interface ExportOptions {
  title?: string;
  author?: string;
  subject?: string;
  includeTimestamp?: boolean;
  format?: 'A4' | 'Letter';
  orientation?: 'portrait' | 'landscape';
}

export class PdfExportService {
  static async exportInvoiceToPdf(
    invoiceData: any,
    options: ExportOptions = {}
  ): Promise<void> {
    const {
      title = 'Invoice',
      author = 'Government Invoice System',
      subject = 'Government Invoice',
      includeTimestamp = true,
      format = 'A4',
      orientation = 'portrait'
    } = options;

    const doc = new jsPDF({
      orientation,
      unit: 'mm',
      format: format.toLowerCase()
    });

    // Set document properties
    doc.setProperties({
      title,
      author,
      subject,
      creator: 'Invoice Suite'
    });

    // Add header
    doc.setFontSize(20);
    doc.setFont('helvetica', 'bold');
    doc.text(title, 20, 25);

    if (includeTimestamp) {
      doc.setFontSize(10);
      doc.setFont('helvetica', 'normal');
      doc.text(`Generated: ${new Date().toLocaleString()}`, 20, 35);
    }

    // Extract invoice data from spreadsheet
    const invoiceInfo = this.extractInvoiceData(invoiceData);
    
    // Add invoice details
    let yPosition = includeTimestamp ? 50 : 40;
    
    // Invoice header info
    if (invoiceInfo.invoiceNumber) {
      doc.setFontSize(12);
      doc.setFont('helvetica', 'bold');
      doc.text(`Invoice #: ${invoiceInfo.invoiceNumber}`, 20, yPosition);
      yPosition += 10;
    }

    if (invoiceInfo.date) {
      doc.text(`Date: ${invoiceInfo.date}`, 20, yPosition);
      yPosition += 10;
    }

    // Bill To / From sections
    yPosition += 10;
    if (invoiceInfo.billTo) {
      doc.text('Bill To:', 20, yPosition);
      yPosition += 8;
      doc.setFont('helvetica', 'normal');
      const billToLines = invoiceInfo.billTo.split('\n');
      billToLines.forEach(line => {
        if (line.trim()) {
          doc.text(line, 25, yPosition);
          yPosition += 6;
        }
      });
    }

    if (invoiceInfo.from) {
      doc.setFont('helvetica', 'bold');
      doc.text('From:', 120, yPosition - (billToLines?.length || 0) * 6);
      let fromY = yPosition - (billToLines?.length || 0) * 6 + 8;
      doc.setFont('helvetica', 'normal');
      const fromLines = invoiceInfo.from.split('\n');
      fromLines.forEach(line => {
        if (line.trim()) {
          doc.text(line, 125, fromY);
          fromY += 6;
        }
      });
    }

    yPosition += 15;

    // Items table
    if (invoiceInfo.items && invoiceInfo.items.length > 0) {
      const tableColumns = ['Description', 'Quantity', 'Rate', 'Amount'];
      const tableRows = invoiceInfo.items.map(item => [
        item.description || '',
        item.quantity?.toString() || '',
        item.rate ? `$${item.rate.toFixed(2)}` : '',
        item.amount ? `$${item.amount.toFixed(2)}` : ''
      ]);

      (doc as any).autoTable({
        startY: yPosition,
        head: [tableColumns],
        body: tableRows,
        theme: 'striped',
        headStyles: {
          fillColor: [41, 128, 185],
          textColor: 255,
          fontStyle: 'bold'
        },
        styles: {
          fontSize: 10,
          cellPadding: 5
        },
        columnStyles: {
          1: { halign: 'center' },
          2: { halign: 'right' },
          3: { halign: 'right' }
        }
      });

      yPosition = (doc as any).lastAutoTable.finalY + 15;
    }

    // Total section
    if (invoiceInfo.total !== undefined) {
      doc.setFontSize(14);
      doc.setFont('helvetica', 'bold');
      doc.text(`Total: $${invoiceInfo.total.toFixed(2)}`, 150, yPosition);
    }

    // Add footer
    const pageCount = doc.getNumberOfPages();
    for (let i = 1; i <= pageCount; i++) {
      doc.setPage(i);
      doc.setFontSize(8);
      doc.setFont('helvetica', 'normal');
      doc.text(
        `Page ${i} of ${pageCount}`,
        doc.internal.pageSize.width - 30,
        doc.internal.pageSize.height - 10
      );
    }

    // Save the PDF
    const filename = `${title.toLowerCase().replace(/\s+/g, '-')}-${Date.now()}.pdf`;
    doc.save(filename);
  }

  private static extractInvoiceData(spreadsheetData: any): any {
    // This function extracts data from the SocialCalc spreadsheet format
    // You'll need to adapt this based on your actual data structure
    
    const extracted = {
      invoiceNumber: '',
      date: '',
      billTo: '',
      from: '',
      items: [] as any[],
      total: 0
    };

    // Extract data from the current sheet
    if (spreadsheetData?.msc?.sheetArr) {
      const currentSheet = spreadsheetData.msc.sheetArr[spreadsheetData.msc.currentid];
      if (currentSheet?.sheetstr?.savestr) {
        const lines = currentSheet.sheetstr.savestr.split('\n');
        
        lines.forEach(line => {
          if (line.startsWith('cell:')) {
            const parts = line.split(':');
            const cellRef = parts[1];
            const cellData = parts.slice(2).join(':');
            
            // Extract specific data based on cell patterns
            // This is a simplified example - you may need to adjust based on your templates
            if (cellData.includes('INVOICE #') || cellData.includes('Invoice #')) {
              // Try to find invoice number in adjacent cells
            }
            if (cellData.includes('DATE') || cellData.includes('Date')) {
              // Try to find date in adjacent cells
            }
            // Add more extraction logic as needed
          }
        });
      }
    }

    return extracted;
  }

  static async exportToExcel(spreadsheetData: any): Promise<void> {
    // For Excel export, you could use libraries like xlsx
    // This is a placeholder implementation
    console.log('Excel export not implemented yet');
    throw new Error('Excel export functionality coming soon');
  }
}
```

**src/components/LoginForm/LoginForm.tsx**
```typescript
import React, { useState } from 'react';
import {
  IonButton,
  IonCard,
  IonCardContent,
  IonCardHeader,
  IonCardTitle,
  IonInput,
  IonItem,
  IonLabel,
  IonAlert,
  IonSpinner,
  IonIcon
} from '@ionic/react';
import { personOutline, lockClosedOutline } from 'ionicons/icons';
import { useAuth } from '../../contexts/AuthContext';
import { useLanguage } from '../../contexts/LanguageContext';
import './LoginForm.css';

export const LoginForm: React.FC = () => {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [showError, setShowError] = useState(false);
  const { login, isLoading, error } = useAuth();
  const { t } = useLanguage();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!username.trim() || !password.trim()) {
      setShowError(true);
      return;
    }

    const success = await login({ username, password });
    if (!success) {
      setShowError(true);
    }
  };

  return (
    <div className="login-container">
      <IonCard className="login-card">
        <IonCardHeader>
          <IonCardTitle className="login-title">
            {t('login.title')}
          </IonCardTitle>
        </IonCardHeader>
        
        <IonCardContent>
          <form onSubmit={handleSubmit}>
            <IonItem>
              <IonIcon icon={personOutline} slot="start" />
              <IonLabel position="stacked">{t('login.username')}</IonLabel>
              <IonInput
                type="text"
                value={username}
                onIonInput={(e) => setUsername(e.detail.value!)}
                required
                disabled={isLoading}
              />
            </IonItem>

            <IonItem>
              <IonIcon icon={lockClosedOutline} slot="start" />
              <IonLabel position="stacked">{t('login.password')}</IonLabel>
              <IonInput
                type="password"
                value={password}
                onIonInput={(e) => setPassword(e.detail.value!)}
                required
                disabled={isLoading}
              />
            </IonItem>

            <IonButton 
              expand="full" 
              type="submit" 
              className="login-button"
              disabled={isLoading}
            >
              {isLoading ? (
                <IonSpinner name="crescent" />
              ) : (
                t('login.submit')
              )}
            </IonButton>
          </form>

          <div className="demo-credentials">
            <p><strong>{t('login.demoCredentials')}</strong></p>
            <p>Admin: admin / password123</p>
            <p>Manager: manager / password123</p>
            <p>User: user / password123</p>
            <p>Viewer: viewer / password123</p>
          </div>
        </IonCardContent>
      </IonCard>

      <IonAlert
        isOpen={showError}
        onDidDismiss={() => setShowError(false)}
        header={t('login.error.title')}
        message={error || t('login.error.required')}
        buttons={[t('common.ok')]}
      />
    </div>
  );
};
```

**src/components/LoginForm/LoginForm.css**
```css
.login-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  padding: 20px;
  background: var(--ion-background-color);
}

.login-card {
  max-width: 400px;
  width: 100%;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
}

.login-title {
  text-align: center;
  color: var(--ion-color-primary);
  margin-bottom: 20px;
}

.login-button {
  margin-top: 20px;
  --border-radius: 8px;
}

.demo-credentials {
  margin-top: 20px;
  padding: 15px;
  background: var(--ion-color-light);
  border-radius: 8px;
  font-size: 0.9em;
  text-align: center;
}

.demo-credentials p {
  margin: 5px 0;
}

.demo-credentials strong {
  color: var(--ion-color-primary);
}

/* Dark theme support */
:root.dark .login-card {
  --background: var(--ion-color-step-50);
}

:root.dark .demo-credentials {
  background: var(--ion-color-step-100);
}
```

**src/components/ThemeToggle/ThemeToggle.tsx**
```typescript
import React from 'react';
import { IonButton, IonIcon } from '@ionic/react';
import { moonOutline, sunnyOutline } from 'ionicons/icons';
import { useTheme } from '../../contexts/ThemeContext';
import { useLanguage } from '../../contexts/LanguageContext';

export const ThemeToggle: React.FC = () => {
  const { theme, toggleTheme } = useTheme();
  const { t } = useLanguage();

  return (
    <IonButton
      fill="clear"
      onClick={toggleTheme}
      title={theme === 'light' ? t('theme.switchToDark') : t('theme.switchToLight')}
    >
      <IonIcon icon={theme === 'light' ? moonOutline : sunnyOutline} />
    </IonButton>
  );
};
```

**src/components/LanguageSelector/LanguageSelector.tsx**
```typescript
import React from 'react';
import { IonSelect, IonSelectOption, IonLabel, IonItem } from '@ionic/react';
import { useLanguage } from '../../contexts/LanguageContext';

const LANGUAGES = [
  { code: 'en', name: 'English' },
  { code: 'es', name: 'Español' },
  { code: 'fr', name: 'Français' }
];

export const LanguageSelector: React.FC = () => {
  const { language, setLanguage, t } = useLanguage();

  return (
    <IonItem>
      <IonLabel>{t('language.select')}</IonLabel>
      <IonSelect
        value={language}
        onSelectionChange={(e) => setLanguage(e.detail.value)}
        interface="popover"
      >
        {LANGUAGES.map((lang) => (
          <IonSelectOption key={lang.code} value={lang.code}>
            {lang.name}
          </IonSelectOption>
        ))}
      </IonSelect>
    </IonItem>
  );
};
```

**src/components/ExportMenu/ExportMenu.tsx**
```typescript
import React, { useState } from 'react';
import {
  IonButton,
  IonIcon,
  IonPopover,
  IonList,
  IonItem,
  IonLabel,
  IonAlert,
  IonSpinner
} from '@ionic/react';
import { 
  downloadOutline, 
  documentTextOutline, 
  documentOutline 
} from 'ionicons/icons';
import { PdfExportService } from '../../services/PdfExportService';
import { useLanguage } from '../../contexts/LanguageContext';
import { useAuth } from '../../contexts/AuthContext';

interface ExportMenuProps {
  invoiceData: any;
  disabled?: boolean;
```
