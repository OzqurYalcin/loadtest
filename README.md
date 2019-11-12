# Simple HTTP load test

```go
package main

import (
	"encoding/json"
	"fmt"
	"net/url"
	"os"
	"strconv"
	"time"

	vegeta "github.com/tsenart/vegeta/lib"
)

func main() {
	frequency, err := strconv.Atoi(os.Args[1])
	if err != nil {
		panic(err)
	}
	duration, err := time.ParseDuration(os.Args[2])
	if err != nil {
		panic(err)
	}
	target, err := url.Parse(os.Args[3])
	if err != nil {
		panic(err)
	}
	targeter := vegeta.NewStaticTargeter(vegeta.Target{Method: "GET", URL: target.String()})
	rate := vegeta.Rate{Freq: frequency, Per: time.Second}
	attacker := vegeta.NewAttacker()
	var metrics vegeta.Metrics
	for res := range attacker.Attack(targeter, rate, duration, "Big Bang!") {
		metrics.Add(res)
	}
	metrics.Close()
	fmt.Println("target:", target)
	fmt.Println("request rate:", metrics.Rate)
	fmt.Println("duration:", metrics.Duration.Seconds())
	fmt.Println("total requests:", metrics.Requests)
	fmt.Println("mean latency:", metrics.Latencies.Mean.Seconds())
	fmt.Println("max latency:", metrics.Latencies.Max.Seconds())
	fmt.Println("total bytes:", metrics.BytesIn.Total)
	fmt.Println("mean bytes:", metrics.BytesIn.Mean)
	fmt.Println("success ratio:", metrics.Success)
	statuscodes, _ := json.MarshalIndent(metrics.StatusCodes, " ", " ")
	fmt.Println("status codes:", string(statuscodes))
	errors, _ := json.MarshalIndent(metrics.Errors, " ", " ")
	fmt.Println("errors:", string(errors))
}
```

```shell
$ go get github.com/tsenart/vegeta
$ go run main.go 50 5s http://www.example.com/
```