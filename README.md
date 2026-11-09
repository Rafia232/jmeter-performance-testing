# JMeter Load Testing

A collection of Apache JMeter test plans for performance, load, and API
testing across multiple projects.

Each project should have its own test plans, test data, and test
results. This keeps scenarios organized and makes it easier to maintain
and compare test runs.

## Repository Structure

Suggested structure:

``` text
.
├── README.md
├── projects/
│   ├── project-name/
│   │   ├── test-plans/
│   │   ├── test-data/
│   │   └── results/
│   └── another-project/
│       ├── test-plans/
│       ├── test-data/
│       └── results/
└── docs/
    └── test-notes.md
```

Adjust the structure as more projects and test scenarios are added.

## Requirements

-   Apache JMeter 5.6.3 (or the version required by the test plan).
-   A compatible Java installation.
-   Access to the target test environment.
-   Valid test accounts and approved test data.
-   A browser such as Chrome, Edge, or Firefox for recording user
    activity.

## 1. Open or Create a JMeter Test Plan

1.  Open the JMeter `bin` folder.
2.  Run `jmeter.bat` on Windows.
3.  Open an existing `.jmx` file using **File → Open**, or create a new
    test plan.
4.  Save the test plan in the relevant project's `test-plans` folder.

## 2. Add a Thread Group

1.  Right-click **Test Plan**.
2.  Select **Add → Threads (Users) → Thread Group**.
3.  Configure the required load settings:

  ---------------------------------------------------------------------
  Setting                            Purpose
  ---------------------------------- ----------------------------------
  Number of Threads (users)          Number of virtual users

  Ramp-up Period (seconds)           Time JMeter takes to start all
                                     virtual users

  Loop Count                         Number of times each user executes
                                     the scenario

  Duration                           Test duration, if using a
                                     duration-based test
  ---------------------------------------------------------------------

Use settings appropriate for the scenario. Start with a small load to
confirm that the test works before increasing the number of users.

## 3. Add the HTTP(S) Test Script Recorder

The recorder captures browser requests and adds them to the JMeter test
plan.

1.  Right-click **Test Plan**.
2.  Select **Add → Non-Test Elements → HTTP(S) Test Script Recorder**.
3.  Select the recorder in the left panel.
4.  Under **Global Settings**, set the port to `8888`.
5.  Under **Requests Filtering**, optionally add URL patterns to exclude
    static files such as images, fonts, CSS, and JavaScript.
6.  Under **Test Script Recorder**, select the Thread Group or a
    Recording Controller as the target controller.

### Optional: Add a Recording Controller

A Recording Controller keeps captured requests grouped in one place.

1.  Right-click the Thread Group.
2.  Select **Add → Logic Controller → Recording Controller**.
3.  Give it a clear name, such as `Recorded User Flow`.
4.  Return to the HTTP(S) Test Script Recorder.
5.  Set **Target Controller** to the Recording Controller you created.

## 4. Configure the Browser Proxy

JMeter must receive browser traffic through its recording proxy.

On Windows:

1.  Open **Settings**.
2.  Go to **Network & internet → Proxy**.
3.  Find **Manual proxy setup** and click **Edit**.
4.  Turn on **Use a proxy server**.
5.  Enter these values:

  ---------------------------------------------------------------------
  Field                              Value
  ---------------------------------- ----------------------------------
  Proxy IP address                   `127.0.0.1`

  Port                               `8888`

  Exceptions                         Leave blank

  Don't use the proxy server for     Leave unchecked
  local (intranet) addresses         
  ---------------------------------------------------------------------

6.  Click **Save**.

The settings should match this configuration:

``` text
Use a proxy server: On
Proxy IP address:   127.0.0.1
Port:               8888
Exceptions:         (blank)
Don't use proxy for local addresses: Unchecked
```

Keep JMeter's recorder port and the browser proxy port the same. If you
change the recorder port, update the browser proxy port too.

