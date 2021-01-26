# Metrics

## Download dependencies for the program
```bash
$ go get -d github.com/prometheus/client_golang/prometheus
```

## Running the app
```bash
$ go run expose-app.go
```

## Access Metrics
```bash
$ curl -s http://localhost:8000/metrics
```

## Perform lint checks for metrics using command line
```bash
$ curl -s http://localhost:8000/metrics | promtool check metrics
hello_worlds_totl counter metrics should have "_total" suffix
$
```

