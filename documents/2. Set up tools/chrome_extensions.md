# Chrome Extensions for Development Productivity

## Overview

Microsoft Edge has been installed instead of Google Chrome for this setup. As Microsoft Edge is a Chromium-based browser, it fully supports all Chrome extensions and provides the same development tools and capabilities. This document outlines the Chrome extensions installed to enhance the development workflow, particularly for React and Redux applications and web performance debugging.

---

## Installed Extensions

### 1. React Developer Tools

**Purpose:** Inspect and debug React component hierarchies, props, and state in real-time.

**Why it's essential:**

- Visual component tree inspection
- Real-time prop and state inspection
- Performance profiling capabilities
- Component highlighting on hover
- Identify components that are re-rendering unnecessarily

**How to use:**

- Access via the Chrome DevTools panel (Components and Profiler tabs)
- Right-click on any React-rendered element and select "Inspect Element"

---

### 2. Redux DevTools

**Purpose:** Debug Redux state management, actions, and time-travel through state changes.

**Why it's essential:**

- Visualise the Redux store state
- Monitor dispatched actions in real-time
- Time-travel debugging (replay actions forward/backward)
- Export and import state snapshots
- Action payload inspection
- Diff view to see state changes between actions

**How to use:**

- Access via the Chrome DevTools panel (Redux tab)
- Requires Redux DevTools Extension setup in your Redux store configuration

---

### 3. JSON Viewer Pro

**Purpose:** An advanced JSON viewer that automatically formats, validates, and beautifies JSON responses from API calls, making them easily readable and debuggable.

**Why it's essential:**

- Automatically detects and formats JSON content in browser tabs
- Enhanced syntax highlighting with colour-coded key-value pairs
- Collapsible/expandable JSON tree structures for navigating large responses
- JSON validation to identify syntax errors instantly
- Search functionality to quickly find specific keys or values within large JSON objects
- Dark mode support for comfortable viewing during long debugging sessions
- Export formatted JSON to clipboard or file
- Line numbers and path indicators for easy reference
- Performance optimised for handling large JSON files efficiently

**How to use:**

- Automatically activates when navigating to URLs that return JSON content
- Works seamlessly with API endpoints, JSON files, and DevTools Network panel responses
- Click on any JSON value to copy it to the clipboard
- Use the search box (Ctrl+F / Cmd+F) to find specific keys or values
- Toggle between formatted and raw view if needed

---

### 4. Lighthouse

**Purpose:** Automated performance, accessibility, best practices, and SEO auditing for web applications.

**Why it's essential:**

- Performance metrics (LCP, FID, CLS, etc.)
- Accessibility audits (WCAG compliance)
- SEO best practices analysis
- Progressive Web App (PWA) validation
- Actionable recommendations for improvements
- Baseline performance tracking

**How to use:**

- Access via Chrome DevTools (Lighthouse tab)
- Run audits on any webpage (mobile or desktop simulation)
- Export reports as JSON or HTML

---

## Most Useful Learning

### **Understanding React Component Re-renders with React DevTools Profiler**

The most valuable insight was learning to use the React DevTools **Profiler** to identify unnecessary re-renders. By recording a session and analysing the flame graph, you can:

1. **Identify expensive renders:** See which components take the longest to render
2. **Detect unnecessary re-renders:** Components re-rendering when props/state haven't changed
3. **Optimise performance:** Use `React.memo`, `useMemo`, or `useCallback` strategically based on profiler data

**Example workflow:**

- Start profiling in React DevTools
- Interact with the application
- Stop profiling and review the flame graph
- Look for components with yellow/orange bars (slow renders)
- Check "Why did this render?" to understand the cause
- Apply optimisations and re-profile to measure improvement

This data-driven approach to performance optimisation is far more effective than guessing which components need optimisation.

---

## Additional Tips

### Best Practices

1. **Keep extensions updated** to avoid compatibility issues
2. **Use Lighthouse regularly** to track performance regressions
3. **Profile in production-like environments** for accurate performance data
4. **Export Lighthouse reports** to track improvements over time

### Extension Links

- [React Developer Tools](https://chromewebstore.google.com/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)
- [Redux DevTools](https://chromewebstore.google.com/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd)
- [JSON Viewer Pro](https://chromewebstore.google.com/detail/json-viewer-pro/eifflpmocdbdmepbjaopkkhbfmdgijcc)
- [Lighthouse](https://chromewebstore.google.com/detail/lighthouse/blipmdconlkpinefehnmjammfjpmpbjk)

---

## Notes

- All extensions should be installed from the official Chrome Web Store
- Some extensions may require additional setup in your application code (e.g., Redux DevTools middleware)
- Consider organisation policies before installing extensions in corporate environments
