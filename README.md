# UI Test Suite - Robot Framework in Docker Alpine, with Firefox and Chrome

## What is it? :question:
 
This project consists of a Docker image :whale2: containing a complete Robot Framework :robot: installation, and a sample UI test suite for eCommerce website, running on Alpine Linux :rocket:, a lightweight linux distro.

This installation also contains Firefox, Chrome and the Selenium library for Robot Framework.

## Versioning

The versioning of this image are as follows:

* [Robot Framework](https://github.com/robotframework/robotframework) 3.2.2
* [Robot Framework DatabaseLibrary](https://github.com/franz-see/Robotframework-Database-Library) 1.2
* [Robot Framework Datadriver](https://github.com/Snooz82/robotframework-datadriver) 1.0.0
* [Robot Framework DateTimeTZ](https://github.com/testautomation/DateTimeTZ) 1.0.6
* [Robot Framework Faker](https://github.com/guykisel/robotframework-faker) 5.0.0
* [Robot Framework FTPLibrary](https://github.com/kowalpy/Robot-Framework-FTP-Library) 1.9
* [Robot Framework IMAPLibrary 2](https://pypi.org/project/robotframework-imaplibrary2/) 0.3.8
* [Robot Framework Pabot](https://github.com/mkorpela/pabot) 1.10.0
* [Robot Framework Requests](https://github.com/bulkan/robotframework-requests) 0.7.2
* [Robot Framework SeleniumLibrary](https://github.com/robotframework/SeleniumLibrary) 4.5.0
* [Robot Framework SSHLibrary](https://github.com/robotframework/SSHLibrary) 3.5.1
* Firefox ESR 78
* Chromium 86.0
* [Amazon AWS CLI](https://pypi.org/project/awscli/) 1.18.200

## Video of test execution :tv:

https://vimeo.com/504810311

## Running the Docker container :ballot_box_with_check:

1) Install Docker Desktop: https://www.docker.com/products/docker-desktop

2) Build the docker image with the following command in PowerShell, your current folder should be the same that holds the Dockerfile:

docker build -t dalefixter/robot-framework/westwing:latest .

3) Once the image is built. You need to run it. Type the following command:

docker run dalefixter/robot-framework/westwing:latest

4) Create one found on your local machine for Reports, suggest C:\reports

5) You can proceed to run the robot tests, run the following command in the Powershell:

docker run -v {$file folder where you want to place reports}:/opt/robotframework/reports:Z -v {$file folder of the testcases folder in this test pack}:/opt/robotframework/tests:Z -v {$file folder of the Resources folder in this test pack}:/opt/robotframework/Resources:Z -v {$file folder of the PageObjects folder in this test pack}:/opt/robotframework/PageObjects:Z -e BROWSER=chrome dalefixter/robot-framework/westwing

replace {$file folder****} with the local path of the same folder on your local machine.

An example is given below:

docker run -v c:\reports:/opt/robotframework/reports:Z -v C:\docker-robotframework-westwing\testfiles\TestCases:/opt/robotframework/tests:Z -v C:\docker-robotframework-westwing\testfiles\Resources:/opt/robotframework/Resources:Z -v C:\docker-robotframework-westwing\testfiles\PageObjects:/opt/robotframework/PageObjects:Z -e BROWSER=chrome dalefixter/robot-framework/westwing:latest

### Switching browsers

Browsers can be easily switched. It is recommended to define `${BROWSER} %{BROWSER}` in your Robot variables and to use `${BROWSER}` in your test cases. This allows to set the browser in a single place if needed.

When running your tests, simply add `-e BROWSER=chrome` or `-e BROWSER=firefox` to the run command.

### Changing the container's screen resolution

It is possible to define the settings of the virtual screen in which the browser is run by changing several environment variables:

* `SCREEN_COLOUR_DEPTH` (default: 24)
* `SCREEN_HEIGHT` (default: 1080)
* `SCREEN_WIDTH` (default: 1920)

### Changing the container's tests and reports directories

It is possible to use different directories to read tests from and to generate reports to. This is useful when using a complex test file structure. To change the defaults, set the following environment variables:

* `ROBOT_REPORTS_DIR` (default: /opt/robotframework/reports)
* `ROBOT_TESTS_DIR` (default: /opt/robotframework/tests)

### Parallelisation

It is possible to parallelise the execution of your test suites. Simply define the `ROBOT_THREADS` environment variable, for example:

    docker run \
        -e ROBOT_THREADS=4 \
        dalefixter/robot-framework/westwing:latest

By default, there is no parallelisation.

#### Parallelisation options

When using parallelisation, it is possible to pass additional [pabot options](https://github.com/mkorpela/pabot#command-line-options), such as `--testlevelsplit`, `--argumentfile`, `--ordering`, etc. These can be passed by using the `PABOT_OPTIONS` environment variable, for example:

    docker run \
        -e ROBOT_THREADS=4 \
        -e PABOT_OPTIONS="--testlevelsplit" \
        dalefixter/robot-framework/westwing:latest

### Passing additional options

RobotFramework supports many options such as `--exclude`, `--variable`, `--loglevel`, etc. These can be passed by using the `ROBOT_OPTIONS` environment variable, for example:

    docker run \
        -e ROBOT_OPTIONS="--loglevel DEBUG" \
        dalefixter/robot-framework/westwing:latest

### Testing emails

This project includes the IMAP library which allows Robot Framework to connect to email servers.

A suggestion to automate email testing is to run a [Mailcatcher instance in Docker which allows IMAP connections](https://github.com/estelora/docker-mailcatcher-imap). This will ensure emails are discarded once the tests have been run.

### Dealing with Datetimes and Timezones

This project is meant to allow your tests to run anywhere. Sometimes that can be in a different timezone than your local one or of the location under test. To help solve such issues, this image includes the [DateTimeTZ Library](https://testautomation.github.io/DateTimeTZ/doc/DateTimeTZ.html).

To set the timezone used inside the Docker image, you can set the `TZ` environment variable:

    docker run \
        -e TZ=America/New_York \
        dalefixter/robot-framework/westwing:latest

## Security consideration

By default, containers are implicitly run using `--user=1000:1000`, please remember to adjust that command-line setting accordingly, for example:

    docker run \
        --user=1001:1001 \
        dalefixter/robot-framework/westwing:latest

Remember that that UID/GID should be allowed to access the mounted volumes in order to read the test suites and to write the output.

Additionally, it is possible to rely on user namespaces to further secure the execution. This is well described in the official container documentation:

* Docker: [Introduction to User Namespaces in Docker Engine](https://success.docker.com/article/introduction-to-user-namespaces-in-docker-engine)
* Podman: [Running rootless Podman as a non-root user](https://www.redhat.com/sysadmin/rootless-podman-makes-sense)

This is a good security practice to make sure containers cannot perform unwanted changes on the host. In that sense, Podman is probably well ahead of Docker by not relying on a root daemon to run its containers.

## Continuous integration

It is possible to run the project from within a Jenkins pipeline by relying on the shell command line directly:

    pipeline {
        agent any
        stages {
            stage('Functional regression tests') {
                steps {
                    sh "docker run --shm-size=1g -e BROWSER=firefox -v $WORKSPACE/robot-tests:/opt/robotframework/tests:Z -v $WORKSPACE/robot-reports:/opt/robotframework/reports:Z dalefixter/robot-framework/westwing:latest"
                }
            }
        }
    }

The pipeline stage can also rely on a Docker agent, as shown in the example below:

    pipeline {
        agent none
        stages {
            stage('Functional regression tests') {
                agent { docker {
                    image 'dalefixter/robot-framework/westwing:latest'
                    args '--shm-size=1g -u root' }
                }
                environment {
                    BROWSER = 'firefox'
                    ROBOT_TESTS_DIR = "$WORKSPACE/robot-tests"
                    ROBOT_REPORTS_DIR = "$WORKSPACE/robot-reports"
                }
                steps {
                    sh '''
                        /opt/robotframework/bin/run-tests-in-virtual-screen.sh
                    '''
                }
            }
        }
    }

### Defining a test run ID

When relying on Continuous Integration tools, it can be useful to define a test run ID such as the build number or branch name to avoid overwriting consecutive execution reports.

For that purpose, the `ROBOT_TEST_RUN_ID` variable was introduced:
* If the test run ID is empty, the reports folder will be: `${ROBOT_REPORTS_DIR}/`
* If the test run ID was provided, the reports folder will be: `${ROBOT_REPORTS_DIR}/${ROBOT_TEST_RUN_ID}/`

It can simply be passed during the execution, such as:

    docker run \
        -e ROBOT_TEST_RUN_ID="feature/branch-name" \
        dalefixter/robot-framework/westwing:latest

By default, the test run ID is empty.

### Upload test reports to an AWS S3 bucket

To upload the report of a test run to an S3 bucket, you need to define the following environment variables:
    
    docker run \
        -e AWS_ACCESS_KEY_ID=<your AWS key> \
        -e AWS_SECRET_ACCESS_KEY=<your AWS secret> \
        -e AWS_DEFAULT_REGION=<your AWS region e.g. eu-central-1> \
        -e AWS_BUCKET_NAME=<name of your S3 bucket> \
        dalefixter/robot-framework/westwing:latest

## Troubleshooting

### Chromium is crashing

Chrome drivers might crash due to the small size of `/dev/shm` in the docker container:
> UnknownError: session deleted because of page crash

This is [a known bug of Chromium](https://bugs.chromium.org/p/chromium/issues/detail?id=715363).

To avoid this error, please change the shm size when starting the container by adding the following parameter: `--shm-size=1g` (or any other size more suited to your tests)

### Accessing the logs

In case further investigation is required, the logs can be accessed by mounting their folder. Simply add the following parameter to your `run` command:

* Linux/Mac: ``-v `pwd`/logs:/var/log:Z``
* Windows: ``-v ${PWD}/logs:/var/log:Z``

Chromium allows to set additional environment properties, which can be useful when debugging:

* `webdriver.chrome.verboseLogging=true`: enables the verbose logging mode
* `webdriver.chrome.logfile=/path/to/chromedriver.log`: sets the path to Chromium's log file

### Error: Suite contains no tests

When running tests, an unexpected error sometimes occurs:

> [Error] Suite contains no tests.

There are two main causes to this:
* Either the test folder is not the right one,
* Or the permissions on the test folder/test files are too restrictive.

As there can sometimes be issues as to where the tests are run from, make sure the correct folder is used by trying the following actions:
* Use a full path to the folder instead of a relative one,
* Replace any`` `pwd` ``or `${PWD}` by the full path to the folder.

It is also important to check if Robot Framework is allowed to access the resources it needs, i.e.:
* The folder where the tests are located,
* The test files themselves.

### Database tests are failing in spite of the DatabaseLibrary being present

As per their official project page, the [Robot Framework DatabaseLibrary](https://github.com/franz-see/Robotframework-Database-Library) contains utilities meant for Robot Framework's usage. This can allow you to query your database after an action has been made to verify the results. This is compatible with any Database API Specification 2.0 module.

It is anyway mandatory to extend the container image to install the specific database module relevant to your tests, such as:
* [MS SQL](https://pymssql.readthedocs.io/en/latest/intro.html): `pip install pymssql`
* [MySQL](https://dev.mysql.com/downloads/connector/python/): `pip install pymysql`
* [Oracle](https://www.oracle.com/uk/database/technologies/appdev/python.html): `pip install py2oracle`
* [PostgreSQL](http://pybrary.net/pg8000/index.html): `pip install pg8000`
