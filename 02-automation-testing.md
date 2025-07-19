
# Automation Testing 🤖

## What is Automation Testing?

Automation testing uses software tools, scripts, and frameworks to execute test cases automatically, comparing actual outcomes with expected results without manual intervention. It's essential for continuous integration/continuous deployment (CI/CD), regression testing, load testing, and scaling testing efforts across large applications and frequent releases.

Automation testing enables teams to achieve faster feedback loops, consistent test execution, improved test coverage, and efficient resource utilization while reducing human error and long-term testing costs.

## Key Concepts

### Types of Automation Testing

#### Functional Automation Testing
- **Unit Testing**: Testing individual code components in isolation
- **Integration Testing**: Testing component interactions and data flow
- **End-to-End (E2E) Testing**: Testing complete user workflows and business scenarios
- **API Testing**: Testing application programming interfaces and services
- **Regression Testing**: Automated re-testing of existing functionality after changes
- **Smoke Testing**: Automated basic functionality verification after builds

#### Non-Functional Automation Testing
- **Performance Testing**: Automated load, stress, and volume testing
- **Security Testing**: Automated vulnerability scanning and penetration testing
- **Compatibility Testing**: Automated cross-browser and cross-platform testing
- **Accessibility Testing**: Automated accessibility compliance verification

#### Specialized Automation Types
- **Data-Driven Testing**: Tests with external data sources and parameters
- **Keyword-Driven Testing**: Tests using reusable business keywords
- **Model-Based Testing**: Tests generated from application models
- **Visual Testing**: Automated UI appearance and layout validation

### Automation Frameworks Architecture

#### Test Pyramid Strategy
```
             /\
            /  \
           /    \
          /  E2E  \     <- Few, High-Level, Slow
         /________\
        /          \
       / Integration \   <- Some, Mid-Level, Medium Speed
      /______________\
     /                \
    /    Unit Tests     \  <- Many, Low-Level, Fast
   /____________________\

Framework Distribution:
- Unit Tests: 70% of test suite
- Integration Tests: 20% of test suite  
- E2E Tests: 10% of test suite
```

#### Automation Framework Types

**Linear/Record-Playback Framework**
```javascript
// Simple recorded script example
const { Builder, By, Key, until } = require('selenium-webdriver');

async function recordedLoginTest() {
    let driver = await new Builder().forBrowser('chrome').build();
    
    try {
        // Recorded steps
        await driver.get('https://example.com/login');
        await driver.findElement(By.id('username')).sendKeys('testuser');
        await driver.findElement(By.id('password')).sendKeys('password123');
        await driver.findElement(By.id('loginBtn')).click();
        await driver.wait(until.urlContains('dashboard'), 5000);
        
        console.log('Login test passed');
    } finally {
        await driver.quit();
    }
}
```

**Data-Driven Framework**
```javascript
// Data-driven testing with external data sources
const fs = require('fs');
const csv = require('csv-parser');

class DataDrivenTestFramework {
    constructor() {
        this.testData = [];
    }
    
    async loadTestData(filePath) {
        return new Promise((resolve) => {
            fs.createReadStream(filePath)
                .pipe(csv())
                .on('data', (row) => this.testData.push(row))
                .on('end', () => resolve(this.testData));
        });
    }
    
    async runLoginTests() {
        await this.loadTestData('./testdata/login_data.csv');
        
        for (const data of this.testData) {
            await this.executeLoginTest(
                data.username, 
                data.password, 
                data.expectedResult
            );
        }
    }
    
    async executeLoginTest(username, password, expected) {
        const driver = await new Builder().forBrowser('chrome').build();
        
        try {
            await driver.get('https://example.com/login');
            await driver.findElement(By.id('username')).sendKeys(username);
            await driver.findElement(By.id('password')).sendKeys(password);
            await driver.findElement(By.id('loginBtn')).click();
            
            const result = await this.validateResult(driver, expected);
            console.log(`Test with ${username}: ${result ? 'PASS' : 'FAIL'}`);
            
        } finally {
            await driver.quit();
        }
    }
}

// CSV test data (login_data.csv)
/*
username,password,expectedResult
valid@user.com,correctpass,success
invalid@user.com,wrongpass,failure
empty@field.com,,failure
special!@user.com,pass123!,success
*/
```

**Page Object Model (POM) Framework**
```javascript
// Page Object Model implementation
class LoginPage {
    constructor(driver) {
        this.driver = driver;
        this.url = 'https://example.com/login';
        
        // Element locators
        this.usernameInput = By.id('username');
        this.passwordInput = By.id('password');
        this.loginButton = By.id('loginBtn');
        this.errorMessage = By.className('error-message');
        this.loadingSpinner = By.className('loading-spinner');
    }
    
    async navigate() {
        await this.driver.get(this.url);
        await this.waitForPageLoad();
    }
    
    async waitForPageLoad() {
        await this.driver.wait(
            until.elementLocated(this.usernameInput), 
            10000
        );
    }
    
    async enterUsername(username) {
        const element = await this.driver.findElement(this.usernameInput);
        await element.clear();
        await element.sendKeys(username);
    }
    
    async enterPassword(password) {
        const element = await this.driver.findElement(this.passwordInput);
        await element.clear();
        await element.sendKeys(password);
    }
    
    async clickLogin() {
        await this.driver.findElement(this.loginButton).click();
        await this.waitForLoginProcessing();
    }
    
    async waitForLoginProcessing() {
        try {
            // Wait for loading spinner to appear and disappear
            await this.driver.wait(until.elementLocated(this.loadingSpinner), 2000);
            await this.driver.wait(until.stalenessOf(
                await this.driver.findElement(this.loadingSpinner)
            ), 10000);
        } catch (error) {
            // Spinner might not appear for fast responses
        }
    }
    
    async getErrorMessage() {
        try {
            const element = await this.driver.findElement(this.errorMessage);
            return await element.getText();
        } catch (error) {
            return null;
        }
    }
    
    async isLoginSuccessful() {
        try {
            await this.driver.wait(until.urlContains('dashboard'), 5000);
            return true;
        } catch (error) {
            return false;
        }
    }
    
    // Complete login workflow
    async login(username, password) {
        await this.navigate();
        await this.enterUsername(username);
        await this.enterPassword(password);
        await this.clickLogin();
        
        return {
            success: await this.isLoginSuccessful(),
            errorMessage: await this.getErrorMessage()
        };
    }
}

class DashboardPage {
    constructor(driver) {
        this.driver = driver;
        this.welcomeMessage = By.className('welcome-message');
        this.userProfile = By.id('user-profile');
        this.logoutButton = By.id('logout-btn');
        this.navigationMenu = By.className('nav-menu');
    }
    
    async isLoaded() {
        try {
            await this.driver.wait(until.elementLocated(this.welcomeMessage), 5000);
            return true;
        } catch (error) {
            return false;
        }
    }
    
    async getWelcomeText() {
        const element = await this.driver.findElement(this.welcomeMessage);
        return await element.getText();
    }
    
    async getUserProfileInfo() {
        const element = await this.driver.findElement(this.userProfile);
        return await element.getText();
    }
    
    async logout() {
        await this.driver.findElement(this.logoutButton).click();
        await this.driver.wait(until.urlContains('login'), 5000);
    }
}

// Test implementation using Page Objects
class LoginTestSuite {
    constructor() {
        this.driver = null;
        this.loginPage = null;
        this.dashboardPage = null;
    }
    
    async setup() {
        this.driver = await new Builder().forBrowser('chrome').build();
        this.loginPage = new LoginPage(this.driver);
        this.dashboardPage = new DashboardPage(this.driver);
    }
    
    async teardown() {
        if (this.driver) {
            await this.driver.quit();
        }
    }
    
    async testValidLogin() {
        const result = await this.loginPage.login('valid@user.com', 'password123');
        
        console.assert(result.success === true, 'Login should succeed with valid credentials');
        console.assert(await this.dashboardPage.isLoaded(), 'Dashboard should load after login');
        
        const welcomeText = await this.dashboardPage.getWelcomeText();
        console.assert(welcomeText.includes('Welcome'), 'Welcome message should be displayed');
    }
    
    async testInvalidLogin() {
        const result = await this.loginPage.login('invalid@user.com', 'wrongpass');
        
        console.assert(result.success === false, 'Login should fail with invalid credentials');
        console.assert(result.errorMessage !== null, 'Error message should be displayed');
        console.assert(result.errorMessage.includes('Invalid'), 'Error message should mention invalid credentials');
    }
    
    async runAllTests() {
        await this.setup();
        
        try {
            await this.testValidLogin();
            await this.testInvalidLogin();
            console.log('All login tests passed');
        } catch (error) {
            console.error('Test failed:', error);
        } finally {
            await this.teardown();
        }
    }
}
```

## Tools & Frameworks

### Popular Automation Frameworks

