# Requirements Document

## Introduction

Components in the Stellar-forge app that render token lists or perform formatting calculations currently re-compute on every render cycle, regardless of whether their inputs have changed. This feature applies React memoization primitives — `useMemo`, `useCallback`, and `React.memo` — to expensive operations and child component boundaries to eliminate unnecessary re-renders, reduce CPU overhead, and maintain stable references for downstream consumers.

## Glossary

- **App**: The Stellar-forge React application.
- **TokenList**: The component responsible for rendering a collection of token items.
- **TokenCard**: The child component that renders a single token item within the TokenList.
- **Memoization**: The technique of caching the result of a computation so it is only re-evaluated when its dependencies change.
- **useMemo**: React hook that memoizes the result of an expensive computation.
- **useCallback**: React hook that memoizes a function reference so it remains stable across renders.
- **React.memo**: Higher-order component that prevents a component from re-rendering when its props have not changed.
- **Profiler**: The React DevTools Profiler panel used to measure render counts and timing.
- **Render**: A React component executing its render function to produce a virtual DOM output.
- **Unrelated State Change**: A state update in a parent or sibling component whose value is not consumed by the component under observation.

## Requirements

### Requirement 1: Memoize Token List Filtering and Sorting

**User Story:** As a developer, I want token list filtering and sorting logic wrapped in `useMemo`, so that the derived list is only recomputed when the source data or filter/sort parameters change.

#### Acceptance Criteria

1. WHEN the token source data and filter/sort parameters are unchanged, THE TokenList SHALL return the previously memoized filtered and sorted array without recomputing.
2. WHEN the token source data changes, THE TokenList SHALL recompute the filtered and sorted array.
3. WHEN a filter or sort parameter changes, THE TokenList SHALL recompute the filtered and sorted array.
4. WHILE the TokenList is rendering, THE App SHALL not perform filtering or sorting computations more than once per unique combination of source data and parameters.

---

### Requirement 2: Memoize Event Handler Callbacks Passed to Child Components

**User Story:** As a developer, I want event handler callbacks passed to child components wrapped in `useCallback`, so that child components receive stable function references and are not re-rendered due to reference inequality.

#### Acceptance Criteria

1. WHEN a parent component re-renders due to an unrelated state change, THE parent component SHALL provide the same callback reference to TokenCard as in the previous render.
2. WHEN the dependencies of a callback change, THE parent component SHALL provide a new callback reference to TokenCard.
3. THE App SHALL wrap all event handler props passed to TokenCard in `useCallback` with explicitly declared dependency arrays.

---

### Requirement 3: Memoize TokenCard with React.memo

**User Story:** As a developer, I want the TokenCard component wrapped with `React.memo`, so that it skips re-rendering when its props have not changed.

#### Acceptance Criteria

1. WHEN TokenCard receives the same props as the previous render, THE TokenCard SHALL skip the render phase entirely.
2. WHEN any prop passed to TokenCard changes, THE TokenCard SHALL re-render with the updated props.
3. IF a prop passed to TokenCard is a callback, THEN THE TokenCard SHALL rely on reference equality provided by `useCallback` in the parent to determine whether to re-render.

---

### Requirement 4: Eliminate Re-renders on Unrelated State Changes

**User Story:** As a developer, I want the token list to remain stable when unrelated state changes occur, so that rendering work is proportional to actual data changes.

#### Acceptance Criteria

1. WHEN an unrelated state change occurs in a parent component, THE TokenList SHALL not re-render.
2. WHEN an unrelated state change occurs in a parent component, THE TokenCard SHALL not re-render for any token whose props are unchanged.
3. THE Profiler SHALL show no render entries for TokenList or TokenCard components during unrelated state changes after memoization is applied.

---

### Requirement 5: No Functional Regressions from Memoization

**User Story:** As a developer, I want memoization applied without breaking existing behavior, so that users experience no change in functionality.

#### Acceptance Criteria

1. WHEN a token's data changes, THE TokenCard SHALL reflect the updated data in the next render.
2. WHEN a user interaction triggers a callback, THE App SHALL execute the correct handler with the current closure values.
3. IF a dependency array for `useMemo` or `useCallback` is incomplete, THEN THE App SHALL still produce correct output by including all referenced values in the dependency array.
4. THE App SHALL pass all existing functional tests after memoization is applied.

---

### Requirement 6: Profiling and Render Count Verification

**User Story:** As a developer, I want to profile the app with React DevTools before and after memoization, so that I can verify the optimization has measurably reduced render counts.

#### Acceptance Criteria

1. WHEN a profiling session is recorded before memoization, THE Profiler SHALL capture a baseline render count for TokenList and TokenCard.
2. WHEN a profiling session is recorded after memoization, THE Profiler SHALL show a reduced render count for TokenList and TokenCard compared to the baseline.
3. THE App SHALL support React DevTools Profiler instrumentation without additional configuration.

---

### Requirement 7: Memoization Documentation

**User Story:** As a developer, I want documentation of which components are memoized and why, so that future contributors understand the optimization decisions.

#### Acceptance Criteria

1. THE App SHALL include inline code comments on each `useMemo`, `useCallback`, and `React.memo` usage that describe the dependency rationale.
2. THE App SHALL maintain a developer-facing note (e.g., a code comment block or README section) listing each memoized component and the specific re-render scenario it prevents.
