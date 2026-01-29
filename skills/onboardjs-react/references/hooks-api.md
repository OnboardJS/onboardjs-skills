# useOnboarding Hook API Reference

## Table of Contents
- [Basic Usage](#basic-usage)
- [Return Value](#return-value)
- [State Object](#state-object)
- [Loading States](#loading-states)
- [Navigation Methods](#navigation-methods)
- [Context Methods](#context-methods)
- [Rendering](#rendering)

---

## Basic Usage

```tsx
import { useOnboarding } from '@onboardjs/react'

function OnboardingUI() {
  const {
    state,
    loading,
    next,
    previous,
    skip,
    goToStep,
    updateContext,
    reset,
    renderStep,
    currentStep,
    isCompleted,
    error,
    engine
  } = useOnboarding()

  // Use these values to build your UI
}
```

---

## Return Value

```typescript
interface UseOnboardingReturn<TContext> {
  // Core
  engine: OnboardingEngine<TContext> | null
  state: EngineState<TContext> | null

  // Loading states
  loading: {
    isHydrating: boolean
    isEngineProcessing: boolean
    isComponentProcessing: boolean
    isAnyLoading: boolean
  }

  // Navigation
  next: (stepData?: Record<string, unknown>) => Promise<void>
  previous: () => Promise<void>
  skip: () => Promise<void>
  goToStep: (stepId: string, data?: unknown) => Promise<void>

  // Context
  updateContext: (data: Partial<TContext>) => Promise<void>
  reset: (config?: Partial<OnboardingEngineConfig>) => Promise<void>

  // Rendering
  renderStep: () => React.ReactNode

  // Convenience
  currentStep: OnboardingStep<TContext> | null
  isCompleted: boolean
  error: Error | null

  // Loading control
  setComponentLoading: (loading: boolean) => void
  isLoading: boolean  // Deprecated: use loading.isAnyLoading
}
```

---

## State Object

The `state` object contains the current engine state:

```typescript
interface EngineState<TContext> {
  currentStep: OnboardingStep<TContext> | null
  context: TContext
  isLoading: boolean
  isHydrating: boolean
  isCompleted: boolean
  isFirstStep: boolean
  isLastStep: boolean
  canGoNext: boolean
  canGoPrevious: boolean
  isSkippable: boolean
  error: Error | null
  currentStepNumber?: number  // May be undefined
  totalSteps?: number         // May be undefined
}
```

> **Warning:** `state.steps` does NOT exist. The state object does not contain the steps array. For step indicators, use `currentStepNumber`/`totalSteps` (if available) or maintain your own `STEP_IDS` array and calculate the index from `currentStep.id`. See the Step Indicator section in SKILL.md.

### Usage Example

```tsx
function OnboardingUI() {
  const { state } = useOnboarding()

  if (!state) return null

  return (
    <div>
      <progress value={state.currentStepNumber} max={state.totalSteps} />
      <p>Step {state.currentStepNumber} of {state.totalSteps}</p>

      {state.error && <ErrorBanner message={state.error.message} />}

      <button disabled={state.isFirstStep}>Back</button>
      <button disabled={!state.canGoNext}>
        {state.isLastStep ? 'Complete' : 'Continue'}
      </button>
      {state.isSkippable && <button>Skip</button>}
    </div>
  )
}
```

---

## Loading States

OnboardJS provides granular loading states:

```typescript
loading: {
  isHydrating: boolean        // Initial load from persistence
  isEngineProcessing: boolean // During next/previous/goToStep
  isComponentProcessing: boolean // Custom component async operations
  isAnyLoading: boolean       // True if any above are true
}
```

### Usage Patterns

```tsx
function OnboardingUI() {
  const { loading, renderStep } = useOnboarding()

  // Show skeleton during initial hydration
  if (loading.isHydrating) {
    return <OnboardingSkeleton />
  }

  // Disable buttons during navigation
  return (
    <div>
      {renderStep()}
      <button disabled={loading.isEngineProcessing}>
        {loading.isEngineProcessing ? 'Loading...' : 'Next'}
      </button>
    </div>
  )
}
```

### setComponentLoading

For async operations within step components:

```tsx
function AsyncStepComponent({ onDataChange }) {
  const { setComponentLoading, loading } = useOnboarding()

  const handleAsyncAction = async () => {
    setComponentLoading(true)
    try {
      const result = await fetchSomething()
      onDataChange?.(result, true)
    } finally {
      setComponentLoading(false)
    }
  }

  return (
    <button onClick={handleAsyncAction} disabled={loading.isComponentProcessing}>
      {loading.isComponentProcessing ? 'Processing...' : 'Load Data'}
    </button>
  )
}
```

---

## Navigation Methods

### next(stepData?)

Navigate to the next step, optionally passing data.

```tsx
const { next } = useOnboarding()

// Simple navigation
await next()

// With step data (merged into flowData)
await next({ selectedPlan: 'premium', billingCycle: 'annual' })
```

The `stepData` is:
1. Passed to `onStepComplete` callback
2. Merged into `context.flowData`
3. Available in subsequent steps via `context.flowData`

### previous()

Navigate to the previous step.

```tsx
const { previous, state } = useOnboarding()

<button onClick={previous} disabled={state?.isFirstStep}>
  Back
</button>
```

### skip()

Skip the current step (only works if `isSkippable: true`).

```tsx
const { skip, state } = useOnboarding()

{state?.isSkippable && (
  <button onClick={skip}>Skip this step</button>
)}
```

### goToStep(stepId, data?)

Jump directly to a specific step.

```tsx
const { goToStep } = useOnboarding()

// Jump to step
await goToStep('settings')

// Jump with data
await goToStep('review', { fromStep: 'summary' })
```

---

## Context Methods

### updateContext(data)

Update the shared context without navigating.

```tsx
const { updateContext, state } = useOnboarding()

// Update user info
await updateContext({
  currentUser: { id: '123', name: 'John' }
})

// Update flowData
await updateContext({
  flowData: {
    ...state?.context.flowData,
    customField: 'value'
  }
})
```

### reset(config?)

Reset the entire flow. Optionally provide new configuration.

```tsx
const { reset } = useOnboarding()

// Full reset to initial state
await reset()

// Reset with new initial context
await reset({
  initialContext: { flowData: { resetReason: 'user_requested' } }
})

// Reset to specific step
await reset({ initialStepId: 'welcome' })
```

---

## Rendering

### renderStep()

Renders the current step component from the registry.

```tsx
function OnboardingUI() {
  const { renderStep, state, loading } = useOnboarding()

  if (loading.isHydrating) return <Skeleton />
  if (state?.isCompleted) return <CompletedView />

  return (
    <div className="onboarding-container">
      <StepIndicator current={state?.currentStepNumber} total={state?.totalSteps} />
      <div className="step-content">
        {renderStep()}
      </div>
      <NavigationButtons />
    </div>
  )
}
```

### How renderStep Works

1. Gets current step from state
2. Looks up component in registry by:
   - `payload.componentKey` (if present)
   - `step.type` (fallback)
3. Passes `StepComponentProps` to component:

```typescript
interface StepComponentProps<P, TContext> {
  payload: P                    // Step's payload
  context: TContext             // Current context
  onDataChange?: (data, isValid) => void  // Report data back
  initialData?: Record<string, unknown>   // Pre-filled data
}
```

---

## Accessing the Engine Directly

For advanced use cases, access the engine directly:

```tsx
const { engine } = useOnboarding()

useEffect(() => {
  if (!engine) return

  // Subscribe to events
  const unsubscribe = engine.addEventListener('stepCompleted', (event) => {
    analytics.track('step_completed', { stepId: event.step.id })
  })

  return () => unsubscribe()
}, [engine])
```

See [patterns.md](patterns.md) for event system details.
