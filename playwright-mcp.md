# Playwright MCP for Engineers: Complete Step-by-Step Guide

*Based on: [Create Playwright Tests with GitHub Copilot and Playwright MCP](https://medium.com/@sonaldwivedi/create-playwright-tests-with-github-copilot-and-playwright-mcp-60c6951bed22)*

## 🎯 Overview

The Playwright MCP (Model Context Protocol) integration allows engineers to leverage GitHub Copilot's AI capabilities to generate, run, and manage Playwright tests seamlessly within VS Code. This guide provides a comprehensive walkthrough for setting up and using Playwright MCP for automated testing.

## 📋 Prerequisites

Before starting, ensure you have the following installed:

- **Node.js** (version 18 or higher)
- **Visual Studio Code**
- **GitHub Copilot extension** for VS Code
- **Git** for version control
- **NPM or Yarn** package manager

## 🚀 Step 1: Initial Project Setup

### 1.1 Create a New Project Directory

```bash
mkdir playwright-mcp-project
cd playwright-mcp-project
```

### 1.2 Initialize Node.js Project

```bash
npm init -y
```

This creates a `package.json` file with default settings.

### 1.3 Install Playwright

```bash
npm init playwright@latest
```

When prompted, choose:
- ✅ Do you want to use TypeScript or JavaScript? → **TypeScript**
- ✅ Where to put your end-to-end tests? → **tests**
- ✅ Add a GitHub Actions workflow? → **Yes**
- ✅ Install Playwright browsers? → **Yes**

## 🔧 Step 2: Install and Configure Playwright MCP

### 2.1 Install Playwright MCP Server

```bash
npm install @playwright/mcp
```

### 2.2 Create MCP Configuration

Create a `.vscode/mcp.json` file in your project root:

```json
{
  "servers": {
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "@playwright/mcp@latest"
      ],
      "env": {
        "PLAYWRIGHT_BROWSERS_PATH": "0"
      }
    }
  }
}
```

### 2.3 Configure VS Code Settings

Add to your `.vscode/settings.json`:

```json
{
  "mcp.servers": {
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["@playwright/mcp@latest"],
      "env": {
        "PLAYWRIGHT_BROWSERS_PATH": "0"
      }
    }
  },
  "github.copilot.enable": {
    "*": true,
    "yaml": true,
    "plaintext": true,
    "markdown": true
  }
}
```

## 🎮 Step 3: Configure GitHub Copilot with Playwright MCP

### 3.1 Enable MCP in VS Code

1. Open VS Code Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`)
2. Type "MCP: Reload Servers"
3. Press Enter to reload MCP configuration

### 3.2 Verify Playwright MCP Connection

1. Open GitHub Copilot Chat
2. Click the tools icon (🔧) in the chat interface
3. Verify that "Playwright" appears in the available tools list

![MCP Tools Available](assets/playwright-mcp-tools.png)
*Screenshot showing Playwright MCP tools available in GitHub Copilot*

## 📝 Step 4: Create Your First Playwright Test with Copilot

### 4.1 Generate Basic Test Structure

In GitHub Copilot Chat, use this prompt:

```
Using the Playwright MCP tools, create a test file that:
1. Tests the login functionality of a sample e-commerce website
2. Includes assertions for successful login
3. Uses page object model pattern
4. Includes proper error handling
```

### 4.2 Example Generated Test

Copilot will generate something like this in `tests/login.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Login Functionality', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('https://demo.testingsite.com');
  });

  test('should login successfully with valid credentials', async ({ page }) => {
    // Navigate to login page
    await page.click('[data-testid="login-button"]');
    
    // Fill in credentials
    await page.fill('[data-testid="email-input"]', 'test@example.com');
    await page.fill('[data-testid="password-input"]', 'password123');
    
    // Submit form
    await page.click('[data-testid="submit-button"]');
    
    // Assert successful login
    await expect(page).toHaveURL(/.*dashboard/);
    await expect(page.locator('[data-testid="welcome-message"]')).toBeVisible();
  });

  test('should show error for invalid credentials', async ({ page }) => {
    await page.click('[data-testid="login-button"]');
    
    await page.fill('[data-testid="email-input"]', 'invalid@example.com');
    await page.fill('[data-testid="password-input"]', 'wrongpassword');
    
    await page.click('[data-testid="submit-button"]');
    
    await expect(page.locator('[data-testid="error-message"]')).toBeVisible();
    await expect(page.locator('[data-testid="error-message"]')).toContainText('Invalid credentials');
  });
});
```

## 🏗️ Step 5: Advanced Test Generation with MCP

### 5.1 Generate Page Object Models

Ask Copilot:

```
Create a page object model for the login page using Playwright MCP tools
```

This generates `pages/LoginPage.ts`:

```typescript
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;
  readonly welcomeMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.locator('[data-testid="email-input"]');
    this.passwordInput = page.locator('[data-testid="password-input"]');
    this.submitButton = page.locator('[data-testid="submit-button"]');
    this.errorMessage = page.locator('[data-testid="error-message"]');
    this.welcomeMessage = page.locator('[data-testid="welcome-message"]');
  }

  async goto() {
    await this.page.goto('https://demo.testingsite.com/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async getErrorMessage() {
    return await this.errorMessage.textContent();
  }
}
```

### 5.2 Generate API Testing

Ask Copilot:

```
Using Playwright MCP, create API tests for user authentication endpoints
```

This creates `tests/api/auth.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Authentication API', () => {
  const baseURL = 'https://api.demo.testingsite.com';

  test('POST /login should return token for valid credentials', async ({ request }) => {
    const response = await request.post(`${baseURL}/login`, {
      data: {
        email: 'test@example.com',
        password: 'password123'
      }
    });

    expect(response.status()).toBe(200);
    
    const responseBody = await response.json();
    expect(responseBody).toHaveProperty('token');
    expect(responseBody).toHaveProperty('user');
    expect(responseBody.user.email).toBe('test@example.com');
  });

  test('POST /login should return 401 for invalid credentials', async ({ request }) => {
    const response = await request.post(`${baseURL}/login`, {
      data: {
        email: 'invalid@example.com',
        password: 'wrongpassword'
      }
    });

    expect(response.status()).toBe(401);
    
    const responseBody = await response.json();
    expect(responseBody).toHaveProperty('error');
    expect(responseBody.error).toContain('Invalid credentials');
  });
});
```

## 🏃‍♂️ Step 6: Running Tests with MCP Integration

### 6.1 Configure Playwright Config

Update `playwright.config.ts`:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html'],
    ['json', { outputFile: 'test-results.json' }]
  ],
  use: {
    baseURL: 'https://demo.testingsite.com',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
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
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
  ],

  webServer: {
    command: 'npm run start',
    url: 'http://127.0.0.1:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

### 6.2 Run Tests via Copilot

In GitHub Copilot Chat:

```
Run my Playwright tests and show me the results
```

Or use terminal commands:

```bash
# Run all tests
npx playwright test

