# GoibiboAirlineAutomation

A Selenium-based automation framework for airline search and booking flows (adapted for a Java + Maven tech stack).

## Tech Stack

- Java 21
- Maven
- Selenium Java 4.x.x
- TestNG 7.x.x
- WebDriverManager 6.x.x
- ExtentReports 5.x.x
- Commons IO
- AutoIT (for native Windows file-upload dialog handling)

## Project Structure

```
.
airline-booking-automation/
├── pom.xml
├── testng.xml
├── Jenkinsfile
├── README.md
│
├── src/
│   ├── main/
│   │   ├── java/com/airline/
│   │   │   ├── pages/
│   │   │   │   ├── HomePage.java
│   │   │   │   ├── FlightResultsPage.java
│   │   │   │   ├── PassengerPage.java
│   │   │   │   ├── PaymentPage.java
│   │   │   │   └── ConfirmationPage.java
│   │   │   ├── core/
│   │   │   │   ├── DriverFactory.java
│   │   │   │   └── BasePage.java
│   │   │   ├── utils/
│   │   │   │   ├── ConfigReader.java
│   │   │   │   ├── WaitUtils.java
│   │   │   │   └── ScreenshotUtils.java
│   │   │   └── models/
│   │   │       └── FlightSearchData.java
│   │   └── resources/
│   │       ├── config.properties
│   │       └── logback.xml
│   │
│   └── test/
│       ├── java/com/airline/tests/
│       │   ├── BaseTest.java
│       │   ├── FlightSearchTest.java
│       │   ├── FlightSelectionTest.java
│       │   ├── PassengerDetailsTest.java
│       │   ├── PaymentTest.java
│       │   ├── BookingConfirmationTest.java
│       │   └── backend/
│       └── resources/
│           └── testdata/
│               └── flight-search.json
│

```

## Framework Overview

- BaseTest loads configuration, starts the browser before each test method, opens the base URL, and quits the driver after each method.
- DriverFactory creates Chrome, Edge, or Firefox browser sessions using WebDriverManager and stores the driver in a ThreadLocal.
- pages contains Page Object Model classes for Login, Dashboard, Leave, and Admin Job Titles pages.
- BasePage provides reusable wait, click, type, text, and URL helper methods.
- TestListener creates ExtentReports entries, logs test results, and captures screenshots plus page source on failures.
- ReportManager, ReportPortal, and Log support report creation and step logging.
- AutoItUtil runs the bundled AutoIT executable for file upload when Selenium cannot use the hidden file input directly.

## Test Scenarios

The intended TestNG suite is defined in testng.xml:

- Login_AdminUser
  - Valid admin login should navigate to Dashboard.
  - Invalid login should show Invalid credentials.
- Dashboard_QuickLaunch
  - Validates that the Assign Leave quick-launch button opens the Leave page.
- Admin_UploadJobSpec
  - Logs in as Admin.
  - Navigates to Admin > Job > Job Titles.
  - Adds a unique job title and uploads src/test/resources/data/sample.pdf.

## Configuration

Default values are stored in:

`src/main/resources/config/config.properties`

Current defaults:

```
baseUrl=https://www.goibibo.com/
username=Admin
password=admin123
browser=chrome
headless=false
```

Supported browsers:

- chrome
- edge
- firefox

ConfigReader allows these values to be overridden with Maven system properties:

```
mvn test -DsuiteXmlFile=testng.xml -Dbrowser=edge -Dheadless=true
```

The suite file also passes browser and headless parameters:

```xml
<parameter name="browser" value="chrome"/>
<parameter name="headless" value="false"/>
```

## Prerequisites

- JDK 17
- Maven
- Chrome, Edge, or Firefox
- Windows, if running the AutoIT upload path

WebDriver binaries are managed automatically by WebDriverManager.

## Running Tests

Run the TestNG suite with Maven:

```
mvn test -DsuiteXmlFile=testng.xml
```

Run in headless Chrome:

```
mvn test -DsuiteXmlFile=testng.xml -Dbrowser=chrome -Dheadless=true
```

Run in Edge:

```
mvn test -DsuiteXmlFile=testng.xml -Dbrowser=edge
```

Run in Firefox:

```
mvn test -DsuiteXmlFile=testng.xml -Dbrowser=firefox
```

You can also run testng.xml directly from an IDE such as Eclipse or IntelliJ.

## Reports and Artifacts

After execution, generated reports and artifacts are written to:

- `target/extent/ExtentReport_<timestamp>.html`
- `target/screenshots/`
- `test-output/`

Failure handling includes:

- Screenshot capture
- HTML page-source capture
- ExtentReports failure logging

## File Upload Notes

The Admin job-title test uploads:

`src/test/resources/data/sample.pdf`

The framework first tries Selenium `sendKeys()` on the file input. If that is not possible, it uses:

`src/test/resources/autoit/UploadFile.exe`

AutoIT works only on Windows. On non-Windows machines, file-upload fallback through AutoIT will throw an unsupported-operation error.

## Adding New Tests

- Create or update a page class under `src/main/java/com/simplilearn/seleniumdemo/pages`.
- Add reusable waits/actions in BasePage only when they are useful across pages.
- Create a test class under `src/test/java/com/simplilearn/seleniumdemo/testcases`.
- Extend BaseTest to reuse browser setup, config loading, and teardown.
- Add the new test class to testng.xml when it should run in the main suite.
