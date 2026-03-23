---
title: "Echo middlewares"
date: 2021-11-26
categories: ["Go"]
---

In this brief article, I aim to highlight an ideal use-case for middleware, shedding light on its functionality and practical implementation.

Middleware serves as a crucial intermediary between user requests and application logic. In the following example, I'll demonstrate how middleware can authenticate users and seamlessly pass custom context to a micro-service, streamlining the authentication process without burdening the service with additional authentication logic.

<!--more-->

## Let's Dive into the Code

Creating a custom context is straightforward. We begin by defining a new structure that embeds `echo.Context`, ensuring seamless implementation of the `Context` interface. While exporting only `userID` could suffice, embedding `echo.Context` offers flexibility and avoids potential conflicts.

```go
type UserContext struct {
	echo.Context
	userID int64
}

func (c *UserContext) GetUserID() int64 {
	return c.userID
}
```

Next, we implement the Middleware function, which retrieves and returns `HandlerFunc`, adhering to the standard middleware pattern in Echo.
```go
type (
  MiddlewareFunc func(HandlerFunc) HandlerFunc
  HandlerFunc func(Context) error
)
```

Here's how we implement the middleware function, requiring an `authenticator` to perform the actual authentication.

```go
type authenticator interface {
	AuthenticateUser(ctx context.Context, token string) (int64, error)
}

func AuthenticateUserMiddleware(authenticator authenticator) echo.MiddlewareFunc {
	return func(next echo.HandlerFunc) echo.HandlerFunc {
		return func(c echo.Context) error {
			// Retrieve token from the request
			authToken := c.Request().Header.Get("Authorization")
			if authToken == "" {
				return echo.NewHTTPError(http.StatusUnauthorized, "No authorization token")
			}

			// Authenticate the user and retrieve the user ID
			userID, err := authenticator.AuthenticateUser(c.Request().Context(), authToken)
			if err != nil {
				return echo.NewHTTPError(http.StatusUnauthorized, "Invalid authorization token")
			}

			// Create a new custom context
			uctx := &UserContext{
				Context: c,
				userID:  userID,
			}

			// Pass the new context forward
			return next(uctx)
		}
	}
}
```

## How to use this middleware and custom Context

You can easily incorporate this middleware into your routes for GET/POST requests.

```go
echo := echox.CreateEchoServer()
echo.GET("/:id", func(c echo.Context) error {
	return s.getRecipe(c)
}, AuthenticateUserMiddleware(s.authenticator))
```

To utilize the custom context, type casting is necessary along with validation.

```go
func (s recipe) getRecipe(c echo.Context) error {
	uc, isOk := c.(*echox.UserContext)
	if !isOk {
		return echo.NewHTTPError(http.StatusInternalServerError, "wrong context type")
	}

	userID := uc.GetUserID()
	...
}
```

## Summary

Middleware proves to be an invaluable tool, simplifying complex tasks such as authentication. I hope this article provides you with a deeper understanding of middleware functionality and how to implement it effectively.

## Further Reading

- [Echo Middleware Documentation](https://echo.labstack.com/middleware/)

### Bonus Trivia

Did you know there are more trees on Earth than stars in our galaxy? Check out these sources for more fascinating insights:

- [Plant-for-the-Planet Study](https://www.plant-for-the-planet.org/media/files/press/27fbf376-nydailynews-there-are-more-trees-on-earth-than-anyone-thought.pdf)
- [Snopes Fact Check](https://www.snopes.com/fact-check/trees-stars-milky-way/)
