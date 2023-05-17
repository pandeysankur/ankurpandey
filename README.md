package com.selenium.bootcamp.tests;

import com.selenium.bootcamp.utillity.SystemUtility;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.*;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.interactions.Actions;
import org.openqa.selenium.remote.RemoteWebDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.Select;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.annotations.AfterClass;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.Test;

import java.io.File;
import java.time.Duration;
import java.util.List;
import java.util.Set;

public class AprilTests {

    final private String USER_DIR = SystemUtility.getUserDirectory();
    final private String SEPARATOR = SystemUtility.getFileSeparator();
    WebDriver driver;

    @BeforeClass
    public void initializeBrowser() throws InterruptedException {
        WebDriverManager.chromedriver().setup();
        ChromeOptions chromeOptions = new ChromeOptions();
        chromeOptions.addExtensions(new File(USER_DIR + SEPARATOR + "browser-extensions" + SEPARATOR + "Ultimate AdBlocker_2_2_6_0.crx"));

        driver = new ChromeDriver(chromeOptions);
        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        Thread.sleep(2000);
    }

    @AfterClass
    public void tearDown() throws InterruptedException {
        Thread.sleep(5000);
        driver.close();
    }

    @Test
    public void inputElementTest() {
        driver.get("http://demo.seleniumeasy.com/input-form-demo.html");
        System.out.println("WebSite Title : " + driver.getTitle());
        driver.findElement(By.name("first_name")).sendKeys("Hitesh");
    }

    @Test
    public void selectElementTest() throws InterruptedException {
        driver.get("http://demo.seleniumeasy.com/input-form-demo.html");
        WebElement selectElement = driver.findElement(By.name("state"));

        Select elementAsSelect = new Select(selectElement);
        elementAsSelect.selectByVisibleText("District of Columbia");
        // get selected option
        String optionSelectedOne = elementAsSelect.getFirstSelectedOption().getText();
        System.out.println("First selected option : " + optionSelectedOne);
        Thread.sleep(5000);
        elementAsSelect.selectByIndex(10);
        // get selected option
        String optionSelectedTwo = elementAsSelect.getFirstSelectedOption().getText();
        System.out.println("Second selected option : " + optionSelectedTwo);
        Thread.sleep(5000);
    }

    @Test
    public void radioComponentTest() throws InterruptedException {
        driver.get("http://demo.seleniumeasy.com/input-form-demo.html");
        List<WebElement> radioElements = driver.findElements(By.name("hosting"));
        // click on yes
        radioElements.get(0).click();
        Thread.sleep(5000);
        // click on No
        radioElements.get(1).click();
    }

    @Test
    public void radioComponentUsingXPathTest() throws InterruptedException {
        driver.get("http://demo.seleniumeasy.com/input-form-demo.html");
        driver.findElement(By.xpath("//label[normalize-space()='Yes']")).click();
        Thread.sleep(5000);
        driver.findElement(By.xpath("//label[normalize-space()='No']")).click();

    }

    @Test
    public void selectorHubPracticePageTest() {
        driver.get("https://selectorshub.com/xpath-practice-page/");

        driver.findElement(By.id("userId")).sendKeys("hiteshprajapati1992@gmail.com");
        driver.findElement(By.id("pass")).sendKeys("SomeRandomPassword");
        driver.findElement(By.name("company")).sendKeys("Paul Mason Consulting");
        driver.findElement(By.name("mobile number")).sendKeys("+9125789544125");
        driver.findElement(By.id("inp_val")).sendKeys("Prefer not to say. lol..");

        // select element
        WebElement carSelElement = driver.findElement(By.id("cars"));
        Select carSelectElement = new Select(carSelElement);
        // print all the options
        carSelectElement.getOptions().forEach(e -> System.out.println(e.getText()));
        // select random options
        carSelectElement.selectByVisibleText("Volvo");
        carSelectElement.selectByVisibleText("Opel");

        // select all the options one by one and print it
        for (int i = 0; i < carSelectElement.getOptions().size(); i++) {
            carSelectElement.selectByIndex(i);
            String currSelectedOption = carSelectElement.getFirstSelectedOption().getText();
            System.out.println("Option Selected At Index [" + i + "] Is : " + currSelectedOption);
        }

        // print all the hrefs
        List<WebElement> allLinkElements = driver.findElements(By.tagName("a")).stream().filter(WebElement::isDisplayed).toList();

        System.out.println("Total Links Found : " + allLinkElements.size());
//        for (WebElement anchorElement : allLinkElements) {
//            String text = anchorElement.getText();
//            String href = anchorElement.getAttribute("href");
//            System.out.println("Text [" + text + "] has href as : " + href);
//        }

        allLinkElements.forEach(element -> {
            String text = element.getText();
            String href = element.getAttribute("href");
            System.out.println("Text [" + text + "] has href as : " + href);
        });

    }