#### Selenium WebDriver (Multi-language support)
```python
# Python Selenium example with pytest
import pytest
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.options import Options

class TestEcommerceWorkflow:
    
    @pytest.fixture(scope="class")
    def driver(self):
        """Setup Chrome driver with options"""
        chrome_options = Options()
        chrome_options.add_argument("--headless")  # Run in headless mode
        chrome_options.add_argument("--no-sandbox")
        chrome_options.add_argument("--disable-dev-shm-usage")
        chrome_options.add_argument("--window-size=1920,1080")
        
        driver = webdriver.Chrome(options=chrome_options)
        driver.implicitly_wait(10)
        yield driver
        driver.quit()
    
    @pytest.fixture(scope="function")
    def login_user(self, driver):
        """Login before each test"""
        driver.get("https://demo-store.com/login")
        
        username = WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.ID, "username"))
        )
        password = driver.find_element(By.ID, "password")
        login_btn = driver.find_element(By.ID, "login-btn")
        
        username.send_keys("testuser@example.com")
        password.send_keys("securepassword123")
        login_btn.click()
        
        # Wait for dashboard to load
        WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.CLASS_NAME, "user-dashboard"))
        )
        
        yield driver
        
        # Logout after test
        logout_btn = driver.find_element(By.ID, "logout-btn")
        logout_btn.click()
    
    def test_product_search_and_filter(self, login_user):
        """Test product search with filters"""
        driver = login_user
        
        # Navigate to products page
        products_link = driver.find_element(By.LINK_TEXT, "Products")
        products_link.click()
        
        # Search for specific product
        search_box = WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.ID, "product-search"))
        )
        search_box.send_keys("laptop")
        
        search_btn = driver.find_element(By.ID, "search-btn")
        search_btn.click()
        
        # Wait for search results
        WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.CLASS_NAME, "search-results"))
        )
        
        # Apply price filter
        price_filter = driver.find_element(By.ID, "price-range-1000-2000")
        driver.execute_script("arguments[0].click();", price_filter)
        
        # Wait for filtered results
        WebDriverWait(driver, 10).until(
            EC.staleness_of(driver.find_element(By.CLASS_NAME, "loading-spinner"))
        )
        
        # Verify results
        products = driver.find_elements(By.CLASS_NAME, "product-card")
        assert len(products) > 0, "Search should return products"
        
        # Verify all products contain search term
        for product in products:
            product_title = product.find_element(By.CLASS_NAME, "product-title").text.lower()
            assert "laptop" in product_title, f"Product '{product_title}' should contain 'laptop'"
    
    def test_shopping_cart_workflow(self, login_user):
        """Test complete shopping cart workflow"""
        driver = login_user
        
        # Add product to cart
        driver.get("https://demo-store.com/products/laptop-pro-15")
        
        add_to_cart_btn = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.ID, "add-to-cart"))
        )
        add_to_cart_btn.click()
        
        # Verify cart update
        cart_count = WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.ID, "cart-count"))
        )
        assert cart_count.text == "1", "Cart should show 1 item"
        
        # Go to cart
        cart_icon = driver.find_element(By.ID, "cart-icon")
        cart_icon.click()
        
        # Verify product in cart
        cart_item = WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.CLASS_NAME, "cart-item"))
        )
        
        item_title = cart_item.find_element(By.CLASS_NAME, "item-title").text
        assert "Laptop Pro 15" in item_title, "Correct product should be in cart"
        
        # Update quantity
        quantity_input = cart_item.find_element(By.CLASS_NAME, "quantity-input")
        quantity_input.clear()
        quantity_input.send_keys("2")
        
        update_btn = cart_item.find_element(By.CLASS_NAME, "update-quantity")
        update_btn.click()
        
        # Verify quantity and total update
        WebDriverWait(driver, 10).until(
            lambda d: d.find_element(By.CLASS_NAME, "quantity-input").get_attribute("value") == "2"
        )
        
        # Proceed to checkout
        checkout_btn = driver.find_element(By.ID, "proceed-checkout")
        checkout_btn.click()
        
        # Verify checkout page loads
        WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.CLASS_NAME, "checkout-form"))
        )
        
        checkout_title = driver.find_element(By.TAG_NAME, "h1").text
        assert "Checkout" in checkout_title, "Should navigate to checkout page"

# Configuration file: pytest.ini
"""
[tool:pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = --verbose --tb=short --strict-markers
markers =
    smoke: Quick smoke tests
    regression: Full regression suite
    integration: Integration tests
    slow: Slow running tests
"""

# Run specific test categories
# pytest -m smoke  # Run only smoke tests
# pytest -m "not slow"  # Run all except slow tests
# pytest tests/test_login.py::TestLogin::test_valid_login  # Run specific test
```

#### Cypress (JavaScript-focused)
```javascript
// Cypress configuration (cypress.config.js)
const { defineConfig } = require('cypress')

module.exports = defineConfig({
  e2e: {
    baseUrl: 'https://demo-ecommerce.com',
    viewportWidth: 1280,
    viewportHeight: 720,
    defaultCommandTimeout: 10000,
    requestTimeout: 10000,
    responseTimeout: 10000,
    retries: {
      runMode: 2,
      openMode: 0
    },
    env: {
      testUser: 'testuser@example.com',
      testPassword: 'securepass123'
    },
    setupNodeEvents(on, config) {
      // implement node event listeners here
      return config
    }
  }
})

// Custom commands (cypress/support/commands.js)
Cypress.Commands.add('login', (username, password) => {
  cy.session([username, password], () => {
    cy.visit('/login')
    cy.get('[data-testid="username"]').type(username)
    cy.get('[data-testid="password"]').type(password)
    cy.get('[data-testid="login-button"]').click()
    cy.url().should('include', '/dashboard')
    cy.get('[data-testid="user-menu"]').should('be.visible')
  })
})

Cypress.Commands.add('addProductToCart', (productId) => {
  cy.visit(`/products/${productId}`)
  cy.get('[data-testid="add-to-cart"]').click()
  cy.get('[data-testid="cart-notification"]').should('contain', 'Added to cart')
})

Cypress.Commands.add('clearCart', () => {
  cy.visit('/cart')
  cy.get('body').then(($body) => {
    if ($body.find('[data-testid="cart-item"]').length > 0) {
      cy.get('[data-testid="clear-cart"]').click()
      cy.get('[data-testid="confirm-clear"]').click()
      cy.get('[data-testid="empty-cart-message"]').should('be.visible')
    }
  })
})

// Page Object Model in Cypress
class ProductPage {
  visit(productId) {
    cy.visit(`/products/${productId}`)
    return this
  }
  
  getProductTitle() {
    return cy.get('[data-testid="product-title"]')
  }
  
  getProductPrice() {
    return cy.get('[data-testid="product-price"]')
  }
  
  selectQuantity(quantity) {
    cy.get('[data-testid="quantity-select"]').select(quantity.toString())
    return this
  }
  
  selectVariant(variantName) {
    cy.get(`[data-testid="variant-${variantName}"]`).click()
    return this
  }
  
  addToCart() {
    cy.get('[data-testid="add-to-cart"]').click()
    cy.get('[data-testid="cart-notification"]').should('be.visible')
    return this
  }
  
  addToWishlist() {
    cy.get('[data-testid="add-to-wishlist"]').click()
    return this
  }
  
  viewProductImages() {
    return cy.get('[data-testid="product-image"]')
  }
  
  readProductReviews() {
    cy.get('[data-testid="reviews-tab"]').click()
    return cy.get('[data-testid="review-item"]')
  }
}

// Test specifications
describe('E-commerce Product Management', () => {
  const productPage = new ProductPage()
  
  beforeEach(() => {
    cy.login(Cypress.env('testUser'), Cypress.env('testPassword'))
    cy.clearCart()
  })
  
  it('should display product details correctly', () => {
    productPage.visit('laptop-pro-15')
    
    productPage.getProductTitle()
      .should('be.visible')
      .and('contain', 'Laptop Pro 15')
    
    productPage.getProductPrice()
      .should('be.visible')
      .and('match', /\$[\d,]+\.\d{2}/)
    
    productPage.viewProductImages()
      .should('have.length.greaterThan', 0)
      .first()
      .should('have.attr', 'src')
      .and('not.be.empty')
  })
  
  it('should handle product variants and quantities', () => {
    productPage.visit('smartphone-x10')
    
    // Test color variant selection
    productPage.selectVariant('black')
    cy.get('[data-testid="selected-variant"]').should('contain', 'Black')
    
    productPage.selectVariant('white')
    cy.get('[data-testid="selected-variant"]').should('contain', 'White')
    
    // Test quantity selection
    productPage.selectQuantity(3)
    cy.get('[data-testid="quantity-select"]').should('have.value', '3')
    
    // Add to cart with selected options
    productPage.addToCart()
    
    // Verify cart contents
    cy.get('[data-testid="cart-count"]').should('contain', '3')
    
    cy.visit('/cart')
    cy.get('[data-testid="cart-item"]').within(() => {
      cy.get('[data-testid="item-name"]').should('contain', 'Smartphone X10')
      cy.get('[data-testid="item-variant"]').should('contain', 'White')
      cy.get('[data-testid="item-quantity"]').should('contain', '3')
    })
  })
  
  it('should handle out of stock products', () => {
    productPage.visit('limited-edition-watch')
    
    cy.get('[data-testid="stock-status"]').then(($status) => {
      if ($status.text().includes('Out of Stock')) {
        cy.get('[data-testid="add-to-cart"]')
          .should('be.disabled')
          .and('contain', 'Out of Stock')
        
        cy.get('[data-testid="notify-when-available"]')
          .should('be.visible')
          .click()
        
        cy.get('[data-testid="email-notification"]')
          .should('be.visible')
          .type('customer@example.com')
        
        cy.get('[data-testid="subscribe-notification"]').click()
        
        cy.get('[data-testid="notification-success"]')
          .should('contain', 'will notify you when available')
      } else {
        cy.get('[data-testid="add-to-cart"]').should('not.be.disabled')
      }
    })
  })
  
  context('Product Reviews and Ratings', () => {
    it('should display and interact with product reviews', () => {
      productPage.visit('bestseller-headphones')
      
      // Check average rating
      cy.get('[data-testid="average-rating"]')
        .should('be.visible')
        .invoke('text')
        .should('match', /\d\.\d/)
      
      // Read reviews
      productPage.readProductReviews()
        .should('have.length.greaterThan', 0)
        .first()
        .within(() => {
          cy.get('[data-testid="reviewer-name"]').should('be.visible')
          cy.get('[data-testid="review-rating"]').should('be.visible')
          cy.get('[data-testid="review-text"]').should('not.be.empty')
          cy.get('[data-testid="review-date"]').should('be.visible')
        })
      
      // Filter reviews by rating
      cy.get('[data-testid="filter-5-stars"]').click()
      cy.get('[data-testid="review-item"]')
        .each(($review) => {
          cy.wrap($review)
            .find('[data-testid="review-rating"]')
            .should('contain', '5')
        })
    })
    
    it('should allow authenticated users to write reviews', () => {
      productPage.visit('wireless-keyboard')
      
      cy.get('[data-testid="write-review"]').click()
      
      cy.get('[data-testid="review-form"]').within(() => {
        // Select rating
        cy.get('[data-testid="rating-5"]').click()
        
        // Write review title
        cy.get('[data-testid="review-title"]')
          .type('Excellent wireless keyboard!')
        
        // Write review text
        cy.get('[data-testid="review-text"]')
          .type('This keyboard has excellent build quality and responsive keys. Highly recommended for both work and gaming.')
        
        // Add pros and cons
        cy.get('[data-testid="review-pros"]')
          .type('Great battery life, comfortable typing, reliable connection')
        
        cy.get('[data-testid="review-cons"]')
          .type('Price could be lower, no backlighting')
        
        // Submit review
        cy.get('[data-testid="submit-review"]').click()
      })
      
      cy.get('[data-testid="review-success"]')
        .should('contain', 'Review submitted successfully')
    })
  })
})

// API testing integration with Cypress
describe('API Integration Tests', () => {
  it('should sync cart between API and UI', () => {
    // Add item via API
    cy.request('POST', '/api/cart/items', {
      productId: 'laptop-pro-15',
      quantity: 2,
      variant: 'silver'
    }).then((response) => {
      expect(response.status).to.eq(200)
      expect(response.body.totalItems).to.eq(2)
    })
    
    // Verify UI reflects API changes
    cy.visit('/cart')
    cy.get('[data-testid="cart-item"]').should('have.length', 1)
    cy.get('[data-testid="item-quantity"]').should('contain', '2')
    
    // Modify via UI
    cy.get('[data-testid="quantity-input"]').clear().type('3')
    cy.get('[data-testid="update-quantity"]').click()
    
    // Verify API reflects UI changes
    cy.request('GET', '/api/cart').then((response) => {
      expect(response.body.items[0].quantity).to.eq(3)
      expect(response.body.totalItems).to.eq(3)
    })
  })
})
```

