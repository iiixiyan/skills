---
name: manual-to-playwright-automation
description: Convert manual/handwritten test cases into Playwright-based UI automation test scripts. Parses test case formats (Excel, Markdown, JSON, TestLink XML, XMind), analyzes automatable vs non-automatable steps, generates Page Object Model (POM) Playwright code with proper locators, assertions, and test data management.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [playwright, ui-automation, test-case-conversion, testing, e2e]
    related_skills: [test-driven-development, writing-plans, subagent-driven-development]
---

# Manual Test Cases → Playwright UI Automation

## Overview

Transform existing manual test cases into production-quality Playwright UI automation scripts. This skill handles the full pipeline: parsing manual test cases, analyzing each step for automation feasibility, deciding the right Page Object Model structure, generating robust Playwright code, and verifying the generated tests.

**Core principle:** Every manual test step becomes a Playwright action or assertion. Non-automatable steps (visual checks, exploratory actions) are flagged and documented — never silently dropped.

## When to Use

- A set of manual test cases (Excel, Markdown, JSON, XMind, TestLink XML) needs Playwright automation
- Existing test cases need to be migrated to a Playwright framework
- A new feature's test cases are ready, and you need to kick off automation in parallel with development
- The user says "帮我把这些用例转成 Playwright 自动化" or similar

## Input Formats Supported

