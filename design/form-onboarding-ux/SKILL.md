# Form & Onboarding UX

Design user-friendly forms and onboarding flows.

## Rules

### 1. Form Layout
- Single column. Multi-column only for related fields (first name + last name).
- Labels ABOVE inputs (faster scanning), not beside.
- Group related fields: Personal Info, Address, Payment.
- Primary action: right-aligned or full-width at bottom.

### 2. Input States
```
Default:  border-[theme]
Focus:    border-primary + ring + label color change
Filled:   border (unchanged, or checkmark for valid)
Error:    border-error + error message below in red
Disabled: bg-gray-100, cursor-not-allowed
```

### 3. Validation
- Real-time validation on blur (not on every keystroke).
- Show error message below the specific field (not a generic top banner).
- Clear error messages: "Email is required" not "Field validation failed".
- Prevent submission with errors. Disable submit button.

### 4. Multi-Step Forms
```
[Step 1 ●] ── [Step 2 ○] ── [Step 3 ○]
   ↓              ↓              ↓
 Pilih Tipe    Pilih Space   Isi Data
```
- Show progress indicator at top.
- Save progress between steps (don't lose data on back).
- Allow going back to edit.
- Final step: summary/review before submit.

### 5. Onboarding Flow
```
Welcome → Create Profile → Set Preferences → Done! 🎉
```
- 3-5 screens max. Show progress (dot indicators).
- One clear action per screen. No information overload.
- Skip option for non-critical steps.
- Celebratory completion screen (confetti/checkmark).
- Allow returning later — don't lock user out if they close midway.

### 6. Error Recovery
- Don't clear the form on error. Preserve user input.
- If API error: show friendly message, suggest retry.
- If network error: show offline state with retry button.
- Never lose user's work on validation error.

## Anti-Patterns
- ❌ Validating on every keystroke (annoying)
- ❌ Generic error messages
- ❌ Clearing form on validation failure
- ❌ No progress indicator on multi-step
- ❌ No skip/back buttons
