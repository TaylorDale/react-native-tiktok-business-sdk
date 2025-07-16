# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a React Native bridge for the TikTok Business SDK v1.4.1, enabling JavaScript apps to initialize the TikTok SDK, identify users, and track various events (standard, content, and custom events). The library supports both iOS and Android platforms with promise-based async APIs.

## Development Commands

### Basic Commands
- **Build**: `yarn prepare` (uses react-native-builder-bob)
- **Test**: `yarn test` (Jest with 93.75% coverage)
- **Lint**: `yarn lint` (ESLint on JS/TS/TSX files)
- **Type Check**: `yarn typecheck` (TypeScript compiler)
- **Clean**: `yarn clean` (removes build artifacts)
- **Release**: `yarn release` (uses release-it)

### Testing Commands
- **All tests**: `yarn test`
- **With coverage**: `yarn test --coverage`
- **Watch mode**: `yarn test --watch`
- **Specific test file**: `yarn test src/__tests__/index.test.tsx`
- **Exclude error handling tests**: `yarn test --testPathIgnorePatterns="error-handling.test.tsx"`
- **Run quality checks**: `yarn test && yarn lint && yarn typecheck`

## Project Architecture

### Core Structure
- **src/index.tsx**: Main entry point exposing the TikTokBusiness module and all enums
- **android/**: Kotlin implementation (TikTokBusinessModule.kt, TikTokBusinessPackage.kt)
- **ios/**: Swift implementation (TikTokBusinessModule.swift)
- **example/**: Demo app showing SDK usage

### Key Components

#### JavaScript Bridge (src/index.tsx)
- Exports main `TikTokBusiness` object with methods: `initializeSdk`, `identify`, `logout`, `trackEvent`, `trackContentEvent`, `trackCustomEvent`
- All methods return promises for proper async/await handling
- **initializeSdk**: Requires appId, ttAppId, accessToken (mandatory in v1.4.1), and optional debug flag
- **identify**: Takes 4 parameters: externalId, externalUserName, phoneNumber, email (uses `identifyWithExternalID` internally)
- Defines enums for event names and parameters:
  - `TikTokEventName`: Standard events (REGISTRATION, LOGIN, etc.)
  - `TikTokContentEventName`: Content events (ADD_TO_CART, PURCHASE, etc.)
  - `TikTokContentEventParameter`: Event parameters
  - `TikTokContentEventContentsParameter`: Content item parameters

#### Native Modules
- **Android**: `android/src/main/java/com/tiktokbusiness/TikTokBusinessModule.kt` (TikTok Business SDK v1.4.1)
- **iOS**: `ios/TikTokBusinessModule.swift` (TikTok Business SDK v1.4.1)
- Both implement promise-based async methods using RCTPromiseResolveBlock/RCTPromiseRejectBlock
- **iOS Implementation Notes**:
  - Uses modern TikTok SDK APIs: `identifyWithExternalID`, `trackTTEvent`, `initializeSdk` with completionHandler
  - Avoids deprecated methods like `trackEvent` (use `trackTTEvent` instead)
  - Proper error handling with specific error codes (INIT_ERROR, IDENTIFY_ERROR, etc.)
  - Uses `TikTokConfig(accessToken:appId:tiktokAppId:)` constructor pattern
- **Android Implementation Notes**:
  - Uses promise-based async methods with proper error handling
  - Implements `TTInitCallback` for initialization success/failure handling
  - Uses `TTConfig.setAccessToken()` for mandatory access token
  - Proper try-catch blocks around all native SDK calls

### Build System
- Uses `react-native-builder-bob` for building CommonJS, ES Module, and TypeScript definitions
- Outputs to `lib/` directory with separate commonjs, module, and typescript folders
- TypeScript configuration in `tsconfig.json` with strict mode enabled

### Testing Architecture
- **Comprehensive test suite** with 70+ tests achieving 93.75% coverage
- **Test files structure**:
  - `index.test.tsx`: Core functionality unit tests for all SDK methods
  - `edge-cases.test.tsx`: Edge cases, error handling, and performance scenarios
  - `integration.test.tsx`: Complete user flow tests (e-commerce, gaming scenarios)
  - `types.test.ts`: TypeScript type validation and enum testing
  - `error-handling.test.tsx`: Native module linking error scenarios
  - `setup.ts`: Test configuration, mocking utilities, and custom matchers
- **Native module mocking**: Comprehensive mock setup for React Native bridge testing
- **Coverage configuration**: Excludes test files, includes TypeScript sources
- **Custom Jest matchers**: Extended assertions for event tracking validation

### Code Quality
- ESLint with React Native and Prettier configs
- Commitlint for conventional commits
- Lefthook for git hooks
- Release-it for automated releases with conventional changelog

## Key Development Patterns

### Error Handling
- Main module uses Proxy pattern for graceful failure when native module isn't linked
- All native methods are async and return promises
- Error handling tests use `await expect(async () => {...}).rejects.toThrow()` pattern

### Event Tracking Architecture
- **Standard events**: Use `TikTokEventName` enum with optional properties
- **Content events**: Use `TikTokContentEventName` with structured parameters via enums
- **Custom events**: Accept any event name with flexible properties
- **Parameter validation**: TypeScript enums ensure consistent parameter naming

### Testing Best Practices
- Mock native modules in `setup.ts` for consistent test environment
- Use `jest.resetModules()` for testing module linking errors
- Test both success and failure scenarios for all methods
- Include integration tests for complete user flows

## Important Notes

### SDK Version and Compatibility
- Updated to TikTok Business SDK v1.4.1 (iOS and Android)
- **Breaking Change**: `accessToken` is now mandatory in `initializeSdk` (required by v1.4.1)
- Uses modern, non-deprecated APIs throughout (future-proof implementation)

### Development Requirements
- The library requires native linking (not compatible with Expo Go)
- iOS: Uses CocoaPods with TikTokBusinessSDK v1.4.1 dependency
- Android: Uses Gradle with tiktok-business-android-sdk v1.4.1 dependency
- Android requires ProGuard rules for TikTok classes: `-keep class com.tiktok.** { *; }`

### API Signatures
- All native methods are async and return promises with proper error handling
- `initializeSdk(appId, ttAppId, accessToken, debug?)` - accessToken is required
- `identify(externalId, externalUserName, phoneNumber, email)` - all 4 parameters required
- Error handling tests may show expected errors in console - this is normal behavior

### Testing and Quality
- Run `yarn test && yarn lint && yarn typecheck` before commits
- 93.75% test coverage with comprehensive integration scenarios
- Native module mocking handles React Native bridge testing complexities