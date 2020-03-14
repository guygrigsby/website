---
title: "Go Scatter Gather Pattern"
date: 2020-01-26T16:18:58-07:00
categories: ["Development"]
tags: ["Development", "Go Patterns", "Concurrency"]

---

One of the strengths of Go is it's native support for concurrency. There are a lot of posts about concurrency in Go, but I was writing this up as a simple example for a coworker and decided to publish it here.
<!--more-->
It takes a little getting used to if you don't use concurrent programming a lot, but once you get beyond a couple hurdles, it's a rockin' good time. I think the term scatter/gather is an old one, but it corresponds to the fan-out/fan-in pattern inf Go. I think it's elegant and works very well. 

```go
package main

import (
	"fmt"
	"io"
	"strings"
)

func main() {
	jobs := []int64{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

	routines := 3

	scatterGather(routines, jobs)
}

func scatterGather(concurrency int, jobs []int64) {
	errors := make(chan error, 0)
	results := make(chan string, 0)

	// Scatter
	for i := 0; i < len(jobs); i += concurrency {
		go func(i int) {
			end := i + concurrency
			if end > len(jobs) {
				end = len(jobs)
			}

			res, err := process(jobs[i:end])
			if err != nil {
				errors <- err
				return
			}
			results <- res

		}(i)
	}
	// Gather
	for i := 0; i < len(jobs); i += concurrency {
		select {
		case err := <-errors:
			fmt.Println("Error", err)
		case result := <-results:
			fmt.Println("Success", result)
		}
	}

}

// just some trash to make strings and errors
func process(jobs []int64) (string, error) {
	if len(jobs) == 0 {
		return "", io.EOF
	}
	if jobs[0]%2 == 0 {
		return "", fmt.Errorf("Fake Error")
	}
	var sb strings.Builder
	for _, n := range jobs {
		sb.WriteString(fmt.Sprintf("%d", n))
	}
	return sb.String(), nil

}
```

For more about Go concurrency patterns check out [pipelines](https://blog.golang.org/pipelines) or stay tuned!
