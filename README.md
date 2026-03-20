# SDET Interview Prep — Complete Study Notes
> Java • Selenium • TestNG • Maven • Cucumber • Framework Design

![Progress](https://img.shields.io/badge/Day%201-7.5%2F10-blue) ![Progress](https://img.shields.io/badge/Day%202-8%2F10-green)

---

## 📅 5-Day Battle Plan

| Day | Topics | Status |
|-----|--------|--------|
| **Day 1** | Java — OOP, Collections, Exception Handling, Strings | ✅ Done |
| **Day 2** | Selenium — Waits, Locators, POM, StaleElement, Windows, JS, Framework Design | ✅ Done |
| **Day 3** | TestNG — Annotations, Groups, Listeners, DataProvider, Parallel | 🔜 |
| **Day 4** | Maven + Selenium Grid + Parallel Execution | 🔜 |
| **Day 5** | Mock Interview — Full Simulation | 🔜 |

---

# DAY 1 — Java for SDET

---

## 1. OOP — Four Pillars

> 💡 Always tie every pillar back to your Selenium framework in the interview.

---

### Encapsulation
Bundling data + methods in a class. Controlling access via access modifiers.
- Locators are `private`, methods are `public` in Page classes.

```java
public class LoginPage {
    private By username = By.id("username");  // private — no direct access

    public void enterUsername(String user) {  // public — accessible
        driver.findElement(username).sendKeys(user);
    }
}
```

---

### Abstraction
Hiding implementation details — exposing only what is needed.
- `WebDriver` is an interface. `ChromeDriver` implements it. We code to the interface.

```java
WebDriver driver = new ChromeDriver();  // abstraction — we use interface
driver.get("https://example.com");      // don't need to know HOW it works internally
```

---

### Inheritance
Child class inherits properties and methods of parent class.
- `BasePage` holds common methods. All page classes extend it.

```java
public class BasePage {
    protected WebDriver driver;

    public void waitForElement(By locator) {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        wait.until(ExpectedConditions.visibilityOfElementLocated(locator));
    }
}

public class LoginPage extends BasePage {  // inherits all BasePage methods
    public LoginPage(WebDriver driver) {
        super(driver);
    }
}
```

---

### Polymorphism
Same reference behaving differently at runtime.
- `WebDriver driver = new ChromeDriver()` — parent reference, child object.
- **Overloading** = compile time | **Overriding** = runtime

```java
// Runtime Polymorphism — parent reference holds child object
WebDriver driver = new ChromeDriver();
WebDriver driver = new FirefoxDriver();  // same reference, different behaviour

// Method Overloading (compile time)
void click(String locator) {}
void click(By locator, int timeout) {}

// Method Overriding (runtime)
@Override
void setUp() { driver = new FirefoxDriver(); }
```

---

## 2. Collections — Cheat Sheet

| Collection | Duplicates | Order | Best Use Case |
|---|---|---|---|
| `ArrayList` | ✅ Yes | Insertion order | Default list, frequent reads |
| `LinkedList` | ✅ Yes | Insertion order | Frequent insert/delete |
| `HashSet` | ❌ No | No order | Just uniqueness |
| `LinkedHashSet` | ❌ No | Insertion order | **Unique + ordered ← interview trap** |
| `TreeSet` | ❌ No | Sorted A-Z | Unique + sorted |
| `HashMap` | Values ✅ | No order | Key-value lookup |
| `LinkedHashMap` | Values ✅ | Insertion order | Key-value + ordered |
| `TreeMap` | Values ✅ | Sorted keys | Key-value + sorted |

> ⚠️ **Interview Trap:** "No duplicates + preserve insertion order" → `LinkedHashSet` NOT `HashSet`

---

### Map Iteration — 3 Ways

```java
Map<String, String> credentials = new HashMap<>();
credentials.put("john", "pass123");
credentials.put("avani", "pass456");

// 1. entrySet() — both key and value (most common)
for(Map.Entry<String, String> entry : credentials.entrySet()) {
    System.out.println(entry.getKey() + " : " + entry.getValue());
}

// 2. keySet() — keys only
for(String key : credentials.keySet()) {
    System.out.println(key);
}

// 3. values() — values only
for(String val : credentials.values()) {
    System.out.println(val);
}

// Bonus — getOrDefault() — elegant word/item count
map.put(word, map.getOrDefault(word, 0) + 1);
```

---

### Key Map Methods

| Method | What it does |
|---|---|
| `map.containsKey("cat")` | Check if key exists |
| `map.get("cat")` | Get value for key |
| `map.getOrDefault("cat", 0)` | Get value or default if missing |
| `map.put("cat", 1)` | Add/update key-value |
| `map.entrySet()` | Set of key-value pairs |

---

### Common Coding Patterns

```java
// Word count
public static Map<String, Integer> countWords(String sentence) {
    Map<String, Integer> wordCount = new HashMap<>();
    String[] words = sentence.split(" ");
    for(String word : words) {
        wordCount.put(word, wordCount.getOrDefault(word, 0) + 1);
    }
    return wordCount;
}

// PASS/FAIL counter
public static void countResults(List<String> results) {
    Map<String, Integer> m = new HashMap<>();
    for(String word : results) {
        if(m.containsKey(word)) {
            m.put(word, m.get(word) + 1);
        } else {
            m.put(word, 1);
        }
    }
    for(Map.Entry<String, Integer> en : m.entrySet()) {
        System.out.println(en.getKey() + " : " + en.getValue());
    }
}

// String reverse (without built-in method)
public static String reverse(String s) {
    String rev = "";
    for(int i = 0; i < s.length(); i++) {
        rev = s.charAt(i) + rev;  // prepend each char
    }
    return rev;
}
```

---

## 3. Exception Handling

| Type | When | Examples |
|---|---|---|
| Checked | Compiler forces you to handle | `IOException`, `SQLException`, `FileNotFoundException` |
| Unchecked | Runtime — may crash | `NullPointerException`, `ArrayIndexOutOfBoundsException` |
| Selenium | Runtime Selenium errors | `NoSuchElementException`, `StaleElementReferenceException` |

---

### Selenium Exceptions — Memorize These 6

| Exception | When It Occurs |
|---|---|
| `NoSuchElementException` | Element not found in DOM |
| `StaleElementReferenceException` | DOM refreshed after element was found |
| `TimeoutException` | Wait condition not met in time |
| `ElementNotInteractableException` | Element exists but can't interact with it |
| `NoSuchFrameException` | Frame not found when switching |
| `NoAlertPresentException` | Alert expected but not present |

---

### throw vs throws

```java
// throws — declares that method MIGHT throw exception
public void openURL() throws IOException { }

// throw — actually throws the exception manually
if(driver == null) {
    throw new RuntimeException("Driver not initialized");
}
```

---

### try-catch-finally in Framework

```java
public void clickElement(By locator) {
    try {
        driver.findElement(locator).click();
    } catch (NoSuchElementException e) {
        System.out.println("Element not found: " + locator);
        throw e;  // rethrow so test fails with clear message
    } catch (StaleElementReferenceException e) {
        driver.findElement(locator).click();  // retry
    } finally {
        System.out.println("Click attempt done");  // ALWAYS runs — even if exception
    }
}
```

> 💡 `finally` is the perfect place to close browser — it runs even if test fails midway.

---

## 4. String — Common Interview Traps

```java
// == vs .equals()
String a = new String("chrome");
String b = new String("chrome");

a == b          // false — different objects in memory
a.equals(b)     // true  — same content

// String pool trick
String x = "chrome";
String y = "chrome";
x == y          // true — both point to same pool object

// Framework bug — using == for title comparison
if(driver.getTitle() == "Login Page") { }   // ❌ WRONG — always false

// Correct
Assert.assertEquals(driver.getTitle(), "Login Page");  // ✅
"Login Page".equals(driver.getTitle());                // ✅ null safe
```

> ⚠️ **Null safe tip:** Always put the known/expected string first — `"expected".equals(actual)` — won't throw NPE if actual is null.

---

## Day 1 — Scorecard

| Topic | Status | Score |
|---|---|---|
| OOP — 4 Pillars with framework examples | ✅ | 8.5/10 |
| Collections — LinkedHashSet trap | ✅ | 8/10 |
| Map iteration syntax | ✅ | 8/10 |
| Exception handling — Selenium exceptions | ⚠️ needs practice | 7/10 |
| == vs .equals() — null safe comparison | ✅ | 9/10 |
| String reverse — solo task | ✅ | 10/10 |
| Word count / PASS-FAIL counter | ✅ | 8.5/10 |

**Overall Day 1 — 7.5/10 ✅**

---

# DAY 2 — Selenium Deep Dive

---

## 1. Waits — Most Asked Selenium Topic

| Wait Type | Scope | Best For |
|---|---|---|
| Implicit Wait | Global — all findElement calls | Simple stable pages |
| **Explicit Wait** | Targeted — specific condition | **Dynamic elements — USE THIS** |
| Fluent Wait | Explicit + polling interval | Slow loading elements |
| Thread.sleep | Hard pause | Only for debugging — never in framework |

> ❌ **NEVER mix implicit and explicit wait** — causes unpredictable timeout behaviour.

```java
// 1. Implicit Wait
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));

// 2. Explicit Wait — USE THIS in framework
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("btn")));
wait.until(ExpectedConditions.elementToBeClickable(By.id("loginBtn")));
wait.until(ExpectedConditions.titleContains("Dashboard"));
wait.until(ExpectedConditions.alertIsPresent());
wait.until(ExpectedConditions.frameToBeAvailableAndSwitchToIt(By.id("frame")));

// 3. Fluent Wait
Wait<WebDriver> fluentWait = new FluentWait<>(driver)
    .withTimeout(Duration.ofSeconds(30))
    .pollingEvery(Duration.ofSeconds(2))        // checks every 2 seconds
    .ignoring(NoSuchElementException.class);    // ignores this during wait

WebElement element = fluentWait.until(
    driver -> driver.findElement(By.id("dynamicElement"))
);
```

---

### ExpectedConditions — Know These

| Condition | Use When |
|---|---|
| `visibilityOfElementLocated` | Element visible on screen |
| `elementToBeClickable` | Element ready to click |
| `presenceOfElementLocated` | Element in DOM but may not be visible |
| `titleContains` | Page title verification |
| `alertIsPresent` | Waiting for alert |
| `frameToBeAvailableAndSwitchToIt` | Before switching to iframe |
| `refreshed(condition)` | After DOM refresh — for stale elements |

---

## 2. Locators & XPath

### CSS Selector Syntax

| Selector | Syntax | Example |
|---|---|---|
| ID | `#value` | `#username` |
| Class | `.value` | `.login-input` |
| Tag + Class | `tag.class` | `input.login-input` |
| Attribute | `[attr='val']` | `[type='text']` |
| Contains | `[attr*='val']` | `[class*='login']` |
| Starts with | `[attr^='val']` | `[id^='user']` |
| Child | `parent > child` | `div > input` |

```java
// All locators for: <input type="text" id="username" class="login-input">
By.id("username")
By.cssSelector("#username")
By.cssSelector(".login-input")
By.cssSelector("input.login-input")
By.cssSelector("input[type='text']")
By.xpath("//input[@id='username']")
By.xpath("//input[@class='login-input']")
```

---

### XPath — Dynamic Elements

```java
// contains() — partial match for dynamic IDs
By.xpath("//input[contains(@id,'user')]");

// starts-with()
By.xpath("//input[starts-with(@id,'user')]");

// text() — for buttons and links
By.xpath("//button[text()='Login']");
By.xpath("//a[contains(text(),'Sign')]");

// XPath Axes
//input[@id='username']/parent::div
//label[text()='Username']/following-sibling::input
//input[@id='username']/ancestor::form
//form[@id='loginForm']/child::input
```

> 💡 **CSS is faster than XPath** — CSS interacts directly with browser engine. XPath traverses the DOM.

---

## 3. POM + PageFactory

```java
public class LoginPage {
    WebDriver driver;

    // @FindBy — lazy initialization (located only when accessed)
    @FindBy(id = "username") private WebElement username;
    @FindBy(id = "password") private WebElement password;
    @FindBy(id = "loginBtn")  private WebElement loginBtn;

    // List of elements
    @FindBy(className = "menu-item") private List<WebElement> menuItems;

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);  // initialize all @FindBy
    }

    public void enterUsername(String user) { username.sendKeys(user); }
    public void enterPassword(String pass) { password.sendKeys(pass); }
    public void clickLogin()               { loginBtn.click(); }

    // Combined method — clean and reusable
    public void login(String user, String pass) {
        enterUsername(user);
        enterPassword(pass);
        clickLogin();
    }
}

// In test class
LoginPage loginPage = new LoginPage(driver);
loginPage.login("admin", "password123");
```

> 💡 **Lazy initialization** — `@FindBy` elements are located only when accessed, not when page object is created.

---

## 4. StaleElementReferenceException

**When it occurs:**
1. You find an element → store reference
2. Page refreshes / DOM updates (AJAX, SPA, form submit)
3. You try to use old reference → Exception!

```java
// Option 1 — Re-find element after DOM change
driver.navigate().refresh();
driver.findElement(By.id("submit")).click();  // fresh reference ✅

// Option 2 — Retry mechanism (best practice in framework)
public void clickWithRetry(By locator) {
    int attempts = 0;
    while(attempts < 3) {
        try {
            driver.findElement(locator).click();
            break;
        } catch(StaleElementReferenceException e) {
            attempts++;
            System.out.println("Stale element, retrying: " + attempts);
        }
    }
}

// Option 3 — Explicit Wait with refreshed()
wait.until(ExpectedConditions.refreshed(
    ExpectedConditions.elementToBeClickable(By.id("submit"))
));
```

---

## 5. Windows & Frames

```java
// ── Window Switching ────────────────────────────────────────────────────────
String parentWindow = driver.getWindowHandle();     // save parent
driver.findElement(By.id("openBtn")).click();        // opens new window

Set<String> allWindows = driver.getWindowHandles(); // get all handles
for(String window : allWindows) {
    if(!window.equals(parentWindow)) {
        driver.switchTo().window(window);            // switch to child
        break;
    }
}
// Do actions in child window...
driver.switchTo().window(parentWindow);              // switch back to parent

// ── iFrame — 3 Ways ─────────────────────────────────────────────────────────
driver.switchTo().frame(0);                                   // by index
driver.switchTo().frame("iframeName");                        // by name/id
driver.switchTo().frame(driver.findElement(By.id("frame"))); // by WebElement

driver.switchTo().defaultContent();  // back to main page
driver.switchTo().parentFrame();     // one level up
```

> ⚠️ **Common trap:** Always switch to iframe before interacting with elements inside it — or you'll get `NoSuchElementException`.

---

## 6. JavaScriptExecutor & Actions Class

```java
// ── When element.click() fails — 3 fallbacks ──────────────────────────────

// 1. Explicit wait first (always try this)
wait.until(ExpectedConditions.elementToBeClickable(By.id("btn"))).click();

// 2. JSExecutor click
JavascriptExecutor js = (JavascriptExecutor) driver;
js.executeScript("arguments[0].click();", element);

// 3. Scroll to element then click
js.executeScript("arguments[0].scrollIntoView(true);", element);
element.click();

// ── Other JSExecutor helpers ──────────────────────────────────────────────
js.executeScript("window.scrollTo(0, document.body.scrollHeight)"); // scroll to bottom
js.executeScript("window.scrollTo(0, 0)");                          // scroll to top
String title = (String) js.executeScript("return document.title");
js.executeScript("arguments[0].style.border='3px solid red'", el); // highlight for debug

// ── Actions Class ─────────────────────────────────────────────────────────
Actions actions = new Actions(driver);

actions.moveToElement(element).perform();                           // hover
actions.contextClick(element).perform();                            // right click
actions.doubleClick(element).perform();                             // double click
actions.dragAndDrop(source, target).perform();                      // drag and drop
actions.keyDown(Keys.CONTROL).sendKeys("a").keyUp(Keys.CONTROL).perform(); // Ctrl+A
```

---

## 7. Screenshot on Failure

```java
// Step 1 — Screenshot utility class
public class ScreenshotUtil {
    public static void takeScreenshot(WebDriver driver, String testName) {
        TakesScreenshot ts = (TakesScreenshot) driver;          // cast driver
        File source = ts.getScreenshotAs(OutputType.FILE);      // capture
        try {
            FileUtils.copyFile(source, new File("screenshots/" + testName + ".png"));
        } catch(IOException e) { e.printStackTrace(); }
    }
}

// Output types
ts.getScreenshotAs(OutputType.FILE);     // as File
ts.getScreenshotAs(OutputType.BASE64);   // for Allure/Extent reports
ts.getScreenshotAs(OutputType.BYTES);    // as byte array

// Step 2 — TestNG Listener (hooks to test failure automatically)
public class TestListener implements ITestListener {
    @Override
    public void onTestFailure(ITestResult result) {
        WebDriver driver = ((BaseTest) result.getInstance()).driver;
        ScreenshotUtil.takeScreenshot(driver, result.getName());
    }
    // Other override methods left empty
}

// Step 3 — Register in testng.xml
// <listeners>
//     <listener class-name="listeners.TestListener"/>
// </listeners>
```

> 💡 **Interview line:** "In my current framework Serenity handles screenshots automatically. But I know the manual implementation — cast to `TakesScreenshot`, call `getScreenshotAs()`, hook to failures via `ITestListener.onTestFailure()`."

---

## 8. Framework Architecture — Your 60-Second Answer

> 🎤 **Say this in the interview:**
>
> *"My framework is built on Java with Maven for dependency management. It follows BDD using Cucumber — scenarios written in Gherkin in feature files, readable by non-technical stakeholders.*
>
> *Architecture follows Page Object Model — page classes under `src/main/java/pages`, private locators, public methods. Step definitions map Gherkin steps to Selenium actions.*
>
> *Test data via Scenario Outline with Examples for simple data. Excel/JSON for complex cases.*
>
> *Serenity BDD for reporting — auto screenshots, living documentation, step-by-step execution.*
>
> *DriverFactory initializes WebDriver by config. Explicit waits with ExpectedConditions for sync.*
>
> *Flow: Runner → Cucumber matches feature file → StepDefs call Page methods → Selenium interacts with browser → Serenity captures results."*

```
src/
├── main/java/
│   ├── pages/            ← BasePage, LoginPage, HomePage
│   └── utils/            ← DriverFactory, ConfigReader
└── test/java/
    ├── stepDefinitions/  ← LoginSteps, HomeSteps
    ├── runners/          ← TestRunner
    └── features/         ← login.feature, home.feature
pom.xml
serenity.properties
```

**End-to-end flow:**
```
Feature File
    ↓
TestRunner picks it up
    ↓
Cucumber matches steps → StepDefinitions
    ↓
StepDefinitions call Page class methods
    ↓
Page classes use Selenium → Browser
    ↓
Serenity captures screenshots + generates report
```

---

## Day 2 — Scorecard

| Topic | Status | Score |
|---|---|---|
| Waits — implicit, explicit, fluent | ✅ | 8/10 |
| CSS Selector syntax | ⚠️ needs practice | 6/10 |
| POM + PageFactory + lazy init | ✅ | 9/10 |
| StaleElementReferenceException + retry | ✅ | 8.5/10 |
| Window switching logic | ✅ | 9/10 |
| iFrame — 3 ways to switch | ✅ | 8/10 |
| JSExecutor + Actions class | ✅ | 8/10 |
| Screenshots — TakesScreenshot + Listener | ✅ | 9/10 |
| Framework architecture walkthrough | ✅ | 9/10 |

**Overall Day 2 — 8/10 ✅**

---

# Quick Revision — Fill In The Blanks

Test yourself before the interview:

| Question | Answer |
|---|---|
| Collection — no duplicates + insertion order? | `LinkedHashSet` |
| Iterate Map with key and value? | `entrySet()` → `Map.Entry<K,V>` |
| Wait that polls every N seconds? | `FluentWait` with `.pollingEvery()` |
| Interface used to take screenshots? | `TakesScreenshot` |
| Method to initialize @FindBy elements? | `PageFactory.initElements(driver, this)` |
| Switch back from iframe to main page? | `driver.switchTo().defaultContent()` |
| When does StaleElementException occur? | DOM refreshes after element was found |
| Null safe string comparison? | `"expected".equals(actual)` |
| Listener method for screenshot on fail? | `onTestFailure(ITestResult result)` |
| Why never mix implicit + explicit wait? | Unpredictable timeout behaviour |
| Parent reference holding child object? | Polymorphism |
| Where is `PageFactory.initElements()` called? | Inside the page class constructor |

---

# Coming Up

| Day | Topics |
|---|---|
| **Day 3** | TestNG — `@Test`, `@BeforeMethod`, `@DataProvider`, Groups, Listeners, Parallel config |
| **Day 4** | Maven — POM.xml, plugins, profiles + Selenium Grid + Parallel execution |
| **Day 5** | Full Mock Interview Simulation |

---

*Prepared during interview prep sessions — 5 days to SDET interview* 🎯