    @Test
    public void synchronizationTest() {
        driver.get("http://the-internet.herokuapp.com/dynamic_loading/1");
        driver.findElement(By.id("start")).findElement(By.tagName("button")).click();

        // create webdriverobject - Example of Explicit Wait
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofMinutes(2));
        wait.until(ExpectedConditions.and(
                ExpectedConditions.invisibilityOfElementLocated(By.id("loading")),
                ExpectedConditions.visibilityOfElementLocated((By.id("finish")))
        ));
        String finalValue = driver.findElement(By.id("finish")).getText();
        System.out.println("Final Value : " + finalValue);
    }

    @Test
    public void multiWindowAndTabTest() throws InterruptedException {
        driver.get("https://www.saucedemo.com/");

        String parentWindow = driver.getWindowHandle();
        System.out.println("Parent Window ID : " + parentWindow);
        System.out.println("Parent Driver Session : " + ((RemoteWebDriver) driver).getSessionId());
        driver.switchTo().newWindow(WindowType.WINDOW);
        //driver.switchTo().newWindow(WindowType.TAB);
        Thread.sleep(3000);
        driver.get("http://the-internet.herokuapp.com/");

        Set<String> windows = driver.getWindowHandles();
        windows.forEach(System.out::println);

        driver.switchTo().window(windows.stream().toList().get(1));
        driver.findElement(By.linkText("Drag and Drop")).click();
        String title = driver.findElement(By.tagName("h3")).getText();
        System.out.println("Title Is : " + title);
        System.out.println("Child Driver Session : " + ((RemoteWebDriver) driver).getSessionId());

        // close child window
        driver.close();

        // switch to parent
        driver.switchTo().window(parentWindow);
        Thread.sleep(3000);

        // try logging in
        driver.findElement(By.id("user-name")).sendKeys("standard_user");
        driver.findElement(By.id("password")).sendKeys("secret_sauce");
        driver.findElement(By.id("login-button")).click();
    }

    @Test
    public void frameSwitchingTest() {
        driver.get("https://demoqa.com/frames");

        String parentWindow = driver.getWindowHandle();
        System.out.println("Parent Window ID : " + parentWindow);
        WebElement frameElement = driver.findElement(By.id("frame1"));
        driver.switchTo().frame(frameElement);

        // get text
        String text = driver.findElement(By.id("sampleHeading")).getText();
        System.out.println("Text within Frame Is : " + text);
        // switch to parent
        driver.switchTo().defaultContent();
        /// get text of main header which is outside of frame
        String mainHeaderText = driver.findElement(By.className("main-header")).getText();
        System.out.println("Main Header Text : " + mainHeaderText);
    }

    @Test
    public void jQueryDropdownTest() throws InterruptedException {
        driver.get("http://demo.seleniumeasy.com/jquery-dropdown-search-demo.html");
        WebElement selectElement = driver.findElement(By.xpath("//*[contains(@aria-labelledby,'select2-country-container')]"));
        selectElement.click();

        new WebDriverWait(driver, Duration.ofSeconds(30))
                .until(ExpectedConditions.visibilityOfElementLocated(By.id("select2-country-results")));

        List<WebElement> liElements = driver.findElement(By.id("select2-country-results")).findElements(By.tagName("li"))
                .stream().filter(WebElement::isDisplayed).toList();
        liElements.forEach(e -> System.out.println(e.getText()));

        for (WebElement ele : liElements) {
            if (ele.getText().equalsIgnoreCase("India")) {
                ele.click();
                break;
            }
        }
        Thread.sleep(5000);
        /// Following is using conventional Select Element Method
        WebElement hiddenSelElement = driver.findElement(By.id("country"));
        Select selElement = new Select(hiddenSelElement);
        selElement.selectByVisibleText("New Zealand");
    }

    @Test
    public void dragAndDropTest() {
        driver.get("https://demoqa.com/droppable");
        WebElement draggableElement = driver.findElement(By.id("draggable"));
        WebElement droppableElement = driver.findElement(By.id("droppable"));

        Actions dragAndDropAction = new Actions(driver);
        dragAndDropAction
                .dragAndDrop(draggableElement, droppableElement)
                .build().perform();

        // refresh the page
        driver.navigate().refresh();

        draggableElement = driver.findElement(By.id("draggable"));
        droppableElement = driver.findElement(By.id("droppable"));
        dragAndDropAction.clickAndHold(draggableElement)
                .moveToElement(droppableElement)
                .release()
                .build()
                .perform();
    }

    @Test
    public void newTabOrWindowUsingActionClassTest() throws InterruptedException {
        driver.get("https://www.selenium.dev/downloads/");

        WebElement documentationLink = driver.findElement(By.xpath("//*[text()='Documentation']"));
        WebElement projectLink = driver.findElement(By.xpath("//*[text()='Projects']"));
        Thread.sleep(2000);
        Actions tabActions = new Actions(driver);

        // open Documentation on new Tab
        tabActions.keyDown(Keys.LEFT_CONTROL)
                .click(documentationLink)
                .keyUp(Keys.LEFT_CONTROL)
                .build().perform();

        Thread.sleep(5000);
        tabActions.keyDown(Keys.LEFT_SHIFT)
                .click(projectLink)
                .keyUp(Keys.LEFT_SHIFT)
                .build().perform();

    }

    @Test
    public void shadowRootComponentTest() throws InterruptedException {
        driver.get("http://watir.com/examples/shadow_dom.html");
        Thread.sleep(5000);
        WebElement shadowHostParentEle = driver.findElement(By.id("shadow_host"));
        SearchContext shadowRootContext = shadowHostParentEle.getShadowRoot();

        WebElement infoLabelEle = shadowRootContext.findElement(By.className("info"));
        System.out.println("Text Withing Parent Shadow Dom : " + infoLabelEle.getText());
    }