#### Playwright (Cross-browser automation)
```javascript
// Playwright configuration (playwright.config.js)
const { defineConfig, devices } = require('@playwright/test');

module.exports = defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html'],
    ['junit', { outputFile: 'test-results/junit.xml' }],
    ['json', { outputFile: 'test-results/results.json' }]
  ],
  use: {
    baseURL: 'https://demo-banking.com',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure'
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
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],
});

// Page Object Model for Banking Application
class BankingPage {
  constructor(page) {
    this.page = page;
    
    // Login elements
    this.usernameInput = page.locator('[data-testid="username"]');
    this.passwordInput = page.locator('[data-testid="password"]');
    this.loginButton = page.locator('[data-testid="login-button"]');
    this.mfaCodeInput = page.locator('[data-testid="mfa-code"]');
    this.verifyMfaButton = page.locator('[data-testid="verify-mfa"]');
    
    // Dashboard elements
    this.accountBalance = page.locator('[data-testid="account-balance"]');
    this.accountsList = page.locator('[data-testid="accounts-list"]');
    this.transactionHistory = page.locator('[data-testid="transaction-history"]');
    
    // Transfer elements
    this.transferButton = page.locator('[data-testid="transfer-funds"]');
    this.fromAccount = page.locator('[data-testid="from-account"]');
    this.toAccount = page.locator('[data-testid="to-account"]');
    this.transferAmount = page.locator('[data-testid="transfer-amount"]');
    this.transferDescription = page.locator('[data-testid="transfer-description"]');
    this.confirmTransfer = page.locator('[data-testid="confirm-transfer"]');
  }
  
  async login(username, password, mfaCode) {
    await this.page.goto('/login');
    await this.usernameInput.fill(username);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
    
    // Handle MFA if required
    if (mfaCode) {
      await this.mfaCodeInput.waitFor();
      await this.mfaCodeInput.fill(mfaCode);
      await this.verifyMfaButton.click();
    }
    
    // Wait for dashboard to load
    await this.accountBalance.waitFor();
  }
  
  async getAccountBalance(accountType = 'checking') {
    const accountCard = this.page.locator(`[data-testid="account-${accountType}"]`);
    const balance = await accountCard.locator('[data-testid="balance"]').textContent();
    return parseFloat(balance.replace(/[$,]/g, ''));
  }
  
  async transferFunds(fromAccount, toAccount, amount, description) {
    await this.transferButton.click();
    
    await this.fromAccount.selectOption(fromAccount);
    await this.toAccount.selectOption(toAccount);
    await this.transferAmount.fill(amount.toString());
    await this.transferDescription.fill(description);
    
    // Take screenshot before confirming
    await this.page.screenshot({ path: 'transfer-confirmation.png' });
    
    await this.confirmTransfer.click();
    
    // Wait for confirmation
    await this.page.locator('[data-testid="transfer-success"]').waitFor();
  }
  
  async getTransactionHistory(limit = 10) {
    const transactions = this.page.locator('[data-testid="transaction-row"]');
    const count = Math.min(await transactions.count(), limit);
    
    const history = [];
    for (let i = 0; i < count; i++) {
      const transaction = transactions.nth(i);
      history.push({
        date: await transaction.locator('[data-testid="transaction-date"]').textContent(),
        description: await transaction.locator('[data-testid="transaction-description"]').textContent(),
        amount: await transaction.locator('[data-testid="transaction-amount"]').textContent(),
        type: await transaction.locator('[data-testid="transaction-type"]').textContent()
      });
    }
    
    return history;
  }
}

// Banking application tests
const { test, expect } = require('@playwright/test');

test.describe('Banking Application', () => {
  let bankingPage;
  
  test.beforeEach(async ({ page }) => {
    bankingPage = new BankingPage(page);
  });
  
  test('should login and display account information', async ({ page }) => {
    await bankingPage.login('testuser@bank.com', 'securePass123!', '123456');
    
    // Verify dashboard loads
    await expect(page).toHaveURL(/.*dashboard/);
    await expect(bankingPage.accountBalance).toBeVisible();
    
    // Verify account information
    const checkingBalance = await bankingPage.getAccountBalance('checking');
    const savingsBalance = await bankingPage.getAccountBalance('savings');
    
    expect(checkingBalance).toBeGreaterThanOrEqual(0);
    expect(savingsBalance).toBeGreaterThanOrEqual(0);
    
    // Take screenshot for visual verification
    await page.screenshot({ path: 'dashboard-loaded.png', fullPage: true });
  });
  
  test('should transfer funds between accounts', async ({ page }) => {
    await bankingPage.login('testuser@bank.com', 'securePass123!', '123456');
    
    // Get initial balances
    const initialChecking = await bankingPage.getAccountBalance('checking');
    const initialSavings = await bankingPage.getAccountBalance('savings');
    
    // Transfer $100 from checking to savings
    const transferAmount = 100;
    await bankingPage.transferFunds(
      'checking',
      'savings',
      transferAmount,
      'Test transfer via automation'
    );
    
    // Verify success message
    await expect(page.locator('[data-testid="transfer-success"]'))
      .toContainText('Transfer completed successfully');
    
    // Verify balances updated
    await page.reload();
    await bankingPage.accountBalance.waitFor();
    
    const finalChecking = await bankingPage.getAccountBalance('checking');
    const finalSavings = await bankingPage.getAccountBalance('savings');
    
    expect(finalChecking).toBe(initialChecking - transferAmount);
    expect(finalSavings).toBe(initialSavings + transferAmount);
    
    // Verify transaction appears in history
    const transactions = await bankingPage.getTransactionHistory(5);
    const transferTransaction = transactions.find(t => 
      t.description.includes('Test transfer via automation')
    );
    
    expect(transferTransaction).toBeTruthy();
    expect(transferTransaction.amount).toContain('100.00');
  });
  
  test('should handle insufficient funds scenario', async ({ page }) => {
    await bankingPage.login('lowbalance@bank.com', 'securePass123!', '123456');
    
    const checkingBalance = await bankingPage.getAccountBalance('checking');
    const transferAmount = checkingBalance + 100; // More than available
    
    await bankingPage.transferButton.click();
    await bankingPage.fromAccount.selectOption('checking');
    await bankingPage.toAccount.selectOption('savings');
    await bankingPage.transferAmount.fill(transferAmount.toString());
    await bankingPage.transferDescription.fill('Test insufficient funds');
    await bankingPage.confirmTransfer.click();
    
    // Verify error message
    await expect(page.locator('[data-testid="transfer-error"]'))
      .toContainText('Insufficient funds');
    
    // Verify balances unchanged
    const finalBalance = await bankingPage.getAccountBalance('checking');
    expect(finalBalance).toBe(checkingBalance);
  });
  
  test('should validate security measures', async ({ page }) => {
    // Test session timeout
    await bankingPage.login('testuser@bank.com', 'securePass123!', '123456');
    
    // Simulate inactivity by waiting
    await page.waitForTimeout(30000); // 30 seconds
    
    // Try to perform sensitive operation
    await bankingPage.transferButton.click();
    
    // Should be redirected to login or see session timeout message
    await expect(page.locator('[data-testid="session-timeout"]')
      .or(page.locator('[data-testid="login-form"]'))).toBeVisible();
  });
  
  test('should work on mobile devices', async ({ page, isMobile }) => {
    test.skip(!isMobile, 'This test is only for mobile');
    
    await bankingPage.login('testuser@bank.com', 'securePass123!', '123456');
    
    // Verify mobile navigation
    await expect(page.locator('[data-testid="mobile-menu"]')).toBeVisible();
    
    // Test mobile-specific interactions
    await page.locator('[data-testid="mobile-menu"]').tap();
    await expect(page.locator('[data-testid="navigation-drawer"]')).toBeVisible();
    
    // Verify responsive design
    const accountCard = page.locator('[data-testid="account-checking"]');
    await expect(accountCard).toBeVisible();
    
    // Take mobile screenshot
    await page.screenshot({ path: 'mobile-dashboard.png', fullPage: true });
  });
});

// API testing integration
test.describe('API Integration', () => {
  test('should sync data between API and UI', async ({ page, request }) => {
    // Login via UI
    const bankingPage = new BankingPage(page);
    await bankingPage.login('testuser@bank.com', 'securePass123!', '123456');
    
    // Get auth token from browser storage
    const token = await page.evaluate(() => localStorage.getItem('authToken'));
    
    // Make API call to get account data
    const apiResponse = await request.get('/api/accounts', {
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      }
    });
    
    expect(apiResponse.ok()).toBeTruthy();
    const apiData = await apiResponse.json();
    
    // Compare with UI data
    const uiBalance = await bankingPage.getAccountBalance('checking');
    const apiBalance = apiData.accounts.find(acc => acc.type === 'checking').balance;
    
    expect(uiBalance).toBe(apiBalance);
  });
});
```

