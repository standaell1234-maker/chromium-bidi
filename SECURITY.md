https://github.com/GoogleChromeLabs/chromium-bidi/issues/3986https://github.com/GoogleChromeLabs/chromium-bidi/issues/3974import org.openqa.selenium.JavascriptExecutor;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.bidi.module.Script;
import org.openqa.selenium.bidi.script.ContextTarget;
import org.openqa.selenium.bidi.script.EvaluateParameters;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.devtools.DevTools;
import org.openqa.selenium.devtools.HasDevTools;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.firefox.FirefoxOptions;

import java.util.Optional;

public class JsEvaluationBenchmark {

    private static final int ITERATIONS = 5000;

    static void main(String[] args) {
        runBenchmark(new ChromeDriver(new ChromeOptions().enableBiDi()), "Chrome");
        runBenchmark(new FirefoxDriver(new FirefoxOptions().enableBiDi()), "Firefox");
    }

    private static void runBenchmark(WebDriver driver, String browserName) {
        System.out.println("\n=== Testing Browser: " + browserName + " ===");
        try {
            driver.get("about:blank");
            JavascriptExecutor setupExec = (JavascriptExecutor) driver;

            setupExec.executeScript(
                    "document.body.innerHTML = `" +
                            "<div style='font-family:Segoe UI, sans-serif; padding:20px; background:#f4f7f6;'>" +
                            "  <h2>" + browserName + " Protocol Benchmark</h2>" +
                            "  <div style='display:flex; gap:15px;'>" +
                            "    <div id='classic-box' style='flex:1; padding:15px; background:white; border-left:5px solid #e74c3c;'>Classic HTTP<div id='classic-counter' style='font-size:24px;'>0</div><div id='classic-res'>-</div></div>" +
                            "    <div id='bidi-box' style='flex:1; padding:15px; background:white; border-left:5px solid #2ecc71;'>WebDriver BiDi<div id='bidi-counter' style='font-size:24px;'>0</div><div id='bidi-res'>-</div></div>" +
                            "    <div id='cdp-box' style='flex:1; padding:15px; background:white; border-left:5px solid #3498db;'>CDP (Legacy WS)<div id='cdp-counter' style='font-size:24px;'>0</div><div id='cdp-res'>-</div></div>" +
                            "  </div>" +
                            "</div>`;"
            );

            String handle = driver.getWindowHandle();

            // 1. Classic Benchmark
            long startClassic = System.nanoTime();
            for (int i = 0; i < ITERATIONS; i++) {
                setupExec.executeScript("document.getElementById('classic-counter').innerText = 'Iter: " + (i + 1) + "';");
            }
            long endClassic = System.nanoTime();
            double classicAvg = printOnPage(setupExec, "classic", endClassic - startClassic);

            // 2. BiDi Benchmark
            long startBidi;
            try (Script bidiModule = new Script(driver)) {
                ContextTarget target = new ContextTarget(handle);
                startBidi = System.nanoTime();
                for (int i = 0; i < ITERATIONS; i++) {
                    bidiModule.evaluateFunction(new EvaluateParameters(target, "document.getElementById('bidi-counter').innerText = 'Iter: " + (i + 1) + "';", false));
                }
            }
            long endBidi = System.nanoTime();
            double bidiAvg = printOnPage(setupExec, "bidi", endBidi - startBidi);

            // 3. CDP Benchmark
            double cdpAvg = 0;
            if (driver instanceof HasDevTools hasDevTools) {
                DevTools devTools = hasDevTools.getDevTools();
                devTools.createSession();
                long startCdp = System.nanoTime();
                for (int i = 0; i < ITERATIONS; i++) {
                    devTools.send(org.openqa.selenium.devtools.v143.runtime.Runtime.evaluate("document.getElementById('cdp-counter').innerText = 'Iter: " + (i + 1) + "';",
                            Optional.empty(), Optional.empty(), Optional.empty(), Optional.empty(),
                            Optional.empty(), Optional.empty(), Optional.empty(), Optional.empty(),
                            Optional.empty(), Optional.empty(), Optional.empty(), Optional.empty(),
                            Optional.empty(), Optional.empty(), Optional.empty()));
                }
                long endCdp = System.nanoTime();
                cdpAvg = printOnPage(setupExec, "cdp", endCdp - startCdp);
            } else {
                setupExec.executeScript("document.getElementById('cdp-counter').innerText = 'Unsupported';");
            }

            System.out.printf("[%s] Classic: %.4f | BiDi: %.4f | CDP: %.4f ms/call%n",
                    browserName, classicAvg, bidiAvg, cdpAvg);
        } finally {
            try {
                Thread.sleep(3000);
            } catch (Exception ignored) {
            }
            driver.quit();
        }
    }

    private static double printOnPage(JavascriptExecutor executor, String prefix, long nanoTime) {
        double avg = (nanoTime / 1_000_000.0) / ITERATIONS;
        executor.executeScript("document.getElementById('" + prefix + "-res').innerText = 'Avg: " + String.format("%.4f", avg) + " ms/call';");
        return avg;
    }
}npm version patch -m 'chore: Release v%s' --no-git-tag-version# Security Policy

The Chromium BiDi project takes security very seriously.

## Disclosure of vulnerability

Please use Chromium's process to report security issues.
See https://www.chromium.org/Home/chromium-security/reporting-security-bugs/
