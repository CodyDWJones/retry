# retry

[![Build Status](https://travis-ci.org/Rican7/retry.svg?branch=master)](https://travis-ci.org/Rican7/retry)
[![Coverage Status](https://coveralls.io/repos/github/Rican7/retry/badge.svg)](https://coveralls.io/github/Rican7/retry)
[![Go Report Card](https://goreportcard.com/badge/Rican7/retry)](http://goreportcard.com/report/Rican7/retry)
[![GoDoc](https://godoc.org/github.com/CodyDWJones/retry?status.png)](https://godoc.org/github.com/CodyDWJones/retry)
[![Latest Stable Version](https://img.shields.io/github/release/Rican7/retry.svg?style=flat)](https://github.com/CodyDWJones/retry/releases)

A simple, stateless, functional mechanism to perform actions repetitively until successful.


## Project Status

This project is currently in "pre-release". While the code is heavily tested, the API may change.
Vendor (commit or lock) this dependency if you plan on using it.


## Install

`go get github.com/CodyDWJones/retry`


## Examples

### Basic

```go
retry.Retry(func(attempt uint) error {
	return nil // Do something that may or may not cause an error
})
```

### File Open

```go
const logFilePath = "/var/log/myapp.log"

var logFile *os.File

err := retry.Retry(func(attempt uint) error {
	var err error

	logFile, err = os.Open(logFilePath)

	return err
})

if nil != err {
	log.Fatalf("Unable to open file %q with error %q", logFilePath, err)
}

logFile.Chdir() // Do something with the file
```

### HTTP request with strategies and backoff

```go
action := func(ctx context.Context, attempt uint) error {
	var response *http.Response

	req, err := NewRequestWithContext(ctx, "GET", "https://api.github.com/repos/Rican7/retry", nil)
	if err == nil {
		response, err = c.Do(req)
	}

	if nil == err && nil != response && response.StatusCode > 200 {
		err = fmt.Errorf("failed to fetch (attempt #%d) with status code: %d", attempt, response.StatusCode)
	}

	return err
}

err := retry.RetryWithContext(
	context.TODO(),
	action,
	strategy.Limit(5),
	strategy.Backoff(backoff.Fibonacci(10*time.Millisecond)),
)

if nil != err {
	log.Fatalf("Failed to fetch repository with error %q", err)
}
```

### Retry with backoff jitter

```go
action := func(attempt uint) error {
	return errors.New("something happened")
}

seed := time.Now().UnixNano()
random := rand.New(rand.NewSource(seed))

retry.Retry(
	action,
	strategy.Limit(5),
	strategy.BackoffWithJitter(
		backoff.BinaryExponential(10*time.Millisecond),
		jitter.Deviation(random, 0.5),
	),
)
```
