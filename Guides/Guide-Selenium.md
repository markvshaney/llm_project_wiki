# Selenium Guide

## Overview
Selenium is a powerful framework for web browser automation and testing. This guide covers essential concepts and common use cases for Selenium with Python.

## Installation
```bash
pip install selenium
# Also install webdriver-manager for automatic driver management
pip install webdriver-manager
```

## Basic Setup
```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from webdriver_manager.chrome import ChromeDriverManager

# Setup Chrome options
chrome_options = Options()
chrome_options.add_argument("--headless")  # Run in headless mode (optional)

# Initialize the driver
driver = webdriver.Chrome(
    service=Service(ChromeDriverManager().install()),
    options=chrome_options
)
```

## Core Concepts

### Locating Elements
Selenium offers multiple strategies to locate elements:
```python
# Find by ID
element = driver.find_element(By.ID, "search")

# Find by Class Name
elements = driver.find_elements(By.CLASS_NAME, "result-item")

# Find by CSS Selector
element = driver.find_element(By.CSS_SELECTOR, "#main-content > div.article")

# Find by XPath
element = driver.find_element(By.XPATH, "//button[contains(text(), 'Submit')]")
```

### Waiting Strategies
Implementing proper waits is crucial for reliable automation:

```python
# Explicit wait
wait = WebDriverWait(driver, 10)
element = wait.until(
    EC.presence_of_element_located((By.ID, "dynamic-content"))
)

# Implicit wait (set once)
driver.implicitly_wait(10)
```

### Interacting with Elements
Common interactions with web elements:
```python
# Click elements
button = driver.find_element(By.ID, "submit-button")
button.click()

# Send keys (type text)
input_field = driver.find_element(By.NAME, "username")
input_field.send_keys("myusername")

# Clear input fields
input_field.clear()

# Get text content
text = element.text

# Get attribute value
href = element.get_attribute("href")
```

### Handling Frames and Windows
Working with multiple frames and windows:
```python
# Switch to frame
driver.switch_to.frame("frame-name")
driver.switch_to.default_content()  # Switch back to main frame

# Handle multiple windows
original_window = driver.current_window_handle
for window_handle in driver.window_handles:
    if window_handle != original_window:
        driver.switch_to.window(window_handle)
        break
```

## Best Practices

1. Always close the browser after use:
```python
try:
    # Your automation code here
finally:
    driver.quit()
```

2. Use explicit waits over implicit waits when possible
3. Implement proper error handling and retries for flaky elements
4. Use page object model pattern for larger projects
5. Keep selectors maintainable and robust

## Common Patterns

### Taking Screenshots
```python
driver.save_screenshot("screenshot.png")
# Or for specific element
element.screenshot("element.png")
```

### Handling Alerts
```python
# Switch to alert
alert = driver.switch_to.alert
alert.accept()  # Click OK
# or
alert.dismiss() # Click Cancel
```

### Executing JavaScript
```python
# Execute JavaScript
driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
```

## Advanced Topics

### Working with Chrome DevTools Protocol
```python
from selenium.webdriver.chrome.options import Options

chrome_options = Options()
chrome_options.set_capability('goog:loggingPrefs', {'performance': 'ALL'})
```

### Custom Expected Conditions
```python
from selenium.webdriver.support.wait import WebDriverWait

class element_has_css_class(object):
    def __init__(self, locator, css_class):
        self.locator = locator
        self.css_class = css_class
    
    def __call__(self, driver):
        element = driver.find_element(*self.locator)
        if self.css_class in element.get_attribute("class"):
            return element
        return False
```

## Troubleshooting

Common issues and solutions:

1. ElementNotInteractableException
   - Ensure element is visible and not covered by other elements
   - Scroll element into view before interaction
   - Wait for element to be clickable

2. StaleElementReferenceException
   - Re-locate element after page updates
   - Use WebDriverWait with refreshed elements

3. TimeoutException
   - Check if selector is correct
   - Increase wait timeout
   - Verify element actually exists in page

## Resources

- [Selenium Documentation](https://www.selenium.dev/documentation/)
- [WebDriver Manager](https://github.com/SergeyPirogov/webdriver_manager)
- [Selenium Python Bindings](https://selenium-python.readthedocs.io/)