## Real-World Examples

### Complete E-commerce Test Automation Suite

#### Test Data Management
```javascript
// Test data factory for generating consistent test data
class TestDataFactory {
  static generateUser(overrides = {}) {
    const faker = require('faker');
    
    return {
      id: faker.random.uuid(),
      firstName: faker.name.firstName(),
      lastName: faker.name.lastName(),
      email: faker.internet.email(),
      password: 'TestPass123!',
      phone: faker.phone.phoneNumber(),
      address: {
        street: faker.address.streetAddress(),
        city: faker.address.city(),
        state: faker.address.state(),
        zipCode: faker.address.zipCode(),
        country: 'US'
      },
      creditCard: {
        number: '4111111111111111',
        expiry: '12/25',
        cvv: '123',
        name: `${faker.name.firstName()} ${faker.name.lastName()}`
      },
      ...overrides
    };
  }
  
  static generateProduct(overrides = {}) {
    const faker = require('faker');
    
    return {
      id: faker.random.uuid(),
      name: faker.commerce.productName(),
      description: faker.commerce.productDescription(),
      price: parseFloat(faker.commerce.price()),
      category: faker.commerce.department(),
      stock: faker.random.number({ min: 0, max: 100 }),
      images: [faker.image.imageUrl(), faker.image.imageUrl()],
      rating: faker.random.number({ min: 1, max: 5, precision: 0.1 }),
      reviews: faker.random.number({ min: 0, max: 500 }),
      ...overrides
    };
  }
  
  static generateOrder(user, products, overrides = {}) {
    const faker = require('faker');
    
    const orderItems = products.map(product => ({
      productId: product.id,
      quantity: faker.random.number({ min: 1, max: 5 }),
      price: product.price
    }));
    
    const subtotal = orderItems.reduce((sum, item) => sum + (item.price * item.quantity), 0);
    const tax = subtotal * 0.08;
    const shipping = subtotal > 50 ? 0 : 9.99;
    
    return {
      id: faker.random.uuid(),
      userId: user.id,
      items: orderItems,
      subtotal: subtotal,
      tax: tax,
      shipping: shipping,
      total: subtotal + tax + shipping,
      status: 'pending',
      shippingAddress: user.address,
      billingAddress: user.address,
      paymentMethod: {
        type: 'credit_card',
        last4: user.creditCard.number.slice(-4)
      },
      createdAt: new Date().toISOString(),
      ...overrides
    };
  }
}

// Environment configuration
class TestEnvironment {
  constructor() {
    this.config = {
      development: {
        baseUrl: 'http://localhost:3000',
        apiUrl: 'http://localhost:3001/api',
        dbUrl: 'mongodb://localhost:27017/ecommerce_test'
      },
      staging: {
        baseUrl: 'https://staging.ecommerce.com',
        apiUrl: 'https://api-staging.ecommerce.com',
        dbUrl: 'mongodb://staging-db:27017/ecommerce'
      },
      production: {
        baseUrl: 'https://ecommerce.com',
        apiUrl: 'https://api.ecommerce.com',
        dbUrl: 'mongodb://prod-db:27017/ecommerce'
      }
    };
  }
  
  get(environment = 'development') {
    return this.config[environment];
  }
}
```

