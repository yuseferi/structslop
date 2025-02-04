# structslop

![Build status](https://github.com/orijtech/structslop/workflows/Go/badge.svg?branch=master)

Package structslop defines an [Analyzer](analyzer_link) that checks if struct fields can be re-arranged to optimize size.

## Installation

With Go modules:

```sh
go install github.com/orijtech/structslop/cmd/structslop@latest
```

Without Go modules:

```sh
$ cd $GOPATH/src/github.com/orijtech/structslop
$ git checkout v0.0.6
$ go get
$ install ./cmd/structslop
```

## Usage

You can run `structslop` either on a Go package or Go files, the same way as
other Go tools work.

Example:

```sh
$ structslop github.com/orijtech/structslop/testdata/src/struct
```

or:

```sh
$ structslop ./testdata/src/struct/p.go
```

Sample output:

```text
/go/src/github.com/orijtech/structslop/testdata/struct/p.go:30:9: struct has size 24 (size class 32), could be 16 (size class 16), you'll save 50.00% if you rearrange it to:
struct {
	y uint64
	x uint32
	z uint32
}

/go/src/github.com/orijtech/structslop/testdata/struct/p.go:36:9: struct has size 40 (size class 48), could be 24 (size class 32), you'll save 33.33% if you rearrange it to:
struct {
	_  [0]func()
	i1 int
	i2 int
	a3 [3]bool
	b  bool
}

/go/src/github.com/orijtech/structslop/testdata/struct/p.go:59:9: struct has size 40 (size class 48), could be 32 (size class 32), you'll save 33.33% if you rearrange it to:
struct {
	y uint64
	t *httptest.Server
	w uint64
	x uint32
	z uint32
}

/go/src/github.com/orijtech/structslop/testdata/struct/p.go:67:9: struct has size 40 (size class 48), could be 32 (size class 32), you'll save 33.33% if you rearrange it to:
struct {
	y uint64
	t *s
	w uint64
	x uint32
	z uint32
}

```

Example, for the first report above, the output meaning:

 - The current struct size is `24`, the size that the Go runtime will allocate for that struct is `32`.
 - The optimal struct size is `16`, the size that the Go runtime will allocate for that struct is `16`.
 - The layout of optimal struct.
 - The percentage savings with new struct layout.
 
That said, some structs may have a smaller size, but for efficiency, the Go runtime will allocate them in the same size class,
then those structs are not considered sloppy:

```go
type s1 struct {
	x uint32
	y uint64
	z *s
	t uint32
}
```

and:

```go
type s2 struct {
	y uint64
	z *s
	x uint32
	t uint32
}
```

have the same size class `32`, though `s2` layout is only `24` byte in size.

However, you can still get this information when you want, using `-verbose` flag:

```sh
$ structslop -verbose ./testdata/src/verbose/p.go
/go/src/github.com/orijtech/structslop/testdata/src/verbose/p.go:17:8: struct has size 0 (size class 0)
/go/src/github.com/orijtech/structslop/testdata/src/verbose/p.go:19:9: struct has size 1 (size class 8)
/go/src/github.com/orijtech/structslop/testdata/src/verbose/p.go:23:9: struct has size 32 (size class 32), could be 24 (size class 32), optimal fields order:
struct {
	y uint64
	z *s
	x uint32
	t uint32
}
```
 
**Note**

For applying suggested fix, use `-apply` flag, instead of `-fix`.

## Development

Go 1.20+

### Running test

Add test case to `testdata/src/struct` directory, then run:

```shell script
go test
```

## Contributing

TODO

[analyzer_link]: https://pkg.go.dev/golang.org/x/tools/go/analysis#Analyzer
