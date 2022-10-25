# gomock example
[gomock](https://github.com/golang/mock) is a mocking framework that allows you to replace network calls with mocked network calls in your tests.

## File Structure
- gomock_example: The core functionality of this package.
- client.go: Contains the example API client that will be mocked. 
- gomock_example_test: The test code implementing `gomock`.
- mock.go: Code generated by `mockgen`.


## Running this Example
```
$ go get
$ go test .
```

## About this Example
Gomock relies on files generated by `mock.go`. These files use interfaces that you define.

The key to setting up mocking is to write interfaces for your objects that are making network calls.

In this example, I created a `client.go` file with the following content
```go
type ApiClient interface {
	// When called, the API is expected to double the provided value
	ApiDouble(valueToDouble int) (int, error)
}
```

This is a simple example of a client that makes a network call during its execution. For simplicity, there is no network call here, but the idea is the same. We can imagine that `ApiDouble` hits some external service to perform its operation. We don't want to include this call in our unit tests to avoid flakiness/eliminate extra dependencies.

In order to generate our mocked code, we first install `mockgen` with:
```
# Assumes Go 1.16+
go install github.com/golang/mock/mockgen@v1.6.0
```
Then, from the root of our project run
```
mockgen -source client.go mock.go > mock.go
```
Now we have mock code that we can use. The mock client can be created with `NewMockApiClient` which returns a `MockApiClient` which implements our provided interface. This is passed into the object we are testing. 

```go
func getMockServer(t *testing.T) (*Server, *MockApiClient, *gomock.Controller) {
	ctrl := gomock.NewController(t)
	cli := NewMockApiClient(ctrl)
	server := NewServer(cli)
	return server, cli, ctrl
}
```

Anywhere that this server uses its `Client`, it will use the mock client instead. 

### More Information
- [Mocking Wikipedia Page](https://en.wikipedia.org/wiki/Mock_object)
- [gomock repo](https://github.com/golang/mock)