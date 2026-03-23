---
title: "Golden File Testing in Go"
date: 2020-12-17
categories: ["Go"]
---

In traditional unit testing, specifying expected outputs directly within the test function is common. However, when dealing with complex data structures, such as a large tree, this approach can become unwieldy. Enter golden file testing.

Golden file testing alleviates the burden of manually specifying expected outputs by storing them in separate files. Each test case has its own file containing the expected output, typically stored in a designated directory (commonly named "testdata" and placed adjacent to the test file).

<!--more-->

**Example of Traditional Unit Test:**

```golang
func TestCreateTree(t *testing.T) {
	tests := []struct {
		name     string
		size     int
		expected Node
	}{
		{"Simple example of JSON", 100,
			Node{
				Value: 0,
				Children: Node{
					{
						"Value": 1,
						"Children": [...],
					},
					...
				},
			}.
		}
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			got := CreateTree(tt.size)
			assert.Equal(t, tt.expected, got)
		})
	}
}
```

**Improved Approach using Golden Files:**

```golang
func TestCreateTree(t *testing.T) {
	tests := []struct {
		name     string
		size     int
		filename string
	}{
		{"Simple example of JSON", 100, "testdata/node100"},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			got := CreateTree(tt.size)

			goldtest.AssertJSON(t, got, tt.filename)
		})
	}
}
```

This approach offers cleaner and more maintainable testing code.

## Introducing the goldtest package

The  [goldtest](https://github.com/piotrowski/goldtest) package simplifies golden file testing in Go. It provides easy-to-use functions like Assert and AssertJSON for comparing actual outputs with expected results stored in golden files.

I have created package [goldtest](https://github.com/apiotrowski312/goldtest), that will help you with this type of testing.

### Usage:

```golang
goldtest.AssertJSON(t, got, tt.filename)
```

### Assert

The Assert function converts any interface into a string and then into bytes, which are then saved into a file. While effective, this approach may result in lower readability on the golden file, especially with larger structs.

### AssertJSON

The AssertJSON function, on the other hand, leverages Go's Marshal function to generate well-structured and readable JSON output, making it easier to understand and maintain.

There are two possible ways to test all data you want. You can export every field that you want to test with goldenfile. Other way is to create your own Marshal function (Example [HERE](http://choly.ca/post/go-json-marshalling/) and [HERE](https://medium.com/@dynastymasra/override-json-marshalling-in-go-cb418102c60f)).

## Further reading

- [https://ieftimov.com/post/testing-in-go-golden-files/](https://ieftimov.com/post/testing-in-go-golden-files/)
- [https://medium.com/@jarifibrahim/golden-files-why-you-should-use-them-47087ec994bf](https://medium.com/@jarifibrahim/golden-files-why-you-should-use-them-47087ec994bf)
- [https://github.com/apiotrowski312/goldtest](https://github.com/apiotrowski312/goldtest)