#### Page Object Model Implementation
```javascript
// Base Page class with common functionality
class BasePage {
  constructor(page) {
    this.page = page;
    this.timeout = 30000;
  }
  
  async waitForElement(selector, options = {}) {
    return await this.page.waitForSelector(selector, { 
      timeout: this.timeout, 
      ...options 
    });
  }
  
  async clickElement(selector) {
    await this.waitForElement(selector);
    await this.page.click(selector);
  }
  
  async fillInput(selector, value) {
    await this.waitForElement(selector);
    await this.page.fill(selector, value);
  }
  
  async getText(selector) {
    await this.waitForElement(selector);
    return await this.page.textContent(selector);
  }
  
  async takeScreenshot(name) {
    await this.page.screenshot({ 
      path: `screenshots/${name}-${Date.now()}.png`,
      fullPage: true 
    });
  }
  
  async waitForNavigation(url) {
    await this.page.waitForURL(url, { timeout: this.timeout });
  }
}

class HomePage extends BasePage {
  constructor(page) {
    super(page);
    this.searchBox = '[data-testid="search-input"]';
    this.searchButton = '[data-testid="search-button"]';
    this.categoriesMenu = '[data-testid="categories-menu"]';
    this.featuredProducts = '[data-testid="featured-products"]';
    this.cartIcon = '[data-testid="cart-icon"]';
    this.userMenu = '[data-testid="user-menu"]';
  }
  
  async navigate() {
    await this.page.goto('/');
    await this.waitForElement(this.searchBox);
  }
  
  async searchProduct(searchTerm) {
    await this.fillInput(this.searchBox, searchTerm);
    await this.clickElement(this.searchButton);
    await this.waitForNavigation(/.*search.*/);
  }
  
  async selectCategory(categoryName) {
    await this.clickElement(this.categoriesMenu);
    await this.clickElement(`[data-testid="category-${categoryName}"]`);
    await this.waitForNavigation(/.*category.*/);
  }
  
  async getFeaturedProducts() {
    const products = await this.page.$$(`${this.featuredProducts} [data-testid="product-card"]`);
    return products;
  }
  
  async goToCart() {
    await this.clickElement(this.cartIcon);
    await this.waitForNavigation(/.*cart.*/);
  }
}

class ProductPage extends BasePage {
  constructor(page) {
    super(page);
    this.productTitle = '[data-testid="product-title"]';
    this.productPrice = '[data-testid="product-price"]';
    this.productDescription = '[data-testid="product-description"]';
    this.quantitySelector = '[data-testid="quantity-select"]';
    this.addToCartButton = '[data-testid="add-to-cart"]';
    this.addToWishlistButton = '[data-testid="add-to-wishlist"]';
    this.productImages = '[data-testid="product-image"]';
    this.reviewsSection = '[data-testid="reviews-section"]';
    this.stockStatus = '[data-testid="stock-status"]';
  }
  
  async visitProduct(productId) {
    await this.page.goto(`/products/${productId}`);
    await this.waitForElement(this.productTitle);
  }
  
  async getProductInfo() {
    return {
      title: await this.getText(this.productTitle),
      price: await this.getText(this.productPrice),
      description: await this.getText(this.productDescription),
      stockStatus: await this.getText(this.stockStatus)
    };
  }
  
  async selectQuantity(quantity) {
    await this.page.selectOption(this.quantitySelector, quantity.toString());
  }
  
  async addToCart(quantity = 1) {
    await this.selectQuantity(quantity);
    await this.clickElement(this.addToCartButton);
    
    // Wait for confirmation
    await this.waitForElement('[data-testid="cart-notification"]');
  }
  
  async getProductReviews() {
    const reviewElements = await this.page.$$(`${this.reviewsSection} [data-testid="review-item"]`);
    const reviews = [];
    
    for (const review of reviewElements) {
      reviews.push({
        rating: await review.$eval('[data-testid="review-rating"]', el => el.textContent),
        text: await review.$eval('[data-testid="review-text"]', el => el.textContent),
        author: await review.$eval('[data-testid="review-author"]', el => el.textContent),
        date: await review.$eval('[data-testid="review-date"]', el => el.textContent)
      });
    }
    
    return reviews;
  }
}

class CheckoutPage extends BasePage {
  constructor(page) {
    super(page);
    this.shippingForm = '[data-testid="shipping-form"]';
    this.billingForm = '[data-testid="billing-form"]';
    this.paymentForm = '[data-testid="payment-form"]';
    this.orderSummary = '[data-testid="order-summary"]';
    this.placeOrderButton = '[data-testid="place-order"]';
    this.orderConfirmation = '[data-testid="order-confirmation"]';
  }
  
  async fillShippingInfo(shippingData) {
    await this.fillInput('[data-testid="shipping-first-name"]', shippingData.firstName);
    await this.fillInput('[data-testid="shipping-last-name"]', shippingData.lastName);
    await this.fillInput('[data-testid="shipping-address"]', shippingData.address.street);
    await this.fillInput('[data-testid="shipping-city"]', shippingData.address.city);
    await this.fillInput('[data-testid="shipping-state"]', shippingData.address.state);
    await this.fillInput('[data-testid="shipping-zip"]', shippingData.address.zipCode);
  }
  
  async fillPaymentInfo(paymentData) {
    await this.fillInput('[data-testid="card-number"]', paymentData.number);
    await this.fillInput('[data-testid="card-expiry"]', paymentData.expiry);
    await this.fillInput('[data-testid="card-cvv"]', paymentData.cvv);
    await this.fillInput('[data-testid="card-name"]', paymentData.name);
  }
  
  async getOrderSummary() {
    return {
      subtotal: await this.getText('[data-testid="order-subtotal"]'),
      tax: await this.getText('[data-testid="order-tax"]'),
      shipping: await this.getText('[data-testid="order-shipping"]'),
      total: await this.getText('[data-testid="order-total"]')
    };
  }
  
  async completeOrder() {
    await this.clickElement(this.placeOrderButton);
    await this.waitForElement(this.orderConfirmation);
    
    return {
      orderNumber: await this.getText('[data-testid="order-number"]'),
      confirmationMessage: await this.getText('[data-testid="confirmation-message"]')
    };
  }
}
```

#### Comprehensive Test Suite
```javascript
// Complete end-to-end test suite
const { test, expect } = require('@playwright/test');

test.describe('E-commerce Application - Complete User Journey', () => {
  let homePage, productPage, cartPage, checkoutPage;
  let testUser, testProducts;
  
  test.beforeEach(async ({ page }) => {
    // Initialize page objects
    homePage = new HomePage(page);
    productPage = new ProductPage(page);
    cartPage = new CartPage(page);
    checkoutPage = new CheckoutPage(page);
    
    // Generate test data
    testUser = TestDataFactory.generateUser();
    testProducts = [
      TestDataFactory.generateProduct({ name: 'Test Laptop', price: 999.99 }),
      TestDataFactory.generateProduct({ name: 'Test Mouse', price: 29.99 })
    ];
  });
  
  test('Complete purchase journey - Guest checkout', async ({ page }) => {
    // Step 1: Navigate to homepage
    await homePage.navigate();
    await expect(page).toHaveTitle(/E-commerce Store/);
    
    // Step 2: Search for product
    await homePage.searchProduct('laptop');
    
    // Step 3: Select product from search results
    await page.click('[data-testid="product-card"]:first-child');
    
    // Step 4: Add product to cart
    const productInfo = await productPage.getProductInfo();
    await productPage.addToCart(2);
    
    // Verify cart notification
    await expect(page.locator('[data-testid="cart-notification"]')).toContainText('Added to cart');
    
    // Step 5: Go to cart
    await homePage.goToCart();
    
    // Step 6: Verify cart contents
    const cartItems = await cartPage.getCartItems();
    expect(cartItems).toHaveLength(1);
    expect(cartItems[0].quantity).toBe('2');
    
    // Step 7: Proceed to checkout
    await cartPage.proceedToCheckout();
    
    // Step 8: Fill checkout information
    await checkoutPage.fillShippingInfo(testUser);
    await checkoutPage.fillPaymentInfo(testUser.creditCard);
    
    // Step 9: Review order summary
    const orderSummary = await checkoutPage.getOrderSummary();
    expect(parseFloat(orderSummary.subtotal.replace('$', ''))).toBeGreaterThan(0);
    
    // Step 10: Place order
    const orderConfirmation = await checkoutPage.completeOrder();
    expect(orderConfirmation.orderNumber).toMatch(/^ORD-\d+$/);
    
    // Step 11: Verify confirmation page
    await expect(page.locator('[data-testid="order-confirmation"]')).toBeVisible();
    await page.screenshot({ path: 'order-confirmation.png' });
  });
  
  test('Shopping cart functionality', async ({ page }) => {
    // Add multiple products to cart
    await productPage.visitProduct('laptop-pro-15');
    await productPage.addToCart(1);
    
    await productPage.visitProduct('wireless-mouse');
    await productPage.addToCart(2);
    
    // Go to cart and verify items
    await homePage.goToCart();
    
    const cartItems = await cartPage.getCartItems();
    expect(cartItems).toHaveLength(2);
    
    // Update quantity
    await cartPage.updateItemQuantity(0, 3);
    
    // Remove item
    await cartPage.removeItem(1);
    
    const updatedItems = await cartPage.getCartItems();
    expect(updatedItems).toHaveLength(1);
    expect(updatedItems[0].quantity).toBe('3');
    
    // Verify total calculation
    const cartTotal = await cartPage.getCartTotal();
    expect(parseFloat(cartTotal.replace('$', ''))).toBeGreaterThan(0);
  });
  
  test('Product search and filtering', async ({ page }) => {
    await homePage.navigate();
    
    // Test search functionality
    await homePage.searchProduct('laptop');
    
    // Verify search results
    const searchResults = await page.$$('[data-testid="product-card"]');
    expect(searchResults.length).toBeGreaterThan(0);
    
    // Apply price filter
    await page.click('[data-testid="price-filter-500-1000"]');
    await page.waitForLoadState('networkidle');
    
    // Verify filtered results
    const filteredResults = await page.$$('[data-testid="product-card"]');
    
    for (const product of filteredResults) {
      const priceText = await product.$eval('[data-testid="product-price"]', el => el.textContent);
      const price = parseFloat(priceText.replace('$', ''));
      expect(price).toBeGreaterThanOrEqual(500);
      expect(price).toBeLessThanOrEqual(1000);
    }
    
    // Apply category filter
    await page.selectOption('[data-testid="category-filter"]', 'electronics');
    await page.waitForLoadState('networkidle');
    
    // Sort by price
    await page.selectOption('[data-testid="sort-by"]', 'price-asc');
    await page.waitForLoadState('networkidle');
    
    // Verify sorting
    const sortedPrices = await page.$$eval('[data-testid="product-price"]', 
      elements => elements.map(el => parseFloat(el.textContent.replace('$', '')))
    );
    
    for (let i = 1; i < sortedPrices.length; i++) {
      expect(sortedPrices[i]).toBeGreaterThanOrEqual(sortedPrices[i - 1]);
    }
  });
  
  test('User account functionality', async ({ page }) => {
    // Register new user
    await page.goto('/register');
    await page.fill('[data-testid="register-email"]', testUser.email);
    await page.fill('[data-testid="register-password"]', testUser.password);
    await page.fill('[data-testid="register-confirm-password"]', testUser.password);
    await page.fill('[data-testid="register-first-name"]', testUser.firstName);
    await page.fill('[data-testid="register-last-name"]', testUser.lastName);
    await page.click('[data-testid="register-button"]');
    
    // Verify registration success
    await expect(page).toHaveURL(/.*dashboard/);
    
    // Update profile information
    await page.click('[data-testid="profile-menu"]');
    await page.click('[data-testid="edit-profile"]');
    
    await page.fill('[data-testid="phone-number"]', testUser.phone);
    await page.click('[data-testid="save-profile"]');
    
    // Verify profile updated
    await expect(page.locator('[data-testid="success-message"]')).toContainText('Profile updated');
    
    // Add shipping address
    await page.click('[data-testid="manage-addresses"]');
    await page.click('[data-testid="add-address"]');
    
    await page.fill('[data-testid="address-street"]', testUser.address.street);
    await page.fill('[data-testid="address-city"]', testUser.address.city);
    await page.fill('[data-testid="address-state"]', testUser.address.state);
    await page.fill('[data-testid="address-zip"]', testUser.address.zipCode);
    await page.click('[data-testid="save-address"]');
    
    // Verify address saved
    await expect(page.locator('[data-testid="address-list"]')).toContainText(testUser.address.street);
    
    // Logout
    await page.click('[data-testid="logout-button"]');
    await expect(page).toHaveURL(/.*login/);
  });
  
  test('Error handling and validation', async ({ page }) => {
    // Test form validation
    await page.goto('/login');
    await page.click('[data-testid="login-button"]');
    
    // Verify validation messages
    await expect(page.locator('[data-testid="email-error"]')).toContainText('Email is required');
    await expect(page.locator('[data-testid="password-error"]')).toContainText('Password is required');
    
    // Test invalid email format
    await page.fill('[data-testid="login-email"]', 'invalid-email');
    await page.click('[data-testid="login-button"]');
    await expect(page.locator('[data-testid="email-error"]')).toContainText('Invalid email format');
    
    // Test network error handling
    await page.route('**/api/login', route => route.abort());
    
    await page.fill('[data-testid="login-email"]', 'test@example.com');
    await page.fill('[data-testid="login-password"]', 'password123');
    await page.click('[data-testid="login-button"]');
    
    await expect(page.locator('[data-testid="network-error"]')).toBeVisible();
  });
});

// Performance testing integration
test.describe('Performance Tests', () => {
  test('Page load performance', async ({ page }) => {
    // Measure homepage load time
    const startTime = Date.now();
    await page.goto('/');
    await page.waitForLoadState('networkidle');
    const loadTime = Date.now() - startTime;
    
    expect(loadTime).toBeLessThan(3000); // 3 seconds
    
    // Measure search performance
    const searchStartTime = Date.now();
    await page.fill('[data-testid="search-input"]', 'laptop');
    await page.click('[data-testid="search-button"]');
    await page.waitForLoadState('networkidle');
    const searchTime = Date.now() - searchStartTime;
    
    expect(searchTime).toBeLessThan(2000); // 2 seconds
  });
});
```