## 5. Start the Recorder and Generate the JMeter Certificate

HTTPS websites require the JMeter certificate to be trusted by the
browser. Otherwise, the browser may show a security warning or HTTPS
requests may not be recorded correctly.

1.  Return to JMeter.
2.  Select **HTTP(S) Test Script Recorder**.
3.  Click **Start**.
4.  If prompted, allow JMeter to create its temporary certificate.
5.  Open the JMeter installation's `bin` folder:

``` text
apache-jmeter-5.6.3\apache-jmeter-5.6.3\bin
```

6.  Find the generated certificate:

``` text
ApacheJMeterTemporaryRootCA.crt
```

If the certificate is not present, make sure the recorder has been
started. JMeter generates the temporary certificate when the recorder is
initialized.

## 6. Import the Certificate into the Browser

Import the certificate as a trusted certificate authority so the browser
can record HTTPS traffic through JMeter.

### Google Chrome or Microsoft Edge

1.  Keep the JMeter recorder running.
2.  Open Chrome or Edge.
3.  Open **Settings**.
4.  Go to **Privacy and security → Security**.
5.  Open **Manage certificates**.
6.  Open the **Authorities** or trusted root certificate section.
7.  Click **Import**.
8.  Browse to the certificate file:

``` text
apache-jmeter-5.6.3\apache-jmeter-5.6.3\bin\ApacheJMeterTemporaryRootCA.crt
```

9.  Select the certificate and continue through the import wizard.
10. If asked where to place the certificate, choose **Trusted Root
    Certification Authorities**.
11. Confirm the import and close the certificate window.
12. Restart the browser if required.

Depending on the browser version, certificate management may open the
Windows certificate manager. In that case, import the certificate into
**Trusted Root Certification Authorities → Certificates** for the
current user.

### Mozilla Firefox

1.  Open Firefox.
2.  Go to **Settings → Privacy & Security**.
3.  Scroll to **Certificates**.
4.  Click **View Certificates**.
5.  Open the **Authorities** tab.
6.  Click **Import**.
7.  Select `ApacheJMeterTemporaryRootCA.crt` from the JMeter `bin`
    folder.
8.  Enable **Trust this CA to identify websites**.
9.  Confirm the import.

After importing the certificate, keep the recorder running while
recording browser activity.

> **Security note:** This certificate allows JMeter to inspect HTTPS
> traffic routed through its proxy. Import it only when needed for
> authorized testing. Remove the temporary certificate from the browser
> or trusted certificate store when recording is finished.

## 7. Record Browser Activity

1.  Confirm that the JMeter recorder is running.
2.  Confirm that the browser proxy is enabled with IP `127.0.0.1` and
    port `8888`.
3.  Open the target website in the browser.
4.  Perform the user flow you want to test, for example:
    -   Log in.
    -   Open a page or module.
    -   Search for a record.
    -   Create or update a record.
    -   Submit or approve an entry.
5.  Check JMeter. The captured requests should appear under the selected
    Recording Controller or Thread Group.
6.  Complete the flow, then return to JMeter.
7.  Click **Stop** in the HTTP(S) Test Script Recorder.

Record only the actions needed for the scenario. Avoid capturing
unrelated browsing activity.

## 8. Clean Up and Prepare the Recorded Script

A recorded script usually needs adjustments before it can be used for
load testing.

1.  Remove unnecessary requests, duplicate calls, and unrelated traffic.
2.  Rename requests so their purpose is clear.
3.  Check HTTP methods, paths, headers, and request bodies.
4.  Add **HTTP Request Defaults** for common server and protocol
    settings.
5.  Add **HTTP Header Manager** for required headers.
6.  Handle authentication correctly. Do not leave expired or personal
    tokens in the test plan.
7.  Replace fixed values that change between users or iterations with
    JMeter variables.
8.  Add **CSV Data Set Config** when requests need different IDs or
    records.
9.  Add timers only when the scenario requires pacing or
    synchronization.