| Format | Detection | Parser Strategy |
|--------|-----------|-----------------|
| **Excel (.xlsx/.xls)** | File extension | `openpyxl`/`xlrd` — reads each sheet as a test suite, columns as fields |
| **Markdown (.md)** | File extension | Parse headings (## = test suite), tables (each row = test case), code blocks |
| **JSON (.json)** | File extension | JSON structure with test_suite → test_cases → steps |
| **XMind (.xmind)** | File extension | `xmindparser` library; test case nodes under suite nodes |
| **TestLink XML (.xml)** | File extension | XML with `<testcase>`, `<step>`, `<expected_result>` elements |
| **CSV (.csv)** | File extension | Headers row, each row = one test case |

## Step 1: Understand the Project Context

Before generating any code, scan the target project to understand its conventions:

```python
# Find existing Playwright test files
search_files("*.spec.ts", target="files", path="/path/to/project")
# Or for Python
search_files("*.py", target="files", path="/path/to/project")

# Check for package.json / pyproject.toml to understand framework
read_file("/path/to/project/package.json")
read_file("/path/to/project/pyproject.toml")

# Look at any existing Page Object files
search_files("*Page*", target="files", path="/path/to/project")
search_files("*page*", target="files", path="/path/to/project")
search_files("*po*", target="files", path="/path/to/project")

# Check test configuration
read_file("/path/to/project/playwright.config.ts")
```
# Or for Python projects:
```python
read_file("/path/to/project/playwright.config.py")
read_file("/path/to/project/pytest.ini")
read_file("/path/to/project/conftest.py")
```

### Key questions to answer:
1. **Language**: TypeScript (default) or Python?
2. **Page Object pattern**: Already exists? How are pages structured?
3. **Test runner**: Playwright Test (default) or pytest-playwright?
4. **Locator strategy**: data-testid, CSS selectors, text selectors, or role selectors?
5. **Test data**: Fixtures, JSON files, or inline?
6. **Reporting**: Allure, HTML, or custom?

If no existing project exists, propose a standard structure (see Scaffolding section).

## Step 2: Read and Parse the Manual Test Cases

Read the source file and extract test cases.

```python
# Read the file
content = read_file("path/to/manual_test_cases.xlsx")
```

### Parse by format:

**Excel:**
```python
import openpyxl

wb = openpyxl.load_workbook("test_cases.xlsx")
sheet = wb.active
# Expected columns: CaseID, Title, Precondition, Steps, ExpectedResult, Priority, Module
# Map columns by header name (case-insensitive)
```

**Markdown:**
```python
# Expected structure:
# ## Module/Suite Name
# | CaseID | Title | Steps | Expected | Priority |
# |--------|-------|-------|----------|----------|
# | TC-001 | Login with valid credentials | 1. Open login page... | User is redirected to... | P0 |
```

**JSON:**
```json
{
  "suites": [
    {
      "name": "Login Module",
      "cases": [
        {
          "id": "TC-001",
          "title": "Valid login",
          "precondition": "User is not logged in",
          "steps": ["Open /login", "Enter username", "...", "Click Login"],
          "expected": "Redirected to dashboard"
        }
      ]
    }
  ]
}
```

### Output a structured representation:

For each test case, produce:
```python
{
    "suite": "Login Module",
    "id": "TC-001",
    "title": "Login with valid credentials",
    "priority": "P0",
    "preconditions": "User has a valid account, user is on login page",
    "test_data": {"username": "test_user", "password": "valid_pass"},
    "steps": [
        {"order": 1, "action": "navigate", "detail": "Open login page", "automatable": True, "playwright_approach": "page.goto('/login')"},
        {"order": 2, "action": "input", "detail": "Enter username 'test_user'", "automatable": True, "playwright_approach": "page.fill('[data-testid=username]', 'test_user')"},
        {"order": 3, "action": "click", "detail": "Click Login button", "automatable": True, "playwright_approach": "page.click('[data-testid=login-btn]')"},
        {"order": 4, "action": "verify", "detail": "Dashboard is displayed", "automatable": True, "playwright_approach": "await expect(page.locator('[data-testid=dashboard]')).toBeVisible()"}
    ]
}
```

## Step 3: Analyze Automatability

For each step, classify:

| Step Type | Automatable? | Playwright Equivalent |
|-----------|-------------|----------------------|
| Navigate to URL | ✅ Yes | `page.goto(url)` |
| Input text | ✅ Yes | `page.fill(selector, text)` or `page.locator(selector).fill(text)` |
| Click element | ✅ Yes | `page.click(selector)` |
| Select dropdown | ✅ Yes | `page.selectOption(selector, value)` |
| Check/uncheck | ✅ Yes | `page.check(selector)` / `page.uncheck(selector)` |
| Hover | ✅ Yes | `page.hover(selector)` |
| Scroll | ✅ Yes | `page.evaluate('window.scrollTo(0, document.body.scrollHeight)')` or `page.locator(selector).scrollIntoViewIfNeeded()` |
| Wait for element | ✅ Yes | `page.waitForSelector(selector)` or `page.locator(selector).waitFor()` |
| Verify text visible | ✅ Yes | `expect(page.locator(selector)).toHaveText(expected)` |
| Verify element exists | ✅ Yes | `expect(page.locator(selector)).toBeVisible()` |
| Verify URL | ✅ Yes | `expect(page).toHaveURL(expected)` |
| Upload file | ✅ Yes | `page.setInputFiles(selector, filePath)` |
| Download file | ✅ Yes | `page.waitForEvent('download')` then `download.saveAs(path)` |
| Switch tab/window | ✅ Yes | `page.waitForEvent('popup')` |
| Handle alert/dialog | ✅ Yes | `page.on('dialog', dialog => dialog.accept())` |
| **Visual check** (looks correct) | ⚠️ Partial | `expect(page).toHaveScreenshot()` — note: flaky |
| **Manual verification** (sensory) | ❌ No | Flag as "Needs manual check" |
| **Data validation in DB** | ⚠️ Partial | Use API/DB queries alongside UI test |
| **Exploratory** ("try various") | ❌ No | Split into specific test cases first |

### Handling non-automatable steps:
- Mark them as `automatable: false`
- Add a comment in the generated code: `// [MANUAL] Step 4: Verify the color matches design spec — cannot automate`
- Summarize all non-automatable steps in a report at the end

## Step 4: Generate Page Object Model (POM)

### Standard POM structure:

```
project/
├── pages/
│   ├── LoginPage.ts
│   ├── DashboardPage.ts
│   ├── ...
├── tests/
│   ├── login.spec.ts
│   ├── dashboard.spec.ts
│   ├── ...
├── fixtures/
│   └── test-data.json
├── utils/
│   ├── helpers.ts
│   └── constants.ts
├── playwright.config.ts
└── package.json
```

### Page Object Template (TypeScript):

```typescript
// pages/LoginPage.ts
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly usernameInput: Locator;
  readonly passwordInput: Locator;
  readonly loginButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.usernameInput = page.locator('[data-testid="username-input"]');
    this.passwordInput = page.locator('[data-testid="password-input"]');
    this.loginButton = page.locator('[data-testid="login-button"]');
    this.errorMessage = page.locator('[data-testid="error-message"]');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(username: string, password: string) {
    await this.usernameInput.fill(username);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }

  async getErrorMessage() {
    return this.errorMessage.textContent();
  }
}
```

### Page Object Template (Python):

```python
# pages/login_page.py
from playwright.sync_api import Page

class LoginPage:
    def __init__(self, page: Page):
        self.page = page
        self.username_input = page.locator('[data-testid="username-input"]')
        self.password_input = page.locator('[data-testid="password-input"]')
        self.login_button = page.locator('[data-testid="login-button"]')
        self.error_message = page.locator('[data-testid="error-message"]')

    def goto(self):
        self.page.goto("/login")

    def login(self, username: str, password: str):
        self.username_input.fill(username)
        self.password_input.fill(password)
        self.login_button.click()
```

### Locator Strategy Priority (best to worst):
1. **`data-testid`** — `page.getByTestId('login-button')` — most stable
2. **Role-based** — `page.getByRole('button', { name: 'Login' })` — accessible
3. **Label** — `page.getByLabel('Username')` — for form fields
4. **Placeholder** — `page.getByPlaceholder('Enter username')`
5. **Text** — `page.getByText('Login')`
6. **CSS selectors** — only when above aren't available
7. **XPath** — avoid unless absolutely necessary

### Determine page boundaries from test steps:
- Group test cases by module/suite → each module gets a spec file
- Extract navigational targets → each page/route gets a Page Object
- Shared components (header, sidebar, modal) → create component objects

## Step 5: Generate Test Specs

### Test spec template (TypeScript with Playwright Test):

```typescript
// tests/login.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { DashboardPage } from '../pages/DashboardPage';
import { testData } from '../fixtures/test-data.json';

test.describe('Login Module', () => {
  test('TC-001: Login with valid credentials @P0', async ({ page }) => {
    const loginPage = new LoginPage(page);
    const dashboardPage = new DashboardPage(page);

    await test.step('Navigate to login page', async () => {
      await loginPage.goto();
    });

    await test.step('Enter credentials and login', async () => {
      await loginPage.login(testData.validUser.username, testData.validUser.password);
    });

    await test.step('Verify successful login', async () => {
      await expect(dashboardPage.header).toBeVisible();
      await expect(page).toHaveURL(/.*dashboard/);
    });
  });

  test('TC-002: Login with invalid password shows error @P1', async ({ page }) => {
    const loginPage = new LoginPage(page);

    await test.step('Navigate to login page', async () => {
      await loginPage.goto();
    });

    await test.step('Enter invalid credentials', async () => {
      await loginPage.login('wrong_user', 'wrong_pass');
    });

    await test.step('Verify error message', async () => {
      await expect(loginPage.errorMessage).toBeVisible();
      await expect(loginPage.errorMessage).toHaveText('Invalid username or password');
    });
  });
});
```

### Test spec template (Python with pytest-playwright):

```python
# tests/test_login.py
import pytest
from pages.login_page import LoginPage
from pages.dashboard_page import DashboardPage

class TestLogin:
    @pytest.mark.p0
    def test_valid_login(self, page, test_data):
        login_page = LoginPage(page)
        dashboard_page = DashboardPage(page)

        with page.expect_navigation():
            login_page.goto()

        login_page.login(
            test_data['valid_user']['username'],
            test_data['valid_user']['password']
        )

        expect(dashboard_page.header).to_be_visible()
        expect(page).to_have_url(/.*dashboard/)

    @pytest.mark.p1
    def test_invalid_login_shows_error(self, page):
        login_page = LoginPage(page)
        login_page.goto()
        login_page.login('wrong_user', 'wrong_pass')

        expect(login_page.error_message).to_be_visible()
        expect(login_page.error_message).to_have_text('Invalid username or password')
```

### Step organization:
- Each manual test case → one `test()` block
- Each manual step → one `test.step()` block (TS) or `with page.expect_navigation()` / comment block (Python)
- Priority tags: `@P0`, `@P1`, `@P2`, `@P3` (P0 = smoke, P3 = edge case)
- Use tag-based test selection: `npx playwright test --grep @P0`

## Step 6: Review and Iterate

### Auto-review checklist before showing to user:
- [ ] Every manual test case has a corresponding `test()` block
- [ ] Every manual step maps to a Playwright action (or is flagged as manual)
- [ ] Page Objects are created for each distinct page/screen
- [ ] Locators use the best strategy (data-testid > role > label > text > CSS)
- [ ] Test data is extracted to fixtures (not hardcoded in tests)
- [ ] Preconditions are handled (setup/teardown in `beforeEach`/`afterEach`)
- [ ] Wait strategies are correct (auto-waiting > explicit waits)
- [ ] Assertions check actual behavior, not just "no error"
- [ ] Non-automatable steps are documented
- [ ] Tests follow existing project conventions (consistent with scanned codebase)

### Common pitfalls:
- ❌ **Hardcoded URLs** — use `baseURL` from config and relative paths
- ❌ **Excessive waits** — `page.waitForTimeout()` is a smell; prefer auto-waiting
- ❌ **Flaky selectors** — avoid index-based selectors: `div:nth-child(2) > span:first-child`
- ❌ **Missing error handling** — test should verify error states too, not just happy path
- ❌ **Overlapping test scope** — each test should test one scenario; avoid testing everything in one test
- ❌ **Forgotten clean-up** — test data created in setup must be cleaned up in teardown

## Step 7: Generate Summary Report

After conversion, present a clear summary to the user:

```
## 📋 Conversion Summary

**Source:** manual_test_cases.xlsx
**Framework:** Playwright (TypeScript)
**Total cases:** 45
**Fully automated:** 38 (84%)
**Partially automated:** 5 (11%) — see notes below
**Non-automatable (manual only):** 2 (5%) — see notes below

### ✅ Generated Files
  - pages/LoginPage.ts
  - pages/DashboardPage.ts
  - pages/OrderPage.ts
  - pages/...
  - tests/login.spec.ts
  - tests/dashboard.spec.ts
  - tests/order.spec.ts
  - tests/...
  - fixtures/test-data.json

### ⚠️ Partially Automated (needs manual attention)
| Test Case | Step | Reason |
|-----------|------|--------|
| TC-015 | Step 3: "Verify page looks correct" | Visual check — consider screenshot comparison |
| TC-023 | Step 2: "Check email notification" | Requires email service mock |

### 🔴 Non-Automatable Steps
| Test Case | Step | Recommendation |
|-----------|------|----------------|
| TC-031 | "Check the sound effect plays correctly" | Keep as manual check |
| TC-042 | "Verify the color under different monitors" | Manual visual check |

### 🚀 Run command:
  npx playwright test --grep @P0  (smoke tests)
  npx playwright test              (all tests)
```

## Scaffolding a New Project

If no existing Playwright project exists, scaffold:

```bash
# TypeScript
npm init playwright@latest my-tests
cd my-tests
mkdir -p pages fixtures utils
```

```bash
# Python
pip install pytest-playwright
playwright install
mkdir -p pages tests fixtures utils
```

### Standard `playwright.config.ts`:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [['html'], ['list']],
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
  ],
});
```

## Advanced Patterns

### Data-Driven Tests (when test data varies):
```typescript
const loginCases = [
  { username: 'admin@test.com', password: 'Admin123!', expected: 'Dashboard' },
  { username: 'user@test.com', password: 'User456!', expected: 'Home' },
];