## Resume Showcase Tips

### Professional Automation Testing Experience

#### Strong Automation Background
```
✅ Excellent Examples:
• "Developed comprehensive automation framework using Selenium WebDriver and TestNG, increasing test coverage from 30% to 85% and reducing regression testing time by 70%"
• "Implemented CI/CD pipeline integration with Jenkins and GitHub Actions, enabling automated test execution on every code commit with 95% success rate"
• "Led automation strategy for microservices architecture, creating API test suites with 500+ test cases achieving 99.5% reliability"
• "Designed data-driven testing framework processing 10,000+ test scenarios, improving defect detection rate by 45%"

✅ Technical Skills to Highlight:
• Programming languages (Java, Python, JavaScript, C#)
• Automation frameworks (Selenium, Cypress, Playwright, TestNG, pytest)
• CI/CD tools (Jenkins, GitHub Actions, Azure DevOps, GitLab CI)
• API testing (REST Assured, Postman, Insomnia)
• Cloud platforms (AWS, Azure, GCP)
• Version control (Git, SVN)
• Database testing (SQL, NoSQL)
```

#### Framework Development Experience
```
Resume Achievement Examples:
• "Architected reusable automation framework reducing test development time by 60%"
• "Implemented Page Object Model pattern improving test maintainability across 50+ test classes"
• "Created custom reporting dashboard providing real-time test execution insights to stakeholders"
• "Established automation best practices and coding standards adopted across 5 development teams"
```

### Skills Progression Matrix
```
Junior Automation Engineer (0-2 years):
- Basic Selenium WebDriver
- Simple test script creation
- Test case execution and reporting
- Version control basics

Mid-Level Automation Engineer (2-5 years):
- Framework design and implementation
- CI/CD integration
- API and database testing
- Performance testing basics
- Team collaboration and mentoring

Senior Automation Engineer (5+ years):
- Automation strategy and architecture
- Cross-platform testing solutions
- Advanced framework patterns
- Team leadership and training
- Tool evaluation and selection
```

## Learning Path

### Beginner Level (0-6 months)

#### Month 1-2: Programming Fundamentals
```
Programming Language Selection:
Choose based on team/project needs:
- Java: Enterprise applications, strong typing
- Python: Rapid development, extensive libraries
- JavaScript: Web applications, Node.js ecosystem
- C#: Microsoft technologies, .NET applications

Week 1-4: Basic Programming
Daily Practice (2-3 hours):
1. Variables, data types, operators
2. Control structures (if/else, loops)
3. Functions and methods
4. Object-oriented programming basics
5. Error handling and debugging

Practical Exercises:
- Build simple calculator
- Create basic data processing scripts
- Practice with collections and arrays
- Implement basic algorithms

Resources:
- Codecademy, freeCodeCamp
- "Automate the Boring Stuff with Python"
- Java/JavaScript tutorials
- Practice on HackerRank, LeetCode
```

#### Month 3-4: Selenium Basics
```
Selenium WebDriver Setup:
1. Install development environment
2. Set up browser drivers
3. Create first test script
4. Understand element locators

Basic Selenium Operations:
// Java example
public class FirstTest {
    WebDriver driver;
    
    @BeforeMethod
    public void setup() {
        System.setProperty("webdriver.chrome.driver", "chromedriver.exe");
        driver = new ChromeDriver();
        driver.manage().timeouts().implicitlyWait(10, TimeUnit.SECONDS);
    }
    
    @Test
    public void testLogin() {
        driver.get("https://example.com/login");
        
        WebElement username = driver.findElement(By.id("username"));
        WebElement password = driver.findElement(By.id("password"));
        WebElement loginBtn = driver.findElement(By.xpath("//button[@type='submit']"));
        
        username.sendKeys("testuser");
        password.sendKeys("password123");
        loginBtn.click();
        
        WebElement dashboard = driver.findElement(By.className("dashboard"));
        Assert.assertTrue(dashboard.isDisplayed());
    }
    
    @AfterMethod
    public void teardown() {
        driver.quit();
    }
}

Weekly Practice:
Week 1: Element identification and interaction
Week 2: Waits and synchronization
Week 3: Handling different element types
Week 4: Basic assertions and validations
```

#### Month 5-6: Framework Introduction
```
Testing Framework Setup:
- TestNG (Java) or pytest (Python)
- Basic test organization
- Test data management
- Simple reporting

Project: Login Test Suite
Objective: Create comprehensive login testing
Features:
- Valid/invalid credentials testing
- Form validation testing
- Session management testing
- Cross-browser testing

Deliverables:
- 20+ automated test cases
- Basic test framework structure
- Test execution reports
- Documentation
```

### Intermediate Level (6-18 months)

#### Month 7-9: Advanced Framework Patterns
```
Page Object Model Implementation:
// Advanced POM with Page Factory
public class LoginPage {
    private WebDriver driver;
    
    @FindBy(id = "username")
    private WebElement usernameField;
    
    @FindBy(id = "password")
    private WebElement passwordField;
    
    @FindBy(xpath = "//button[@type='submit']")
    private WebElement loginButton;
    
    @FindBy(className = "error-message")
    private WebElement errorMessage;
    
    public LoginPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }
    
    public void enterUsername(String username) {
        usernameField.clear();
        usernameField.sendKeys(username);
    }
    
    public void enterPassword(String password) {
        passwordField.clear();
        passwordField.sendKeys(password);
    }
    
    public void clickLogin() {
        loginButton.click();
    }
    
    public String getErrorMessage() {
        return errorMessage.getText();
    }
    
    public DashboardPage login(String username, String password) {
        enterUsername(username);
        enterPassword(password);
        clickLogin();
        return new DashboardPage(driver);
    }
}

Data-Driven Testing:
@DataProvider
public Object[][] loginData() {
    return new Object[][] {
        {"valid@user.com", "password123", true},
        {"invalid@user.com", "wrongpass", false},
        {"", "password123", false},
        {"valid@user.com", "", false}
    };
}

@Test(dataProvider = "loginData")
public void testLogin(String username, String password, boolean shouldSucceed) {
    LoginPage loginPage = new LoginPage(driver);
    loginPage.enterUsername(username);
    loginPage.enterPassword(password);
    loginPage.clickLogin();
    
    if (shouldSucceed) {
        Assert.assertTrue(driver.getCurrentUrl().contains("dashboard"));
    } else {
        Assert.assertTrue(loginPage.getErrorMessage().length() > 0);
    }
}
```

