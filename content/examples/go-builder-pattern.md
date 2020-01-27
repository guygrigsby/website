---
title: "Go Builder Pattern"
date: 2020-01-26T16:54:48-07:00
---

I came from the Java world so I am familiar with the Java builder pattern. I didn't use it much, but I reviewed some Go code recently that used a builder. It was written by a former Java dev and I have been thinking about it.

I came to the realization that, while you can use a builder pattern in Go, it really isn't idiomatic. 

A more idiomatic way to do the same thing is the options pattern. It's used much the same way as the builder pattern.

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
	r, err := http.NewRequest(method, uri, bytes.NewBuffer(body)) // already returns a pointer
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

More on [functional options in Go](https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis) from Dave Cheney.
