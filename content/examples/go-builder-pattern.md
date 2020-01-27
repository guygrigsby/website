---
title: "Go Builder Pattern"
date: 2020-01-26T16:54:48-07:00
draft: true
---

While you can use a builder pattern in Go, it was brought over from languages like Java. 
<!--more-->
A more idiomatic way to achieve that end would be the options pattern. It can be used much the same way as the builder pattern, but it doesn't obscure what's happening and remains functional in paradigm ( sort of :) ).

https://play.golang.org/p/6f85-hcUQy2
```go

func main() {
	req, err := NewRequest(
		http.MethodPost,
		"localhost:443",
		[]byte("this is the body"),
		WithHeader("X-Powered-by", "Frontdoor"),
		WithHeader("Content-Type", "application/json"),
		WithCookie(&http.Cookie{}),
	)
	if err != nil {
		fmt.Println(err)
	}
	_ = req
}

type Option func(*http.Request)

func NewRequest(method string, uri string, body []byte, options ...Option) (*http.Request, error) {
	r, err := http.NewRequest(method, uri, bytes.NewBuffer(body))
	if err != nil {
		return nil, err
	}
	for _, o := range options {
		o(r)
	}
	return r, nil
}

func WithCookie(c *http.Cookie) Option {
	return func(req *http.Request) {
		req.AddCookie(c)
	}
}
func WithHeader(k, v string) Option {
	return func(req *http.Request) {
		key := http.CanonicalHeaderKey(k)
		req.Header[key] = []string{k}
	}
}
```