#### Month 10-12: CI/CD Integration
```
Jenkins Pipeline Setup:
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/company/automation-tests.git'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Test') {
            parallel {
                stage('Chrome Tests') {
                    steps {
                        sh 'mvn test -Dbrowser=chrome -Dsuite=smoke'
                    }
                }
                stage('Firefox Tests') {
                    steps {
                        sh 'mvn test -Dbrowser=firefox -Dsuite=smoke'
                    }
                }
            }
        }
        
        stage('Report') {
            steps {
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'test-output',
                    reportFiles: 'index.html',
                    reportName: 'Test Report'
                ])
            }
        }
    }
    
    post {
        always {
            archiveArtifacts artifacts: 'screenshots/*.png', fingerprint: true
            publishTestResults testResultsPattern: 'test-output/testng-results.xml'
        }
        failure {
            emailext (
                subject: "Test Execution Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                body: "Test execution failed. Please check the build for details.",
                to: "team@company.com"
            )
        }
    }
}

GitHub Actions Workflow:
name: Automated Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        browser: [chrome, firefox]
        suite: [smoke, regression]
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Set up JDK 11
      uses: actions/setup-java@v2
      with:
        java-version: '11'
        distribution: 'adopt'
    
    - name: Cache Maven dependencies
      uses: actions/cache@v2
      with:
        path: ~/.m2
        key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
    
    - name: Run tests
      run: mvn test -Dbrowser=${{ matrix.browser }} -Dsuite=${{ matrix.suite }}
    
    - name: Upload test results
      uses: actions/upload-artifact@v2
      if: always()
      with:
        name: test-results-${{ matrix.browser }}-${{ matrix.suite }}
        path: |
          test-output/
          screenshots/
```

#### Month 13-15: API and Database Testing
```
REST API Testing Framework:
public class APITestBase {
    protected RequestSpecification requestSpec;
    protected ResponseSpecification responseSpec;
    
    @BeforeClass
    public void setupAPI() {
        RestAssured.baseURI = "https://api.example.com";
        
        requestSpec = new RequestSpecBuilder()
            .setContentType(ContentType.JSON)
            .addHeader("Authorization", "Bearer " + getAuthToken())
            .build();
            
        responseSpec = new ResponseSpecBuilder()
            .expectResponseTime(lessThan(5000L))
            .build();
    }
    
    protected String getAuthToken() {
        return given()
            .contentType(ContentType.JSON)
            .body("{'username': 'api_user', 'password': 'api_pass'}")
        .when()
            .post("/auth/login")
        .then()
            .statusCode(200)
            .extract()
            .path("token");
    }
}

public class UserAPITests extends APITestBase {
    
    @Test
    public void testCreateUser() {
        User newUser = User.builder()
            .firstName("John")
            .lastName("Doe")
            .email("john.doe@example.com")
            .build();
            
        ValidatableResponse response = given(requestSpec)
            .body(newUser)
        .when()
            .post("/users")
        .then(responseSpec)
            .statusCode(201)
            .body("id", notNullValue())
            .body("firstName", equalTo("John"))
            .body("email", equalTo("john.doe@example.com"));
            
        String userId = response.extract().path("id");
        createdUserIds.add(userId); // For cleanup
    }
    
    @Test
    public void testGetUser() {
        given(requestSpec)
        .when()
            .get("/users/123")
        .then(responseSpec)
            .statusCode(200)
            .body("id", equalTo(123))
            .body("firstName", notNullValue())
            .body("email", matchesPattern("^[^@]+@[^@]+\\.[^@]+$"));
    }
    
    @Test
    public void testUpdateUser() {
        Map<String, Object> updates = new HashMap<>();
        updates.put("firstName", "Jane");
        updates.put("lastName", "Smith");
        
        given(requestSpec)
            .body(updates)
        .when()
            .put("/users/123")
        .then(responseSpec)
            .statusCode(200)
            .body("firstName", equalTo("Jane"))
            .body("lastName", equalTo("Smith"));
    }
}

Database Testing Integration:
public class DatabaseTestUtils {
    private static final String DB_URL = "jdbc:mysql://localhost:3306/testdb";
    private static final String USERNAME = "testuser";
    private static final String PASSWORD = "testpass";
    
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(DB_URL, USERNAME, PASSWORD);
    }
    
    public static Map<String, Object> executeQuery(String query, Object... params) {
        try (Connection conn = getConnection();
             PreparedStatement stmt = conn.prepareStatement(query)) {
            
            for (int i = 0; i < params.length; i++) {
                stmt.setObject(i + 1, params[i]);
            }
            
            ResultSet rs = stmt.executeQuery();
            if (rs.next()) {
                Map<String, Object> result = new HashMap<>();
                ResultSetMetaData metaData = rs.getMetaData();
                
                for (int i = 1; i <= metaData.getColumnCount(); i++) {
                    result.put(metaData.getColumnName(i), rs.getObject(i));
                }
                
                return result;
            }
        } catch (SQLException e) {
            throw new RuntimeException("Database query failed", e);
        }
        
        return null;
    }
    
    public static void cleanupTestData(String table, String condition) {
        String query = "DELETE FROM " + table + " WHERE " + condition;
        try (Connection conn = getConnection();
             Statement stmt = conn.createStatement()) {
            stmt.executeUpdate(query);
        } catch (SQLException e) {
            throw new RuntimeException("Cleanup failed", e);
        }
    }
}

@Test
public void testUserCreationPersistence() {
    // Create user via API
    User newUser = createUserViaAPI();
    
    // Verify in database
    Map<String, Object> dbUser = DatabaseTestUtils.executeQuery(
        "SELECT * FROM users WHERE email = ?", 
        newUser.getEmail()
    );
    
    Assert.assertNotNull(dbUser);
    Assert.assertEquals(newUser.getFirstName(), dbUser.get("first_name"));
    Assert.assertEquals(newUser.getLastName(), dbUser.get("last_name"));
    Assert.assertEquals(newUser.getEmail(), dbUser.get("email"));
}
```

#### Month 16-18: Advanced Topics
```
Cross-Browser Testing with Selenium Grid:
// Grid configuration
public class BrowserFactory {
    public static WebDriver createDriver(String browserName, String gridUrl) {
        DesiredCapabilities capabilities;
        
        switch (browserName.toLowerCase()) {
            case "chrome":
                capabilities = DesiredCapabilities.chrome();
                ChromeOptions chromeOptions = new ChromeOptions();
                chromeOptions.addArguments("--headless");
                chromeOptions.addArguments("--no-sandbox");
                capabilities.merge(chromeOptions);
                break;
                
            case "firefox":
                capabilities = DesiredCapabilities.firefox();
                FirefoxOptions firefoxOptions = new FirefoxOptions();
                firefoxOptions.addArguments("--headless");
                capabilities.merge(firefoxOptions);
                break;
                
            default:
                throw new IllegalArgumentException("Browser not supported: " + browserName);
        }
        
        try {
            return new RemoteWebDriver(new URL(gridUrl), capabilities);
        } catch (MalformedURLException e) {
            throw new RuntimeException("Invalid Grid URL", e);
        }
    }
}

Performance Testing Integration:
public class PerformanceTestListener implements ITestListener {
    
    @Override
    public void onTestStart(ITestResult result) {
        startPerformanceMonitoring();
    }
    
    @Override
    public void onTestSuccess(ITestResult result) {
        recordPerformanceMetrics(result);
    }
    
    private void startPerformanceMonitoring() {
        // Start monitoring CPU, memory, network
    }
    
    private void recordPerformanceMetrics(ITestResult result) {
        // Record and analyze performance data
        // Fail test if performance thresholds exceeded
    }
}
```

### Advanced Level (18+ months)

#### Advanced Framework Architecture
```
Microservices Testing Strategy:
public class ServiceTestOrchestrator {
    private final Map<String, TestContainer> services = new HashMap<>();
    
    @BeforeClass
    public void setupTestEnvironment() {
        // Start required services in containers
        services.put("user-service", startUserService());
        services.put("order-service", startOrderService());
        services.put("payment-service", startPaymentService());
        
        // Wait for services to be ready
        waitForServicesReady();
        
        // Configure service discovery
        configureServiceDiscovery();
    }
    
    @Test
    public void testCompleteOrderWorkflow() {
        // Test cross-service communication
        String userId = createUser();
        String productId = createProduct();
        String orderId = createOrder(userId, productId);
        String paymentId = processPayment(orderId);
        
        // Verify workflow completion
        verifyOrderCompleted(orderId);
        verifyPaymentProcessed(paymentId);
        verifyInventoryUpdated(productId);
    }
    
    private TestContainer startUserService() {
        return new GenericContainer<>("user-service:latest")
            .withExposedPorts(8080)
            .withEnv("DATABASE_URL", getDatabaseUrl())
            .waitingFor(Wait.forHttp("/health"));
    }
}

Machine Learning Test Validation:
public class MLModelTestFramework {
    
    @Test
    public void testRecommendationEngine() {
        // Load test dataset
        Dataset testData = loadTestDataset("recommendations_test.csv");
        
        // Generate predictions
        List<Recommendation> predictions = recommendationService.generateRecommendations(
            testData.getUserIds()
        );
        
        // Validate prediction quality
        double precision = calculatePrecision(predictions, testData.getExpectedResults());
        double recall = calculateRecall(predictions, testData.getExpectedResults());
        double f1Score = 2 * (precision * recall) / (precision + recall);
        
        // Assert quality thresholds
        Assert.assertTrue("Precision too low", precision >= 0.85);
        Assert.assertTrue("Recall too low", recall >= 0.80);
        Assert.assertTrue("F1 score too low", f1Score >= 0.82);
    }
    
    @Test
    public void testModelBiasDetection() {
        // Test for demographic parity
        Map<String, Double> demographicResults = analyzeRecommendationsByDemographic();
        
        for (Map.Entry<String, Double> entry : demographicResults.entrySet()) {
            String demographic = entry.getKey();
            Double accuracy = entry.getValue();
            
            Assert.assertTrue(
                String.format("Bias detected in %s demographic", demographic),
                Math.abs(accuracy - 0.85) < 0.05 // Within 5% of expected accuracy
            );
        }
    }
}
```

