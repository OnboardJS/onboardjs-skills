# Advanced Patterns

## Table of Contents
- [Complete Setup Example](#complete-setup-example)
- [Form Validation](#form-validation)
- [Event System](#event-system)
- [Backend Persistence Examples](#backend-persistence-examples)
- [Multi-Branch Flows](#multi-branch-flows)
- [Analytics Integration](#analytics-integration)
- [Plugin System](#plugin-system)
- [TypeScript Patterns](#typescript-patterns)
- [Next.js Patterns](#nextjs-patterns)

---

## Complete Setup Example

Full working example with all pieces:

```tsx
// types.ts
import { OnboardingContext } from '@onboardjs/react'

export interface AppOnboardingContext extends OnboardingContext {
  flowData: {
    name?: string
    email?: string
    role?: 'admin' | 'user'
    features?: string[]
    _internal?: {
      completedSteps: Record<string, number>
      startedAt: number
    }
  }
  currentUser?: {
    id: string
    email: string
  }
}

// steps.tsx
import { OnboardingStep } from '@onboardjs/react'
import { AppOnboardingContext } from './types'
import { WelcomeStep } from './components/WelcomeStep'
import { ProfileForm } from './components/ProfileForm'
import { RoleSelect } from './components/RoleSelect'
import { AdminSetup } from './components/AdminSetup'
import { FeatureSelect } from './components/FeatureSelect'
import { CompleteStep } from './components/CompleteStep'

export const steps: OnboardingStep<AppOnboardingContext>[] = [
  {
    id: 'welcome',
    component: WelcomeStep,
    payload: {
      title: 'Welcome to Acme',
      description: 'Let\'s get you set up in just a few steps.'
    },
    nextStep: 'profile'
  },
  {
    id: 'profile',
    component: ProfileForm,
    nextStep: 'role',
    onStepComplete: async (data, ctx) => {
      await saveProfileToBackend(data)
    }
  },
  {
    id: 'role',
    component: RoleSelect,
    payload: {
      title: 'What\'s your role?',
      options: [
        { value: 'admin', label: 'Admin' },
        { value: 'user', label: 'Team Member' }
      ]
    },
    nextStep: (ctx) => ctx.flowData.role === 'admin' ? 'admin-setup' : 'features'
  },
  {
    id: 'admin-setup',
    component: AdminSetup,
    condition: (ctx) => ctx.flowData.role === 'admin',
    nextStep: 'features'
  },
  {
    id: 'features',
    component: FeatureSelect,
    payload: {
      title: 'Which features interest you?',
      options: [
        { value: 'analytics', label: 'Analytics' },
        { value: 'reports', label: 'Reports' },
        { value: 'api', label: 'API Access' }
      ],
      minSelections: 1
    },
    nextStep: 'complete'
  },
  {
    id: 'complete',
    component: CompleteStep,
    payload: {
      title: 'You\'re all set!',
      message: 'Start exploring your new workspace.'
    },
    nextStep: null
  }
]

// App.tsx
import { OnboardingProvider } from '@onboardjs/react'
import { steps } from './steps'
import { OnboardingUI } from './OnboardingUI'

function App() {
  return (
    <OnboardingProvider
      steps={steps}
      localStoragePersistence={{ key: 'acme_onboarding_v1' }}
      onFlowComplete={(ctx) => {
        console.log('Onboarding complete!', ctx.flowData)
        window.location.href = '/dashboard'
      }}
    >
      <OnboardingUI />
    </OnboardingProvider>
  )
}
```

---

## Form Validation

### Client-side Validation with onDataChange

```tsx
const EmailStep: React.FC<StepComponentProps> = ({ onDataChange, initialData }) => {
  const [email, setEmail] = useState(initialData?.email || '')
  const [error, setError] = useState('')

  const validate = (value: string) => {
    const isValid = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)
    setError(isValid ? '' : 'Invalid email address')
    return isValid
  }

  const handleChange = (value: string) => {
    setEmail(value)
    const isValid = validate(value)
    onDataChange?.({ email: value }, isValid)
  }

  return (
    <div>
      <input
        type="email"
        value={email}
        onChange={(e) => handleChange(e.target.value)}
      />
      {error && <span className="error">{error}</span>}
    </div>
  )
}
```

### Server-side Validation with onStepComplete

```tsx
import { EmailStep } from './components/EmailStep'

{
  id: 'email',
  component: EmailStep,
  onStepComplete: async (stepData, context) => {
    const response = await fetch('/api/validate-email', {
      method: 'POST',
      body: JSON.stringify({ email: stepData.email })
    })

    if (!response.ok) {
      const { error } = await response.json()
      throw new Error(error)  // Prevents navigation, sets state.error
    }
  }
}
```

### Handling Validation Errors in UI

```tsx
function OnboardingUI() {
  const { state, next, renderStep } = useOnboarding()

  return (
    <div>
      {state?.error && (
        <div className="error-banner">
          {state.error.message}
        </div>
      )}
      {renderStep()}
      <button onClick={() => next()}>Continue</button>
    </div>
  )
}
```

---

## Event System

### Available Events

| Event | Trigger | Payload |
|-------|---------|---------|
| `stateChange` | Any state change | `{ state }` |
| `stepActive` | Step becomes current | `{ step, context }` |
| `stepCompleted` | Step completed | `{ step, stepData, context }` |
| `stepSkipped` | Step skipped | `{ step, context }` |
| `flowCompleted` | Flow finished | `{ context }` |
| `flowReset` | Flow reset | `{ context }` |
| `contextUpdate` | Context changed | `{ context, changes }` |
| `error` | Error occurred | `{ error, step?, context }` |
| `beforeStepChange` | Before navigation | `{ fromStep, toStep, context }` |

### Subscribing to Events

```tsx
function AnalyticsTracker() {
  const { engine } = useOnboarding()

  useEffect(() => {
    if (!engine) return

    const unsubscribers = [
      engine.addEventListener('stepCompleted', (event) => {
        analytics.track('onboarding_step_completed', {
          stepId: event.step.id,
          stepNumber: event.step.meta?.stepNumber
        })
      }),
      engine.addEventListener('flowCompleted', (event) => {
        analytics.track('onboarding_completed', {
          totalTime: Date.now() - event.context.flowData._internal?.startedAt
        })
      }),
      engine.addEventListener('error', (event) => {
        errorReporter.capture(event.error, {
          stepId: event.step?.id,
          context: event.context
        })
      })
    ]

    return () => unsubscribers.forEach(fn => fn())
  }, [engine])

  return null  // Invisible component
}
```

### Event Interception with beforeStepChange

```tsx
engine.addEventListener('beforeStepChange', async (event) => {
  // Show confirmation before leaving certain steps
  if (event.fromStep.id === 'important-form') {
    const confirmed = await showConfirmDialog('Are you sure?')
    if (!confirmed) {
      event.preventDefault()  // Cancel navigation
    }
  }
})
```

---

## Backend Persistence Examples

### Supabase

```tsx
import { createClient } from '@supabase/supabase-js'

const supabase = createClient(SUPABASE_URL, SUPABASE_KEY)

<OnboardingProvider
  steps={steps}
  customOnDataLoad={async () => {
    const { data, error } = await supabase
      .from('onboarding_progress')
      .select('context, current_step_id')
      .eq('user_id', userId)
      .single()

    if (error || !data) return undefined
    return {
      context: data.context,
      currentStepId: data.current_step_id
    }
  }}
  customOnDataPersist={async (context, currentStepId) => {
    await supabase
      .from('onboarding_progress')
      .upsert({
        user_id: userId,
        context,
        current_step_id: currentStepId,
        updated_at: new Date().toISOString()
      })
  }}
  customOnClearPersistedData={async () => {
    await supabase
      .from('onboarding_progress')
      .delete()
      .eq('user_id', userId)
  }}
>
```

### REST API

```tsx
<OnboardingProvider
  steps={steps}
  customOnDataLoad={async () => {
    const response = await fetch(`/api/onboarding/${userId}`)
    if (!response.ok) return undefined
    return response.json()
  }}
  customOnDataPersist={async (context, currentStepId) => {
    await fetch(`/api/onboarding/${userId}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ context, currentStepId })
    })
  }}
  customOnClearPersistedData={async () => {
    await fetch(`/api/onboarding/${userId}`, { method: 'DELETE' })
  }}
>
```

---

## Multi-Branch Flows

### Role-based Branching

```tsx
import { OnboardingStep } from '@onboardjs/react'
import { WelcomeStep } from './components/WelcomeStep'
import { RoleSelectStep } from './components/RoleSelectStep'
import { DevToolsStep } from './components/DevToolsStep'
import { DesignToolsStep } from './components/DesignToolsStep'
import { TeamSetupStep } from './components/TeamSetupStep'
import { GenericSetupStep } from './components/GenericSetupStep'
import { CompleteStep } from './components/CompleteStep'

const steps: OnboardingStep[] = [
  { id: 'welcome', component: WelcomeStep, nextStep: 'role-select' },
  {
    id: 'role-select',
    component: RoleSelectStep,
    payload: {
      options: [
        { value: 'developer', label: 'Developer' },
        { value: 'designer', label: 'Designer' },
        { value: 'manager', label: 'Manager' }
      ]
    },
    nextStep: (ctx) => {
      switch (ctx.flowData.role) {
        case 'developer': return 'dev-tools'
        case 'designer': return 'design-tools'
        case 'manager': return 'team-setup'
        default: return 'generic-setup'
      }
    }
  },
  // Developer branch
  { id: 'dev-tools', component: DevToolsStep, condition: (ctx) => ctx.flowData.role === 'developer', nextStep: 'complete' },
  // Designer branch
  { id: 'design-tools', component: DesignToolsStep, condition: (ctx) => ctx.flowData.role === 'designer', nextStep: 'complete' },
  // Manager branch
  { id: 'team-setup', component: TeamSetupStep, condition: (ctx) => ctx.flowData.role === 'manager', nextStep: 'complete' },
  // Fallback
  { id: 'generic-setup', component: GenericSetupStep, nextStep: 'complete' },
  // Convergence point
  { id: 'complete', component: CompleteStep, nextStep: null }
]
```

### Feature Flag Branching

```tsx
import { BetaFeatureStep } from './components/BetaFeatureStep'

{
  id: 'beta-feature',
  component: BetaFeatureStep,
  condition: (ctx) => ctx.currentUser?.featureFlags?.includes('beta_onboarding'),
  nextStep: 'next-step'
}
```

---

## Analytics Integration

### Built-in Analytics

```typescript
const engine = new OnboardingEngine({
  steps,
  analytics: {
    enabled: true,
    autoTrack: true,  // Auto-track step events
    providers: [
      {
        name: 'mixpanel',
        track: (event, properties) => mixpanel.track(event, properties),
        identify: (userId, traits) => mixpanel.identify(userId)
      }
    ]
  }
})
```

### Manual Tracking

```tsx
function OnboardingUI() {
  const { engine, state } = useOnboarding()

  const handleNext = async () => {
    engine?.trackCustomEvent('cta_clicked', {
      stepId: state?.currentStep?.id,
      buttonText: 'Continue'
    })
    await next()
  }
}
```

---

## Plugin System

### Creating a Plugin

```typescript
import { OnboardingPlugin, OnboardingEngine } from '@onboardjs/core'

class TimingPlugin implements OnboardingPlugin {
  name = 'timing-plugin'
  private stepStartTimes: Map<string, number> = new Map()

  async onInstall(engine: OnboardingEngine) {
    engine.addEventListener('stepActive', (event) => {
      this.stepStartTimes.set(event.step.id, Date.now())
    })

    engine.addEventListener('stepCompleted', (event) => {
      const startTime = this.stepStartTimes.get(event.step.id)
      if (startTime) {
        const duration = Date.now() - startTime
        console.log(`Step ${event.step.id} took ${duration}ms`)
      }
    })
  }
}

// Usage
const engine = new OnboardingEngine({ steps })
engine.use(new TimingPlugin())
```

### Official Plugins

- `@onboardjs/plugin-supabase` - Supabase persistence
- `@onboardjs/plugin-posthog` - PostHog analytics
- `@onboardjs/plugin-mixpanel` - Mixpanel analytics

---

## TypeScript Patterns

### Typed Context

```tsx
import { OnboardingContext, OnboardingStep } from '@onboardjs/react'
import { PlanSelectStep } from './components/PlanSelectStep'
import { EnterpriseSetupStep } from './components/EnterpriseSetupStep'
import { BasicSetupStep } from './components/BasicSetupStep'

interface MyContext extends OnboardingContext {
  flowData: {
    email?: string
    plan?: 'free' | 'pro' | 'enterprise'
    teamSize?: number
  }
  currentUser?: {
    id: string
    name: string
  }
}

const steps: OnboardingStep<MyContext>[] = [
  {
    id: 'plan',
    component: PlanSelectStep,
    nextStep: (ctx) => {
      // ctx.flowData is fully typed here
      if (ctx.flowData.plan === 'enterprise') return 'enterprise-setup'
      return 'basic-setup'
    }
  },
  { id: 'enterprise-setup', component: EnterpriseSetupStep, nextStep: null },
  { id: 'basic-setup', component: BasicSetupStep, nextStep: null }
]
```

### Typed Payloads

```tsx
import { StepComponentProps } from '@onboardjs/react'

interface ProfilePayload {
  title: string
  fields: Array<{
    name: string
    type: 'text' | 'email' | 'tel'
    required: boolean
  }>
}

const ProfileStep: React.FC<StepComponentProps<ProfilePayload, MyContext>> = ({
  payload,  // Typed as ProfilePayload
  context,  // Typed as MyContext
  onDataChange
}) => {
  // Full type safety
}

// Use in steps
const steps: OnboardingStep<MyContext>[] = [
  {
    id: 'profile',
    component: ProfileStep,
    payload: {
      title: 'Your Profile',
      fields: [
        { name: 'email', type: 'email', required: true }
      ]
    }
  }
]
```

---

## Next.js Patterns

### Complete App Router Setup

```tsx
// types.ts
import { OnboardingContext } from '@onboardjs/react'

export interface AppContext extends OnboardingContext {
  flowData: {
    name?: string
    email?: string
  }
}

// components/steps/WelcomeStep.tsx
'use client'

import { StepComponentProps } from '@onboardjs/react'

export const WelcomeStep: React.FC<StepComponentProps> = ({ payload }) => (
  <div>
    <h1>{payload.title}</h1>
    <p>{payload.description}</p>
  </div>
)

// components/steps/index.ts
'use client'

export { WelcomeStep } from './WelcomeStep'
export { ProfileStep } from './ProfileStep'
export { CompleteStep } from './CompleteStep'

// lib/onboarding/steps.ts
'use client'

import { OnboardingStep } from '@onboardjs/react'
import { WelcomeStep, ProfileStep, CompleteStep } from '@/components/steps'
import type { AppContext } from '@/types'

export const steps: OnboardingStep<AppContext>[] = [
  {
    id: 'welcome',
    component: WelcomeStep,
    payload: { title: 'Welcome', description: 'Let\'s get started' },
    nextStep: 'profile'
  },
  {
    id: 'profile',
    component: ProfileStep,
    nextStep: 'complete'
  },
  {
    id: 'complete',
    component: CompleteStep,
    nextStep: null
  }
]

// components/OnboardingProvider.tsx
'use client'

import { OnboardingProvider as Provider } from '@onboardjs/react'
import { useRouter } from 'next/navigation'
import { steps } from '@/lib/onboarding/steps'

export function OnboardingProvider({ children }: { children: React.ReactNode }) {
  const router = useRouter()

  return (
    <Provider
      steps={steps}
      localStoragePersistence={{ key: 'onboarding_v1' }}
      onFlowComplete={() => router.push('/dashboard')}
    >
      {children}
    </Provider>
  )
}

// components/OnboardingUI.tsx
'use client'

import { useOnboarding } from '@onboardjs/react'

export function OnboardingUI() {
  const { renderStep, next, previous, state, loading } = useOnboarding()

  if (loading.isHydrating) return <div>Loading...</div>
  if (state?.isCompleted) return <div>Redirecting...</div>

  return (
    <div className="max-w-md mx-auto p-6">
      <div className="mb-4">
        Step {state?.currentStepNumber} of {state?.totalSteps}
      </div>
      {renderStep()}
      <div className="flex gap-2 mt-4">
        <button onClick={previous} disabled={state?.isFirstStep}>
          Back
        </button>
        <button onClick={() => next()} disabled={!state?.canGoNext}>
          {state?.isLastStep ? 'Complete' : 'Continue'}
        </button>
      </div>
    </div>
  )
}

// app/onboarding/page.tsx
import { OnboardingProvider } from '@/components/OnboardingProvider'
import { OnboardingUI } from '@/components/OnboardingUI'

export default function OnboardingPage() {
  return (
    <OnboardingProvider>
      <OnboardingUI />
    </OnboardingProvider>
  )
}
```

### Server Actions for Persistence

```tsx
// app/actions/onboarding.ts
'use server'

import { cookies } from 'next/headers'
import { db } from '@/lib/db'

export async function loadOnboardingData(userId: string) {
  const data = await db.onboarding.findUnique({
    where: { userId }
  })
  return data?.context
}

export async function saveOnboardingData(userId: string, context: any, stepId: string) {
  await db.onboarding.upsert({
    where: { userId },
    update: { context, currentStepId: stepId },
    create: { userId, context, currentStepId: stepId }
  })
}

export async function clearOnboardingData(userId: string) {
  await db.onboarding.delete({ where: { userId } })
}

// components/OnboardingProvider.tsx
'use client'

import { OnboardingProvider as Provider } from '@onboardjs/react'
import { loadOnboardingData, saveOnboardingData, clearOnboardingData } from '@/app/actions/onboarding'
import { useSession } from 'next-auth/react'
import { steps } from '@/lib/onboarding/steps'

export function OnboardingProvider({ children }: { children: React.ReactNode }) {
  const { data: session } = useSession()
  const userId = session?.user?.id

  return (
    <Provider
      steps={steps}
      customOnDataLoad={() => userId ? loadOnboardingData(userId) : Promise.resolve(undefined)}
      customOnDataPersist={(ctx, stepId) => userId ? saveOnboardingData(userId, ctx, stepId) : Promise.resolve()}
      customOnClearPersistedData={() => userId ? clearOnboardingData(userId) : Promise.resolve()}
    >
      {children}
    </Provider>
  )
}
```

### Route Protection Middleware

```tsx
// middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  const onboardingComplete = request.cookies.get('onboarding_complete')

  // Redirect to onboarding if not complete
  if (!onboardingComplete && !request.nextUrl.pathname.startsWith('/onboarding')) {
    return NextResponse.redirect(new URL('/onboarding', request.url))
  }

  // Redirect away from onboarding if already complete
  if (onboardingComplete && request.nextUrl.pathname.startsWith('/onboarding')) {
    return NextResponse.redirect(new URL('/dashboard', request.url))
  }

  return NextResponse.next()
}

export const config = {
  matcher: ['/dashboard/:path*', '/onboarding/:path*']
}
```

### Setting Completion Cookie

```tsx
// components/OnboardingProvider.tsx
'use client'

import { OnboardingProvider as Provider } from '@onboardjs/react'
import { useRouter } from 'next/navigation'
import Cookies from 'js-cookie'
import { steps } from '@/lib/onboarding/steps'

export function OnboardingProvider({ children }: { children: React.ReactNode }) {
  const router = useRouter()

  const handleFlowComplete = (ctx: any) => {
    Cookies.set('onboarding_complete', 'true', { expires: 365 })
    router.push('/dashboard')
  }

  return (
    <Provider steps={steps} onFlowComplete={handleFlowComplete}>
      {children}
    </Provider>
  )
}
```

### Lazy Loading Heavy Step Components

```tsx
// lib/onboarding/steps.ts
'use client'

import dynamic from 'next/dynamic'
import { OnboardingStep } from '@onboardjs/react'

// Eagerly load lightweight steps
import { WelcomeStep } from '@/components/steps/WelcomeStep'
import { CompleteStep } from '@/components/steps/CompleteStep'

// Lazy load heavy steps
const ProfileStep = dynamic(
  () => import('@/components/steps/ProfileStep').then(m => m.ProfileStep),
  { loading: () => <div className="animate-pulse h-64 bg-gray-100 rounded" /> }
)

const IntegrationStep = dynamic(
  () => import('@/components/steps/IntegrationStep').then(m => m.IntegrationStep),
  { loading: () => <div className="animate-pulse h-64 bg-gray-100 rounded" /> }
)

export const steps: OnboardingStep[] = [
  { id: 'welcome', component: WelcomeStep, nextStep: 'profile' },
  { id: 'profile', component: ProfileStep, nextStep: 'integrations' },
  { id: 'integrations', component: IntegrationStep, nextStep: 'complete' },
  { id: 'complete', component: CompleteStep, nextStep: null }
]
```

### Parallel Route Modal (App Router)

```tsx
// app/@modal/(.)onboarding/page.tsx
import { OnboardingProvider } from '@/components/OnboardingProvider'
import { OnboardingModal } from '@/components/OnboardingModal'

export default function OnboardingModalPage() {
  return (
    <OnboardingProvider>
      <OnboardingModal />
    </OnboardingProvider>
  )
}

// components/OnboardingModal.tsx
'use client'

import { useRouter } from 'next/navigation'
import { useOnboarding } from '@onboardjs/react'

export function OnboardingModal() {
  const router = useRouter()
  const { renderStep, state, next, previous } = useOnboarding()

  if (state?.isCompleted) {
    router.back()
    return null
  }

  return (
    <div className="fixed inset-0 bg-black/50 flex items-center justify-center">
      <div className="bg-white rounded-lg p-6 max-w-md w-full">
        {renderStep()}
        <div className="flex gap-2 mt-4">
          <button onClick={previous} disabled={state?.isFirstStep}>Back</button>
          <button onClick={() => next()}>
            {state?.isLastStep ? 'Complete' : 'Continue'}
          </button>
        </div>
      </div>
    </div>
  )
}
```