for (const { username, password, expected } of loginCases) {
  test(`Login as ${username.split('@')[0]} @P1`, async ({ page }) => {
    // ...
  });
}
```

### API Integration (for setup/teardown):
```typescript
import { request } from '@playwright/test';

test.beforeAll(async () => {
  const apiContext = await request.newContext();
  await apiContext.post('/api/test-setup', { data: { /* ... */ } });
});
```

### Component-Level Tests (for reusable components):
```typescript
// Treat shared components as mini Page Objects
export class HeaderComponent {
  readonly userMenu: Locator;
  readonly logoutButton: Locator;

  constructor(page: Page) {
    this.userMenu = page.locator('[data-testid="user-menu"]');
    this.logoutButton = page.locator('[data-testid="logout-btn"]');
  }

  async logout() {
    await this.userMenu.click();
    await this.logoutButton.click();
  }
}
```

## Verification

Before marking complete:

- [ ] All manual test cases accounted for (no silent drops)
- [ ] Non-automatable steps clearly documented
- [ ] Page Objects follow project conventions
- [ ] Locators use best practice strategy
- [ ] Test data is externalized
- [ ] Test spec can be run with a single command
- [ ] Generated code is reviewed for correctness
- [ ] Summary report provided to user

## When Stuck

| Problem | Solution |
|---------|----------|
| Don't know the app URL structure | Ask user for base URL and route patterns |
| No data-testid in the app | Use role/label selectors, or ask user to add test IDs |
| Test cases are very vague | Ask user to clarify; break ambiguous steps into concrete actions |
| Complex preconditions | Suggest API setup (API calls are faster than UI flows for setup) |
| Auth tokens/cookies needed | Use `page.context().addCookies()` or `page.context().storageState()` |
