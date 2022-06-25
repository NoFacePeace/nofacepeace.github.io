---
title: go error wrap
date: 2022-06-14 16:02:12
tags:
---

```go
package main

import (
	"errors"
	"fmt"
)

// HTTPError http error struct
type HTTPError struct {
	Status int
	Msg    string
}

func (h *HTTPError) Error() string {
	return fmt.Sprintf("status: %d Message: %s", h.Status, h.Msg)
}

// NewHTTPError new http error
func NewHTTPError(status int, msg string) *HTTPError {
	return &HTTPError{
		Status: status,
		Msg:    msg,
	}
}

// HTTP error class
var (
	ErrBadRequest     = NewHTTPError(400, "Bad Request")
	ErrGatewayTimeout = NewHTTPError(503, "Gateway Timeout")
)

// API struct
type API struct {
}

// NewAPI new api
func NewAPI() *API {
	return &API{}
}

// Get api get
func (a *API) Get() error {
	return fmt.Errorf("method get error: %w", ErrBadRequest)
}

// Post api post
func (a *API) Post() error {
	return fmt.Errorf("method post error: %w", ErrGatewayTimeout)
}

func main() {
	api := NewAPI()
	err := api.Get()
	fmt.Println(err)
	// check http error
	var httpError *HTTPError
	fmt.Println(errors.As(err, &httpError))
	// check bad request
	fmt.Println(errors.Is(err, ErrBadRequest))
	// unwrap
	fmt.Println(errors.Unwrap(err))
}

```

## 参考

- Working with Errors in Go 1.13：https://go.dev/blog/go1.13-errors
