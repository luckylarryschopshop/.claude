---
name: mobile
description: Mobile application framework — agents, phases, and skill loading for iOS/Android products.
---

# Framework: Mobile Application

## Agent Roster
| Agent | Phase | Trigger |
|-------|-------|---------|
| PM | 1 | Define requirements and platform strategy |
| UX | 1–2 | Mobile-specific user flows and gestures |
| Designer | 2 | Platform-native design language |
| Architect | 2 | State management and API strategy |
| Backend | 2–3 | Mobile-optimised API (if building backend) |
| Frontend | 3–4 | React Native or native implementation |
| Security | 4 | Mobile-specific security review |
| DevOps | 4–5 | CI/CD and app store submission |
| Tester | All | After every phase — mandatory gate |

## Phase Structure

### Phase 1 — Discovery + Platform Strategy
Goal: Requirements defined; platform strategy agreed (React Native vs native vs PWA).
Agents: PM → UX
Skills: SKILL-requirements, SKILL-ux-research, SKILL-mobile
Key decisions: iOS-only / Android-only / both; React Native vs native; min OS version

### Phase 2 — Architecture and Design
Goal: App architecture settled; design system aligned to platform conventions.
Agents: Architect + Designer (parallel)
Skills: SKILL-system-design, SKILL-visual-design, SKILL-mobile
Note: iOS uses Human Interface Guidelines; Android uses Material Design — Designer must align

### Phase 3 — Core Implementation
Goal: Core screens and flows working on device.
Agents: Frontend (React Native) + Backend (API, if applicable) — parallel
Skills: SKILL-frontend, SKILL-mobile, SKILL-code-quality, SKILL-tdd, SKILL-api-design
Must test on real devices, not just emulator, before TESTER approval

### Phase 4 — Polish, Security, and Compliance
Goal: App store ready; security reviewed; permissions and privacy labels complete.
Agents: Frontend (polish) → Security → Tester
Skills: SKILL-security, SKILL-security-audit, SKILL-mobile
App store requirements: privacy manifest, data safety form, screenshots, description

### Phase 5 — CI/CD and Submission
Goal: Automated build pipeline; app submitted to stores.
Agents: DevOps → Tester
Skills: SKILL-devops, SKILL-mobile
Tools: Fastlane, Expo EAS, or Bitrise for automated signing and submission

## Mobile-Specific Considerations
- **Deep linking**: define URL scheme and universal links in Phase 2
- **Offline mode**: decide in Phase 1 — harder to retrofit
- **Push notifications**: backend infrastructure needed before frontend implementation
- **Analytics**: instrument in Phase 3; agree event taxonomy with PM first
- **Crash reporting**: Sentry or Firebase Crashlytics — wire up before first TestFlight build

## Skill Loading Table
| Phase | Load these skills |
|-------|-------------------|
| 1 | SKILL-requirements, SKILL-ux-research, SKILL-mobile |
| 2 | SKILL-system-design, SKILL-visual-design, SKILL-mobile |
| 3 | SKILL-frontend, SKILL-mobile, SKILL-code-quality, SKILL-tdd |
| 4 | SKILL-security, SKILL-security-audit, SKILL-mobile |
| 5 | SKILL-devops |
