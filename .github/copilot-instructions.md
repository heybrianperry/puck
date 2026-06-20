# Puck AI Provider Instructions

## ADR 1: Client-Side Components - Credential Security

Client-side components MUST NOT expose sensitive API credentials or authentication tokens in client-accessible code; authentication MUST be handled through secure server-side proxies

---

## ADR 2: Client-Side Integration - Optimistic Updates

Client-side API integration components MAY implement optimistic updates to improve perceived performance while awaiting server confirmation

---

## ADR 3: Dynamic Route Parameters - Validation and Sanitization

Dynamic route parameters used in client-side API integrations SHOULD be validated and sanitized before being passed to external API endpoints

---

## ADR 4: Client-Side Components - Error Handling

Client-side components consuming external APIs MUST implement proper error boundaries and loading states to handle network failures gracefully

---

## ADR 5: Visual Content Editing - State Management

Visual content editing interfaces that integrate with external APIs SHOULD implement client-side state management to handle real-time updates without server round-trips

---

## ADR 6: Client-Side Integration - File Organization

Client-side API integration components MUST be isolated in dedicated files with clear naming conventions (e.g., client.tsx, *.client.tsx) to distinguish them from server-rendered components

---

## ADR 7: External Integration Components - Client-Side Rendering

External API integration components that provide interactive editing capabilities MUST be implemented as client-side rendered components using the 'use client' directive in Next.js or equivalent client-side markers in other frameworks

---

## ADR 8: Console Logging - Data Sanitization

Any data logged MUST be sanitized to remove or redact sensitive fields before being passed to console methods

---

## ADR 9: Console Logging - Structured Logging Wrappers

Teams MAY implement structured logging wrappers that provide consistent formatting and automatic sanitization of sensitive data

---

## ADR 10: Production Builds - Console Logging Optimization

Production builds SHOULD strip or disable console logging statements through build optimization tools (e.g., terser, babel plugins) to reduce bundle size

---

## ADR 11: Development Logging - Contextual Information

Development logging SHOULD include contextual information such as component names, function names, or operation identifiers to aid in debugging

---

## ADR 12: Console Logging - Log Levels

All console logging MUST use appropriate log levels (console.log for info, console.warn for warnings, console.error for errors) to enable proper filtering and monitoring

---

## ADR 13: Console Logging - Sensitive Information

Logging statements MUST NOT include sensitive information such as authentication tokens, API keys, passwords, personal identifiable information (PII), or any data marked as confidential

---

## ADR 14: Console Logging - Conditional Execution

Console logging statements MUST be conditionally executed based on environment variables or build-time flags to prevent execution in production

---

## ADR 78: Public-Facing Applications - Credential Security

Public-facing applications MUST NOT hardcode API keys, tokens, or sensitive credentials in source code or configuration files

---

## ADR 79: External Analytics - Secure Configuration

External analytics and monitoring service integrations MUST use environment variables or secure configuration management for API keys and secrets

---

## ADR 142: Sensitive Configuration - Version Control

Sensitive configuration values (API keys, secrets, credentials) MUST_NOT be committed to version control or included in client-side bundles

---

## ADR 146: Runtime Configuration - Environment Variables

All runtime configuration values MUST be accessed via process.env rather than hardcoded in application code

---

## ADR 155: Console-Based Logging - Custom Utilities

Applications MAY wrap console methods with custom logging utilities for enhanced formatting or filtering while maintaining console as the underlying transport

---

## ADR 156: Core Libraries - Logging Dependencies

Core libraries MUST NOT introduce third-party logging frameworks as required dependencies

---

## ADR 157: Development Debugging - Console Output

Development and debugging logs SHOULD use console.log for informational output that aids troubleshooting

---

## ADR 158: Warning Conditions - Console Methods

Warning conditions SHOULD use console.warn to distinguish severity levels from informational logs

---

## ADR 159: Error Conditions - Console Error

Error conditions and exceptional states MUST be logged using console.error with descriptive context

---

## ADR 160: Core Library Functions - Logging

All core library functions MUST use console methods (console.log, console.warn, console.error) for logging and debugging output
