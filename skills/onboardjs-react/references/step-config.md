# Step Configuration Reference

## Table of Contents
- [OnboardingStep Type](#onboardingstep-type)
- [Step Properties](#step-properties)
- [Navigation Functions](#navigation-functions)
- [Payload Examples](#payload-examples)
- [Lifecycle Callbacks](#lifecycle-callbacks)

---

## OnboardingStep Type

```typescript
import { OnboardingStep } from '@onboardjs/react'

type OnboardingStep<TContext> = {
  id: string | number
  component: React.ComponentType<StepComponentProps>  // Required: Your step component
  payload?: any
  nextStep?: string | null | ((context: TContext) => string | null)
  previousStep?: string | null | ((context: TContext) => string | null)
  condition?: (context: TContext) => boolean
  isSkippable?: boolean
  skipToStep?: string | null | ((context: TContext) => string | null)
  onStepActive?: (context: TContext) => Promise<void> | void
  onStepComplete?: (stepData: any, context: TContext) => Promise<void> | void
  meta?: Record<string, any>
}
```

---

## Step Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `id` | `string \| number` | Yes | Unique step identifier |
| `component` | `React.ComponentType` | Yes | The React component to render |
| `payload` | `any` | No | Data passed to step component |
| `nextStep` | `string \| function \| null` | No | Next step ID or dynamic function |
| `previousStep` | `string \| function \| null` | No | Previous step ID or dynamic function |
| `condition` | `function` | No | Show/hide step based on context |
| `isSkippable` | `boolean` | No | Allow skipping this step |
| `skipToStep` | `string \| function \| null` | No | Where to navigate when skipped |
| `onStepActive` | `function` | No | Called when step becomes active |
| `onStepComplete` | `function` | No | Called when step completes |
| `meta` | `object` | No | Custom metadata |

---

## Navigation Functions

Navigation properties can be static or dynamic:

### Static Navigation
```typescript
{ nextStep: 'step-2' }
{ previousStep: 'step-1' }
{ skipToStep: 'final-step' }
```

### Dynamic Navigation
```tsx
import { PlanStep } from './components/PlanStep'

{
  id: 'plan',
  component: PlanStep,
  nextStep: (context) => {
    if (context.flowData.plan === 'premium') return 'premium-features'
    if (context.flowData.plan === 'basic') return 'basic-features'
    return 'free-features'
  }
}
```

### Null = End of Flow
```typescript
{ nextStep: null } // This is the last step
```

---

## Payload Examples

Payloads are flexible - pass any data your component needs:

### Welcome/Information Step
```tsx
import { WelcomeStep } from './components/WelcomeStep'

{
  id: 'welcome',
  component: WelcomeStep,
  payload: {
    title: 'Welcome',
    description: 'Get started with our app',
    imageUrl: '/welcome.png'
  }
}
```

### Single Choice Step
```tsx
import { RoleSelectStep } from './components/RoleSelectStep'

{
  id: 'role',
  component: RoleSelectStep,
  payload: {
    title: 'Select your role',
    options: [
      { value: 'startup', label: 'Startup', description: '1-10 employees' },
      { value: 'growth', label: 'Growth', description: '11-100 employees' },
      { value: 'enterprise', label: 'Enterprise', description: '100+ employees' }
    ]
  }
}
```

### Multiple Choice Step
```tsx
import { FeatureSelectStep } from './components/FeatureSelectStep'

{
  id: 'features',
  component: FeatureSelectStep,
  payload: {
    title: 'Select features',
    options: [
      { value: 'analytics', label: 'Analytics' },
      { value: 'reports', label: 'Reports' },
      { value: 'api', label: 'API Access' }
    ],
    minSelections: 1,
    maxSelections: 3
  }
}
```

### Checklist Step
```tsx
import { SetupChecklistStep } from './components/SetupChecklistStep'

{
  id: 'setup',
  component: SetupChecklistStep,
  payload: {
    items: [
      { id: 'profile', label: 'Complete your profile' },
      { id: 'invite', label: 'Invite a team member' },
      { id: 'project', label: 'Create your first project' }
    ],
    dataKey: 'setupChecklist',
    minItemsToComplete: 2
  }
}
```

### Form Step
```tsx
import { ProfileFormStep } from './components/ProfileFormStep'

{
  id: 'profile',
  component: ProfileFormStep,
  payload: {
    fields: ['name', 'email', 'company'],
    required: ['name', 'email']
  }
}
```

---

## Lifecycle Callbacks

### onStepActive
Called when step becomes the current step (after navigation completes).

```tsx
import { AnalyticsStep } from './components/AnalyticsStep'

{
  id: 'analytics',
  component: AnalyticsStep,
  onStepActive: async (context) => {
    // Track step view
    analytics.track('step_viewed', { stepId: 'analytics' })

    // Fetch data needed for this step
    const data = await fetchAnalyticsConfig()
    // Note: To update context, use engine.updateContext() from event listener
  }
}
```

### onStepComplete
Called when user completes the step (before navigation to next step).

```tsx
import { ProfileStep } from './components/ProfileStep'

{
  id: 'profile',
  component: ProfileStep,
  onStepComplete: async (stepData, context) => {
    // Validate data
    if (!stepData.email?.includes('@')) {
      throw new Error('Invalid email')  // Prevents navigation
    }

    // Save to backend
    await saveProfile(stepData)

    // Track completion
    analytics.track('profile_completed', stepData)
  }
}
```

### Throwing Errors
Throwing an error in `onStepComplete` prevents navigation and sets `state.error`.

```typescript
onStepComplete: async (stepData) => {
  const isValid = await validateWithServer(stepData)
  if (!isValid) {
    throw new Error('Validation failed')
  }
}
```
