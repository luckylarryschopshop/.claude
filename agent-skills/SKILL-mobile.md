---
name: mobile
description: Mobile development methodology — React Native, performance, platform conventions, app store. Load when acting as Frontend agent on mobile projects.
---

# Mobile Skill

## Core Methodology

### React Native Architecture Principles
- **Navigation**: React Navigation is the standard; use native stack navigator for performance
- **State**: same principles as web — local state first, lift when shared, global (Zustand/Redux) when truly app-wide
- **Platform differences**: use `Platform.OS` checks sparingly; prefer cross-platform components
- **Native modules**: use well-maintained community libraries before writing native code
- **Performance budget**: 60fps for all animations; Hermes engine enabled by default (RN 0.70+)

### Performance Rules
```javascript
// ALWAYS use useCallback for functions passed as props
const handlePress = useCallback(() => { ... }, [dependencies]);

// ALWAYS use memo for expensive components
const ExpensiveList = memo(({ items }) => { ... });

// ALWAYS use FlatList (not ScrollView + map) for lists > 20 items
<FlatList
  data={items}
  keyExtractor={(item) => item.id}
  renderItem={({ item }) => <ItemComponent item={item} />}
  getItemLayout={...} // Add when items have fixed height for performance
/>

// Use InteractionManager for post-navigation heavy work
InteractionManager.runAfterInteractions(() => {
  // expensive setup
});
```

### Platform-Specific Guidelines

**iOS:**
- Safe area insets: always use `SafeAreaView` or `useSafeAreaInsets()`
- Navigation: back swipe gesture must work (never disable without reason)
- Typography: use SF Pro (system font) as default; -apple-system
- Status bar: manage explicitly with `StatusBar` component

**Android:**
- Back button: handle with `BackHandler` for modals and nested navigators
- Typography: Roboto as system default
- Ripple effect: use `TouchableNativeFeedback` instead of `TouchableOpacity` on Android
- Permissions: request at point of use, explain why before the system dialog

### Touch Targets and Spacing
| Element | Minimum size |
|---------|-------------|
| Tap target | 44pt (iOS) / 48dp (Android) |
| Bottom navigation items | 56dp tall |
| List items | 48dp minimum height |
| Form inputs | 48dp minimum height |

### Push Notifications
- Request permission only when user has experienced the value first
- Provide a pre-permission explanation screen ("We'll notify you when...")
- Never request notifications on app launch without context
- Deep link from notification to the relevant in-app screen

### App Store Requirements
**iOS App Store:**
- Privacy nutrition labels: declare all data collected
- Privacy manifest required for certain APIs (NSPrivacyAccessedAPITypes)
- No references to other platforms or pricing from other stores

**Google Play:**
- Data safety section: complete for all data types collected and shared
- Target API: must be within 1 year of current target API requirement
- 64-bit requirement: all native code must include arm64 slice

### Offline and Network Handling
- All API calls must handle: loading, success, error, and no-network states
- Use optimistic updates for user-initiated actions (faster perceived performance)
- Cache API responses for offline read access (React Query, SWR, or custom)
- Sync queue for offline writes: store locally, replay when connected
- Show offline banner when network is unavailable