10. Add assertions to validate important response codes or response
    content.
11. Save the updated `.jmx` file.

For example, a fixed record ID can be replaced with a variable such as
`${recordId}`, with values supplied by a CSV file.

## 9. Restore Browser Proxy Settings

After recording:

1.  Return to **Windows Settings → Network & internet → Proxy**.
2.  Open **Manual proxy setup → Edit**.
3.  Turn **Use a proxy server** off.
4.  Click **Save**.

This prevents normal browser traffic from continuing to pass through
JMeter when recording is not required.

Also remove the temporary JMeter certificate from the browser or trusted
certificate store when it is no longer needed.

## 10. Run a Load Test

### Using the JMeter GUI

1.  Open the required `.jmx` test plan.
2.  Confirm the target environment and test data.
3.  Review the Thread Group settings.
4.  Check which samplers and controllers are enabled.
5.  Click **Start**.
6.  Review the results using the configured listeners.

### Using the Command Line

Non-GUI mode is recommended for larger load tests.

``` bash
jmeter -n \
  -t "projects/project-name/test-plans/test-plan.jmx" \
  -l "projects/project-name/results/results.jtl" \
  -e \
  -o "projects/project-name/results/html-report"
```

Replace the example paths with the paths for the project you want to
test. The HTML report output directory should be new or empty before the
run.

## 11. JMeter Components

Depending on the project, a test plan may include:

-   **Thread Group** --- configures virtual users, ramp-up time,
    duration, and loop count.
-   **HTTP Request Defaults** --- defines common protocol, server, and
    port settings.
-   **HTTP Header Manager** --- configures request headers and
    authentication.
-   **HTTP Cookie Manager** --- manages cookies during requests.
-   **HTTP Cache Manager** --- manages browser-like cache behavior.
-   **CSV Data Set Config** --- supplies dynamic test data such as
    customer IDs or record IDs.
-   **Timers** --- control request timing or synchronize concurrent
    requests.
-   **Transaction Controllers** --- group related requests into business
    workflows.
-   **HTTP Request Samplers** --- execute API requests using methods
    such as GET and POST.
-   **Listeners and reports** --- review response times, throughput,
    errors, and other test metrics.
-   **JSR223 elements** --- apply custom test logic when required.
-   **HTTP(S) Test Script Recorder** --- captures browser requests for
    script creation.

Not every test plan will use all of these components.

## 12. Results and Reporting

Review the results to understand:

-   Total number of requests and throughput.
-   Average, minimum, and maximum response times.
-   90th, 95th, and 99th percentile response times, when available.
-   Error count and error percentage.
-   HTTP response codes.
-   Behavior under concurrent load.

Save the `.jtl` result file and, when generated, the HTML dashboard in
the relevant project's `results` directory. Record the test
configuration and environment alongside the results so that runs can be
compared reliably.

## 13. Adding Another Project

1.  Create a folder under `projects/` using the project name.
2.  Add its JMeter `.jmx` files under `test-plans/`.
3.  Place required CSV files and supporting test data under
    `test-data/`.
4.  Keep generated results under `results/`.
5.  Add project-specific setup notes or scenarios to a project-level
    README if needed.

## Security and Test Data

-   Do not commit passwords, API keys, bearer tokens, cookies, or other
    credentials.
-   Keep secrets outside the JMX file where possible, and supply them
    through secure local configuration or runtime properties.
-   Use anonymized or approved test data.
-   Check and update local CSV file paths before sharing a test plan.
-   Do not run load tests against production or third-party systems
    without authorization.
-   Be careful with requests that create, submit, approve, or delete
    records. Use a dedicated test environment and understand the data
    impact before running them.
-   Do not commit generated certificates, private keys, or browser
    profile data.

------------------------------------------------------------------------

**Note:** Test plans are project-specific. Always review the request
payloads, test data, enabled samplers, authentication, and load settings
before execution.