## Best Practices

### Framework Design Principles

#### SOLID Principles in Test Automation
```
Single Responsibility Principle:
// Good: Each page class has single responsibility
public class LoginPage {
    // Only login-related functionality
    public void enterCredentials(String username, String password) { }
    public void clickLogin() { }
    public String getErrorMessage() { }
}

public class NavigationHelper {
    // Only navigation-related functionality
    public void navigateToPage(String url) { }
    public void waitForPageLoad() { }
}

Open/Closed Principle:
// Base test class open for extension, closed for modification
public abstract class BaseTest {
    protected WebDriver driver;
    
    @BeforeMethod
    public void setup() {
        driver = createDriver();
    }
    
    protected abstract WebDriver createDriver();
    
    @AfterMethod
    public void teardown() {
        if (driver != null) {
            driver.quit();
        }
    }
}

// Extended for specific browser
public class ChromeTest extends BaseTest {
    @Override
    protected WebDriver createDriver() {
        return new ChromeDriver();
    }
}

Dependency Inversion Principle:
// Depend on abstractions, not concretions
public interface ReportGenerator {
    void generateReport(TestResults results);
}

public class HTMLReportGenerator implements ReportGenerator {
    @Override
    public void generateReport(TestResults results) {
        // Generate HTML report
    }
}

public class TestRunner {
    private final ReportGenerator reportGenerator;
    
    public TestRunner(ReportGenerator reportGenerator) {
        this.reportGenerator = reportGenerator;
    }
    
    public void runTests() {
        TestResults results = executeTests();
        reportGenerator.generateReport(results);
    }
}
```

### Error Handling and Reliability
```
Robust Element Interaction:
public class WebElementUtils {
    private static final int DEFAULT_TIMEOUT = 10;
    private static final int RETRY_COUNT = 3;
    
    public static void safeClick(WebDriver driver, By locator) {
        WebDriverWait wait = new WebDriverWait(driver, DEFAULT_TIMEOUT);
        
        for (int attempt = 1; attempt <= RETRY_COUNT; attempt++) {
            try {
                WebElement element = wait.until(ExpectedConditions.elementToBeClickable(locator));
                
                // Scroll element into view
                ((JavascriptExecutor) driver).executeScript(
                    "arguments[0].scrollIntoView(true);", element
                );
                
                // Try normal click first
                element.click();
                return;
                
            } catch (ElementClickInterceptedException e) {
                // Try JavaScript click if normal click fails
                WebElement element = driver.findElement(locator);
                ((JavascriptExecutor) driver).executeScript("arguments[0].click();", element);
                return;
                
            } catch (Exception e) {
                if (attempt == RETRY_COUNT) {
                    throw new RuntimeException(
                        String.format("Failed to click element after %d attempts: %s", 
                        RETRY_COUNT, locator), e
                    );
                }
                
                // Wait before retry
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt();
                    throw new RuntimeException("Interrupted during retry", ie);
                }
            }
        }
    }
    
    public static void safeType(WebDriver driver, By locator, String text) {
        WebDriverWait wait = new WebDriverWait(driver, DEFAULT_TIMEOUT);
        
        WebElement element = wait.until(ExpectedConditions.visibilityOfElementLocated(locator));
        
        // Clear field multiple ways to ensure it's empty
        element.clear();
        element.sendKeys(Keys.CONTROL + "a");
        element.sendKeys(Keys.DELETE);
        
        // Type text
        element.sendKeys(text);
        
        // Verify text was entered correctly
        String enteredText = element.getAttribute("value");
        if (!text.equals(enteredText)) {
            throw new RuntimeException(
                String.format("Text verification failed. Expected: '%s', Actual: '%s'", 
                text, enteredText)
            );
        }
    }
}

Screenshot and Logging Strategy:
public class TestLogger {
    private static final Logger logger = LoggerFactory.getLogger(TestLogger.class);
    
    public static void logStep(String step) {
        logger.info("Test Step: {}", step);
    }
    
    public static void logError(String message, Throwable throwable) {
        logger.error("Test Error: {}", message, throwable);
    }
    
    public static String captureScreenshot(WebDriver driver, String testName) {
        try {
            TakesScreenshot screenshot = (TakesScreenshot) driver;
            byte[] sourceFile = screenshot.getScreenshotAs(OutputType.BYTES);
            
            String fileName = String.format("%s_%s_%d.png", 
                testName, 
                LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMdd_HHmmss")),
                Thread.currentThread().getId()
            );
            
            Path screenshotPath = Paths.get("screenshots", fileName);
            Files.createDirectories(screenshotPath.getParent());
            Files.write(screenshotPath, sourceFile);
            
            logger.info("Screenshot captured: {}", screenshotPath.toString());
            return screenshotPath.toString();
            
        } catch (Exception e) {
            logger.error("Failed to capture screenshot", e);
            return null;
        }
    }
}

Test Retry Mechanism:
public class RetryAnalyzer implements IRetryAnalyzer {
    private int count = 0;
    private static final int MAX_RETRY_COUNT = 2;
    
    @Override
    public boolean retry(ITestResult result) {
        if (count < MAX_RETRY_COUNT) {
            count++;
            
            // Log retry attempt
            TestLogger.logStep(String.format("Retrying test: %s (Attempt %d/%d)", 
                result.getMethod().getMethodName(), count, MAX_RETRY_COUNT));
            
            // Capture screenshot for failed attempt
            if (result.getInstance() instanceof BaseTest) {
                BaseTest test = (BaseTest) result.getInstance();
                TestLogger.captureScreenshot(test.getDriver(), 
                    result.getMethod().getMethodName() + "_retry_" + count);
            }
            
            return true;
        }
        
        return false;
    }
}

// Apply retry to specific tests
@Test(retryAnalyzer = RetryAnalyzer.class)
public void flakyTest() {
    // Test implementation
}
```

### Continuous Improvement and Metrics
```
Test Metrics Collection:
public class TestMetricsCollector {
    private final InfluxDB influxDB;
    
    public void recordTestExecution(ITestResult result) {
        Point.Builder pointBuilder = Point.measurement("test_execution")
            .time(System.currentTimeMillis(), TimeUnit.MILLISECONDS)
            .tag("test_class", result.getTestClass().getName())
            .tag("test_method", result.getMethod().getMethodName())
            .tag("browser", getBrowserName())
            .tag("environment", getTestEnvironment())
            .field("duration_ms", result.getEndMillis() - result.getStartMillis())
            .field("status", getTestStatus(result));
            
        if (result.getStatus() == ITestResult.FAILURE) {
            pointBuilder.field("failure_reason", result.getThrowable().getMessage());
        }
        
        influxDB.write(pointBuilder.build());
    }
    
    public TestSuiteMetrics generateSuiteMetrics(List<ITestResult> results) {
        return TestSuiteMetrics.builder()
            .totalTests(results.size())
            .passedTests(countByStatus(results, ITestResult.SUCCESS))
            .failedTests(countByStatus(results, ITestResult.FAILURE))
            .skippedTests(countByStatus(results, ITestResult.SKIP))
            .averageDuration(calculateAverageDuration(results))
            .passRate(calculatePassRate(results))
            .build();
    }
}

Automated Maintenance:
public class TestMaintenanceBot {
    
    public void analyzeTestStability() {
        // Identify flaky tests
        List<String> flakyTests = findTestsWithHighFailureRate();
        
        // Create maintenance tickets
        for (String testName : flakyTests) {
            createMaintenanceTicket(testName, "High failure rate detected");
        }
    }
    
    public void updateOutdatedSelectors() {
        // Scan for broken selectors
        List<String> brokenSelectors = findBrokenSelectors();
        
        // Suggest alternative selectors
        for (String selector : brokenSelectors) {
            List<String> alternatives = suggestAlternativeSelectors(selector);
            logSelectorSuggestions(selector, alternatives);
        }
    }
    
    public void optimizeTestSuite() {
        // Identify redundant tests
        List<TestPair> redundantTests = findRedundantTests();
        
        // Generate optimization report
        generateOptimizationReport(redundantTests);
    }
}
```

---

**Next Step**: Test your APIs thoroughly! Continue to [API Testing](./03-api-testing.md) to learn REST, GraphQL, and reverse API testing techniques.
