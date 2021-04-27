# WebDriverClient
[![codecov](https://codecov.io/gh/aaronrenner/web_driver_client/branch/master/graph/badge.svg)](https://codecov.io/gh/aaronrenner/web_driver_client)

A low-level [WebDriver] client for Elixir. This library is still a work in progress.

## Overview

WebDriverClient is designed to be a low-level library that allows projects to call WebDriver
REST APIs while abstracting away the differences between the [JWP] and [W3C] protocols. This
library is designed to be the API client for higher-level libraries, like [Wallaby] or [Hound].

```elixir
{:ok, session} =
  "http://localhost:9515"
  |> WebDriverClient.Config.build(protocol: :w3c)
  |> WebDriverClient.start_session(%{"capabilities" => %{}})


:ok = WebDriverClient.navigate_to(session, "http://dockyard.com")

{:ok, element} = WebDriverClient.find_element(session, :css_selector, ".site-nav__logo__link")

WebDriverClient.fetch_element_text(session, element) # => {:ok, "DockYard Home"}

:ok = WebDriverClient.end_session(session)
```

### Design considerations
* Should be a thin, well-documented API client that calls the WebDriver REST APIs.
* Should provide a main API that abstracts away the differences between the JWP and W3C
  protocols.
* (Future) Should provide protocol-specific APIs as an escape-hatch to access functionality
  that is not common to both protocols.
* Sometimes the user will request one protocol and the server returns the other protocol. This
  happens if the user sends the incorrect payload on session start, or requests an
  endpoint that is not tied to an individual session (like listing sessions).

  This library should gracefully handle these situations, while still notifying the user
  that the wrong protocol was returned.

### What this library is not
* This is not a high-level library to be used for day to day
  testing. Features like retries, pipeline commands, etc
  are outside the scope of this project. These features are better
  delegated to a high-level library like [Wallaby] or [Hound].



## Testing

In order to ensure the reliability of this library, test coverage is very
important. This comes through the following types of tests.

### Unit test

The unit test suite can be run with the following command.

```
mix test
```

### Integration tests

Although each webdriver implementation should be the same in theory, it's good to double-check that this library actually works. The following webdriver implementations are supported via the test suite:

* `chromedriver`
* `phantomjs`
* `selenium_2`
* `selenium_3`

Note: This project does not start any WebDriver program. Those need to be started separately.

#### ChromeDriver

To install ChromeDriver on OS X:

```
$ brew cask install chromedriver
```

Before running test suite, start up chromedriver

```
$ chomedriver
```

Then to run the integration tests, run:

```
$ mix test --only integration_test_driver:chromedriver
```

#### PhantomJS

To install PhantomJS on OS X:

```
$ brew cask install phantomjs
```

Before running test suite, start up phantomjs

```
$ phantomjs --wd
```

Then to run the integration tests, run:

```
$ mix test --only integration_test_driver:phantomjs
```

#### Selenium

To install ChromeDriver on OS x:

```
$ brew install selenium-server-standalone
```

You also need to install the webdriver servers

For firefox:
```
$ brew install geckodriver
```

For chrome:
```
$ brew install chromedriver
```

Before running test suite, start up selenium

```
$ selenium-server
```

Then to run the integration tests, run:

```
$ mix test --only integration_test_driver:selenium_3
```

If you'd like to run only the selenium tests for chrome, run:

```
$ mix test --only integration_test_driver_browser:selenium_3-chrome
```

And to only run selenium tests for firefox, run:

```
$ mix test --only integration_test_driver_browser:selenium_3-firefox
```

##### Selenium 2
If you'd like to run integration tests for `selenium_2`, just replace `selenium_3` with
`selenium_2`.

```
$ mix test --only integration_test_driver:selenium_2
```

#### Running on remote webdriver servers

Sometimes it's nice to be able to run against a remote webdriver server.
Here's an example that runs the tests against a docker container.

1. Start the docker container for the webdriver server

    ```
    $ docker run -p 4446:4444 --shm-size=2g selenium/standalone-chrome:3
    ```

2. Run the tests

    ```
    $ SELENIUM_3_BASE_URL="http://localhost:4446/wd/hub" TEST_SERVER_HOSTAME="host.docker.internal" mix test --only integration_test_driver_browser:selenium_3-chrome
    ```

    The environment variables work like this:

    * `<DRIVER_NAME>_BASE_URL` - The base url for the webdriver server to run
       against. Can also use `WEBDRIVER_BASE_URL` to set this across all scenarios.
    * ` TEST_SERVER_HOSTNAME` - The hostname the webdriver server should use to access
      the machine that's running the test suite.

        The test suite starts up a test HTTP server, but when using a remote webdriver
        server, the webdriver server needs to where the test pages live. When running in
        docker for Mac, the hostname of the host computer is `host.docker.internal`.



## Installation

If [available in Hex](https://hex.pm/docs/publish), the package can be installed
by adding `web_driver_client` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [
    {:web_driver_client, "~> 0.1.0"}
  ]
end
```

Documentation can be generated with [ExDoc](https://github.com/elixir-lang/ex_doc)
and published on [HexDocs](https://hexdocs.pm). Once published, the docs can
be found at [https://hexdocs.pm/web_driver_client](https://hexdocs.pm/web_driver_client).

## Documentation

Documentation can be built locally with the following command.

```sh
$ mix docs
```

To view documentation for pre-release APIs, docs can be built
with:

```sh
$ MIX_ENV=docs_prerelease mix docs
```

It's important to note the pre-release APIs aren't public and
may change at any time.

[WebDriver]: https://w3c.github.io/webdriver/
[JWP]: https://github.com/SeleniumHQ/selenium/wiki/JsonWireProtocol
[W3C]: https://w3c.github.io/webdriver/
[Wallaby]: https://github.com/elixir-wallaby/wallaby
[Hound]: https://github.com/HashNuke/hound