# Run specific test file
npx playwright test tests/login.spec.ts

# Run tests in headed mode
npx playwright test --headed

# Run tests with UI mode
npx playwright test --ui
```

## 📊 Step 7: Test Reporting and Analysis

### 7.1 Generate Reports with Copilot

Ask Copilot:

```
Generate a comprehensive test report for my Playwright tests including screenshots and traces
```

### 7.2 View HTML Report

```bash
npx playwright show-report
```

![Playwright Test Report](assets/playwright-test-report.png)
*Screenshot of Playwright HTML test report*

### 7.3 Analyze Test Results

Ask Copilot to analyze your test results:

```
Analyze my test results and suggest improvements for failing tests
```

## 🔄 Step 8: Continuous Integration Setup

### 8.1 GitHub Actions Workflow

Copilot can generate `.github/workflows/playwright.yml`:

```yaml
name: Playwright Tests
on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: lts/*
    - name: Install dependencies
      run: npm ci
    - name: Install Playwright Browsers
      run: npx playwright install --with-deps
    - name: Run Playwright tests
      run: npx playwright test
    - uses: actions/upload-artifact@v4
      if: always()
      with:
        name: playwright-report
        path: playwright-report/
        retention-days: 30
```

## 🛠️ Step 9: Advanced MCP Features

### 9.1 Custom Test Generation

Use advanced prompts with Copilot:

```
Create a comprehensive e2e test suite that:
- Tests user registration flow
- Includes form validation testing
- Tests file upload functionality  
- Includes accessibility testing
- Uses data-driven testing with multiple test cases
```

### 9.2 Visual Testing

Ask Copilot to add visual regression tests:

```
Add visual regression testing to my existing tests using Playwright's screenshot comparison
```

Generated test:

```typescript
import { test, expect } from '@playwright/test';

test('visual regression test for homepage', async ({ page }) => {
  await page.goto('/');
  await page.waitForLoadState('networkidle');
  
  // Full page screenshot
  await expect(page).toHaveScreenshot('homepage-full.png', {
    fullPage: true
  });
  
  // Element screenshot
  await expect(page.locator('.hero-section')).toHaveScreenshot('hero-section.png');
});
```

### 9.3 Performance Testing

Generate performance tests:

```
Create performance tests that measure page load times and Core Web Vitals
```

## 🐛 Step 10: Debugging with MCP

### 10.1 Debug Tests with Copilot

When tests fail, ask Copilot:

```
My login test is failing. Help me debug the issue and suggest fixes.
```

### 10.2 Trace Viewer Integration

```bash
# Generate trace
npx playwright test --trace on

# View trace
npx playwright show-trace test-results/tests-login-chromium/trace.zip
```

![Playwright Trace Viewer](assets/playwright-trace-viewer.png)
*Screenshot of Playwright Trace Viewer interface*

## 📚 Step 11: Best Practices and Tips

### 11.1 Test Organization

Ask Copilot to help organize your tests:

```
Help me organize my test files using best practices for a large Playwright test suite
```

### 11.2 Test Data Management

Generate test data utilities:

```
Create a test data factory for managing user credentials and test data
```

### 11.3 Custom Assertions

Create custom assertions with Copilot:

```typescript
// Custom assertion example
import { expect as baseExpected } from '@playwright/test';

export const expect = baseExpected.extend({
  async toBeLoggedIn(page) {
    const welcomeMessage = page.locator('[data-testid="welcome-message"]');
    const userMenu = page.locator('[data-testid="user-menu"]');
    
    await baseExpected(welcomeMessage).toBeVisible();
    await baseExpected(userMenu).toBeVisible();
    
    return {
      message: () => 'User should be logged in',
      pass: true
    };
  }
});
```

## 🔧 Troubleshooting Common Issues

### Issue 1: MCP Server Not Starting

**Solution:**
```bash
# Reinstall MCP server
npm uninstall -g @modelcontextprotocol/server-playwright
npm install -g @modelcontextprotocol/server-playwright

# Reload VS Code MCP servers
# Command Palette > MCP: Reload Servers
```

### Issue 2: Copilot Not Using Playwright Tools

**Solution:**
1. Check `.vscode/mcp.json` configuration
2. Restart VS Code
3. Verify GitHub Copilot extension is enabled
4. Check that Playwright MCP appears in tools list

### Issue 3: Browser Launch Failures

**Solution:**
```bash
# Install browsers
npx playwright install

# Install system dependencies
npx playwright install-deps
```

## 📋 Quick Reference Commands

```bash
# Project setup
npm init playwright@latest
npm install -g @modelcontextprotocol/server-playwright

# Running tests
npx playwright test                    # Run all tests
npx playwright test --headed          # Run with browser visible
npx playwright test --ui              # Run in UI mode
npx playwright test --debug           # Run in debug mode

# Generating tests
npx playwright codegen               # Generate tests by recording

# Reports and debugging
npx playwright show-report           # View HTML report
npx playwright show-trace            # View trace files
```

## 🎯 Conclusion

The Playwright MCP integration with GitHub Copilot provides a powerful combination for automated testing. By following this guide, engineers can:

- Rapidly generate comprehensive test suites
- Leverage AI assistance for debugging and optimization
- Maintain high-quality test code with best practices
- Integrate seamlessly with CI/CD pipelines

The combination of Playwright's robust testing capabilities with MCP's protocol standardization and GitHub Copilot's AI assistance creates an efficient and productive testing workflow.

## 🔗 Additional Resources

- [Playwright Official Documentation](https://playwright.dev/)
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [GitHub Copilot Documentation](https://docs.github.com/copilot)
- [Playwright MCP Server Repository](https://github.com/modelcontextprotocol/servers/tree/main/src/playwright)

## 📝 License

This guide is provided under the MIT License. Feel free to adapt and share.

---

*Last updated: August 2025*
*Source: Based on [Create Playwright Tests with GitHub Copilot and Playwright MCP](https://medium.com/@sonaldwivedi/create-playwright-tests-with-github-copilot-and-playwright-mcp-60c6951bed22)*
