# Go Benchmarks

[![test](https://github.com/SimonWaldherr/golang-benchmarks/actions/workflows/audit.yml/badge.svg?branch=master&event=push)](https://github.com/SimonWaldherr/golang-benchmarks/actions/workflows/audit.yml) 
[![DOI](https://zenodo.org/badge/154216722.svg)](https://zenodo.org/badge/latestdoi/154216722) 
[![Go Report Card](https://goreportcard.com/badge/github.com/SimonWaldherr/golang-benchmarks)](https://goreportcard.com/report/github.com/SimonWaldherr/golang-benchmarks) 
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)  

In programming in general, and in Golang in particular, many roads lead to Rome.
From time to time I ask myself which of these ways is the fastest. 
In Golang there is a wonderful solution, with `go test -bench` you can measure the speed very easily and quickly.
In order for you to benefit from it too, I will publish such benchmarks in this repository in the future.

## ToC

* [base64](https://github.com/SimonWaldherr/golang-benchmarks#base64)
* [between](https://github.com/SimonWaldherr/golang-benchmarks#between)
* [caseinsensitivecompare](https://github.com/SimonWaldherr/golang-benchmarks#caseinsensitivecompare)
* [concat](https://github.com/SimonWaldherr/golang-benchmarks#concat)
* [contains](https://github.com/SimonWaldherr/golang-benchmarks#contains)
* [concurrency_counter](https://github.com/SimonWaldherr/golang-benchmarks#concurrency_counter)
* [embed](https://github.com/SimonWaldherr/golang-benchmarks#embed)
* [floodfill](https://github.com/SimonWaldherr/golang-benchmarks#floodfill)
* [foreach](https://github.com/SimonWaldherr/golang-benchmarks#foreach)
* [hash](https://github.com/SimonWaldherr/golang-benchmarks#hash)
* [index](https://github.com/SimonWaldherr/golang-benchmarks#index)
* [json](https://github.com/SimonWaldherr/golang-benchmarks#json)
* [math](https://github.com/SimonWaldherr/golang-benchmarks#math)
* [parse](https://github.com/SimonWaldherr/golang-benchmarks#parse)
* [random](https://github.com/SimonWaldherr/golang-benchmarks#random)
* [regexp](https://github.com/SimonWaldherr/golang-benchmarks#regexp)
* [sql](https://github.com/SimonWaldherr/golang-benchmarks#sql)
* [template](https://github.com/SimonWaldherr/golang-benchmarks#template)
* [trim](https://github.com/SimonWaldherr/golang-benchmarks#trim)

## Golang?

I published another repository where I show some Golang examples.
If you\'re interested in new programming languages, you should definitely take a look at Golang:

* [Golang examples](https://github.com/SimonWaldherr/golang-examples)
* [tour.golang.org](https://tour.golang.org/)
* [Go by example](https://gobyexample.com/)
* [Golang Book](http://www.golang-book.com/)
* [Go-Learn](https://github.com/skippednote/Go-Learn)

## Is it any good?

[Yes](https://news.ycombinator.com/item?id=3067434)

## Benchmark Results

Golang Version: [go version go1.27.0 darwin/arm64](https://tip.golang.org/doc/go1.27)  
Hardware Spec: [Apple MacBook Pro 16-Inch M2 Max 2023](https://support.apple.com/kb/SP890) [(?)](https://everymac.com/systems/apple/macbook_pro/specs/macbook-pro-m2-max-12-core-cpu-30-core-gpu-16-2023-specs.html) [(buy)](https://amzn.to/3K80lP4)  

### base64

```go
// Package base64 benchmarks some base64 functions.
// On all tested systems it's faster to decode a
// base64 encoded string instead of a check via regex.
package base64

import (
	"encoding/base64"
	"regexp"
	"testing"
)

func base64decode(s string) bool {
	_, err := base64.StdEncoding.DecodeString(s)
	return err == nil
}

func base64regex(s string) bool {
	matched, _ := regexp.MatchString(`^([A-Za-z0-9+/]{4})*([A-Za-z0-9+/]{4}|[A-Za-z0-9+/]{3}=|[A-Za-z0-9+/]{2}==)$`, s)
	return matched
}

func BenchmarkBase64decode(b *testing.B) {
	isNotBase64 := `Invalid string`
	isBase64 := `VmFsaWQgc3RyaW5nCg==`

	for n := 0; n < b.N; n++ {
		base64decode(isNotBase64)
		base64decode(isBase64)
	}
}

func BenchmarkBase64regex(b *testing.B) {
	isNotBase64 := `Invalid string`
	isBase64 := `VmFsaWQgc3RyaW5nCg==`

	for n := 0; n < b.N; n++ {
		base64regex(isNotBase64)
		base64regex(isBase64)
	}
}
```

```
$ go test -bench . -benchmem
goos: darwin
goarch: arm64
pkg: github.com/SimonWaldherr/golang-benchmarks/base64
cpu: Apple M2 Max
BenchmarkBase64decode-12    	23344996	        50.48 ns/op	      32 B/op	       2 allocs/op
BenchmarkBase64regex-12     	  138854	      8582 ns/op	   21813 B/op	     196 allocs/op
PASS
ok  	github.com/SimonWaldherr/golang-benchmarks/base64	2.882s
```

### between

```go
// Package between compares the performance of checking
// if a number is between two other numbers via regex
// and by parsing the number as integers.
package between

import (
	"regexp"
	"simonwaldherr.de/go/golibs/as"
	"simonwaldherr.de/go/ranger"
	"testing"
)

func BenchmarkNumberRegEx(b *testing.B) {
	re := ranger.Compile(89, 1001)
	re = "^(" + re + ")$"
	b.ResetTimer()

	for n := 0; n < b.N; n++ {
		matched, err := regexp.MatchString(re, "404")
		if !matched || err != nil {
			b.Log("Error in Benchmark")
		}

		matched, err = regexp.MatchString(re, "2000")
		if matched || err != nil {
			b.Log("Error in Benchmark")
		}
	}
}

func BenchmarkFulltextRegEx(b *testing.B) {
	re := ranger.Compile(89, 1001)
	re = " (" + re + ") "
	b.ResetTimer()

	for n := 0; n < b.N; n++ {
		matched, err := regexp.MatchString(re, "lorem ipsum 404 dolor sit")
		if !matched || err != nil {
			b.Log("Error in Benchmark")
		}

		matched, err = regexp.MatchString(re, "lorem ipsum 2000 dolor sit")
		if matched || err != nil {
			b.Log("Error in Benchmark")
		}
	}
}

func BenchmarkNumberParse(b *testing.B) {
	for n := 0; n < b.N; n++ {
		i1 := as.Int("404")
		i2 := as.Int("2000")

		if i1 < 89 || i1 > 1001 {
			b.Log("Error in Benchmark")
		}

		if !(i2 < 89 || i2 > 1001) {
			b.Log("Error in Benchmark")
		}
	}
}

func BenchmarkFulltextParse(b *testing.B) {
	re := regexp.MustCompile("[0-9]+")
	b.ResetTimer()

	for n := 0; n < b.N; n++ {
		i1 := as.Int(re.FindString("lorem ipsum 404 dolor sit"))
		i2 := as.Int(re.FindString("lorem ipsum 2000 dolor sit"))

		if i1 < 89 || i1 > 1001 {
			b.Log("Error in Benchmark")
		}

		if !(i2 < 89 || i2 > 1001) {
			b.Log("Error in Benchmark")
		}
	}
}
```

```
$ go test -bench . -benchmem
goos: darwin
goarch: arm64
pkg: github.com/SimonWaldherr/golang-benchmarks/between
cpu: Apple M2 Max
BenchmarkNumberRegEx-12      	  198457	      5840 ns/op	   16796 B/op	     140 allocs/op
BenchmarkFulltextRegEx-12    	  251331	      4881 ns/op	   12067 B/op	     104 allocs/op
BenchmarkNumberParse-12      	30390710	        38.99 ns/op	       0 B/op	       0 allocs/op
BenchmarkFulltextParse-12    	 2534812	       474.4 ns/op	      32 B/op	       2 allocs/op
PASS
ok  	github.com/SimonWaldherr/golang-benchmarks/between	5.753s
```

### caseinsensitivecompare

```go
package caseinsensitivecompare

import (
	"strings"
	"testing"
)

var caseInsensitiveCompareResult bool

func BenchmarkEqualFold(b *testing.B) {
	b.ReportAllocs()
	for n := 0; n < b.N; n++ {
		caseInsensitiveCompareResult = strings.EqualFold("abc", "ABC")
		caseInsensitiveCompareResult = strings.EqualFold("ABC", "ABC")
		caseInsensitiveCompareResult = strings.EqualFold("1aBcD", "1AbCd")
	}
}

func BenchmarkToUpper(b *testing.B) {
	b.ReportAllocs()
	for n := 0; n < b.N; n++ {
		caseInsensitiveCompareResult = strings.ToUpper("abc") == strings.ToUpper("ABC")
		caseInsensitiveCompareResult = strings.ToUpper("ABC") == strings.ToUpper("ABC")
		caseInsensitiveCompareResult = strings.ToUpper("1aBcD") == strings.ToUpper("1AbCd")
	}
}

func BenchmarkToLower(b *testing.B) {
	b.ReportAllocs()
	for n := 0; n < b.N; n++ {
		caseInsensitiveCompareResult = strings.ToLower("abc") == strings.ToLower("ABC")
		caseInsensitiveCompareResult = strings.ToLower("ABC") == strings.ToLower("ABC")
		caseInsensitiveCompareResult = strings.ToLower("1aBcD") == strings.ToLower("1AbCd")
	}
}
```

```
$ go test -bench . -benchmem
goos: darwin
goarch: arm64
pkg: github.com/SimonWaldherr/golang-benchmarks/caseinsensitivecompare
cpu: Apple M2 Max
BenchmarkEqualFold-12    	82123342	        15.16 ns/op	       0 B/op	       0 allocs/op
BenchmarkToUpper-12      	10408836	       114.1 ns/op	      24 B/op	       3 allocs/op
BenchmarkToLower-12      	 8714013	       139.2 ns/op	      40 B/op	       5 allocs/op
PASS
ok  	github.com/SimonWaldherr/golang-benchmarks/caseinsensitivecompare	4.229s
```

### concat

```go
// Package concat benchmarks the performance of
// various string concatenation methods.
// Instead of just concatenating a string to another string
// it is also possible (and much faster) to use
// a bytes buffer.
package concat

import (
	"bytes"
	"strings"
	"testing"
)

const concatParts = 64

var concatResult string

func BenchmarkConcatString(b *testing.B) {
	b.ReportAllocs()

	for n := 0; n < b.N; n++ {
		var str string
		for i := 0; i < concatParts; i++ {
			str += "x"
		}
		concatResult = str
	}
}

func BenchmarkConcatBuffer(b *testing.B) {
	b.ReportAllocs()

	for n := 0; n < b.N; n++ {
		var buffer bytes.Buffer
		for i := 0; i < concatParts; i++ {
			buffer.WriteString("x")
		}
		concatResult = buffer.String()
	}
}

func BenchmarkConcatBuilder(b *testing.B) {
	b.ReportAllocs()

	for n := 0; n < b.N; n++ {
		var builder strings.Builder
		for i := 0; i < concatParts; i++ {
			builder.WriteString("x")
		}
		concatResult = builder.String()
	}
}

func BenchmarkConcatBuilderGrow(b *testing.B) {
	b.ReportAllocs()

	for n := 0; n < b.N; n++ {
		var builder strings.Builder
		builder.Grow(concatParts)
		for i := 0; i < concatParts; i++ {
			builder.WriteString("x")
		}
		concatResult = builder.String()
	}
}

func BenchmarkConcat(b *testing.B) {
	b.Run("String", func(b *testing.B) {
		b.ReportAllocs()
		for n := 0; n < b.N; n++ {
			var str string
			for i := 0; i < concatParts; i++ {
				str += "x"
			}
			concatResult = str
		}
	})

	b.Run("Buffer", func(b *testing.B) {
		b.ReportAllocs()
		for n := 0; n < b.N; n++ {
			var buffer bytes.Buffer
			for i := 0; i < concatParts; i++ {
				buffer.WriteString("x")
			}
			concatResult = buffer.String()
		}
	})

	b.Run("Builder", func(b *testing.B) {
		b.ReportAllocs()
		for n := 0; n < b.N; n++ {
			var builder strings.Builder
			for i := 0; i < concatParts; i++ {
				builder.WriteString("x")
			}
			concatResult = builder.String()
		}
	})

	b.Run("BuilderGrow", func(b *testing.B) {
		b.ReportAllocs()
		for n := 0; n < b.N; n++ {
			var builder strings.Builder
			builder.Grow(concatParts)
			for i := 0; i < concatParts; i++ {
				builder.WriteString("x")
			}
			concatResult = builder.String()
		}
	})
}
```

```
$ go test -bench . -benchmem
goos: darwin
goarch: arm64
pkg: github.com/SimonWaldherr/golang-benchmarks/concat
cpu: Apple M2 Max
BenchmarkConcatString-12         	  897886	      1306 ns/op	    2400 B/op	      63 allocs/op
BenchmarkConcatBuffer-12         	 4717789	       255.7 ns/op	     128 B/op	       2 allocs/op
BenchmarkConcatBuilder-12        	 7698123	       159.0 ns/op	     120 B/op	       4 allocs/op
BenchmarkConcatBuilderGrow-12    	 8839840	       136.2 ns/op	      64 B/op	       1 allocs/op
BenchmarkConcat/String-12        	  899257	      1323 ns/op	    2400 B/op	      63 allocs/op
BenchmarkConcat/Buffer-12        	 4731350	       256.2 ns/op	     128 B/op	       2 allocs/op
BenchmarkConcat/Builder-12       	 5874370	       205.0 ns/op	     120 B/op	       4 allocs/op
BenchmarkConcat/BuilderGrow-12   	 7911778	       176.9 ns/op	      64 B/op	       1 allocs/op
PASS
ok  	github.com/SimonWaldherr/golang-benchmarks/concat	11.396s
```

### contains

```go
// Package contains tests various ways of checking
// if a string is contained in another string.
package contains

import (
	"bytes"
	"regexp"
	"strings"
	"testing"
)

// strings.Contains
func contains() bool {
	return strings.Contains("Lorem Ipsum", "em Ip")
}

func containsNot() bool {
	return strings.Contains("Lorem Ipsum", "Dolor")
}

func TestContains(t *testing.T) {
	if contains() == false {
		t.Error("ERROR: contains")
	}
	if containsNot() == true {
		t.Error("ERROR: contains not")
	}
}

func BenchmarkContains(b *testing.B) {
	for n := 0; n < b.N; n++ {
		contains()
	}
}

func BenchmarkContainsNot(b *testing.B) {
	for n := 0; n < b.N; n++ {
		containsNot()
	}
}

// bytes.Contains
func containsBytes() bool {
	return bytes.Contains([]byte("Lorem Ipsum"), []byte("em Ip"))
}

func containsBytesNot() bool {
	return bytes.Contains([]byte("Lorem Ipsum"), []byte("Dolor"))
}

func TestContainsBytes(t *testing.T) {
	if containsBytes() == false {
		t.Error("ERROR: bytes contains")
	}
	if containsBytesNot() == true {
		t.Error("ERROR: bytes contains not")
	}
}

func BenchmarkContainsBytes(b *testing.B) {
	for n := 0; n < b.N; n++ {
		containsBytes()
	}
}

func BenchmarkContainsBytesNot(b *testing.B) {
	for n := 0; n < b.N; n++ {
		containsBytesNot()
	}
}

// regexp.MustCompile + regexp.MatchString
func compileMatch(re *regexp.Regexp) bool {
	matched := re.MatchString("Lorem Ipsum")
	return matched
}

func compileMatchNot(re *regexp.Regexp) bool {
	matched := re.MatchString("Lorem Ipsum")
	return matched
}

func TestCompileMatch(t *testing.T) {
	re1 := regexp.MustCompile("em Ip")
	re2 := regexp.MustCompile("Dolor")
	if compileMatch(re1) == false {
		t.Error("ERROR: compile match")
	}
	if compileMatchNot(re2) == true {
		t.Error("ERROR: compile match not")
	}
}

func BenchmarkCompileMatch(b *testing.B) {
	re := regexp.MustCompile("em Ip")
	for n := 0; n < b.N; n++ {
		compileMatch(re)
	}
}

func BenchmarkCompileMatchNot(b *testing.B) {
	re := regexp.MustCompile("Dolor")
	for n := 0; n < b.N; n++ {
		compileMatchNot(re)
	}
}

// regexp.MatchString
func match() bool {
	matched, _ := regexp.MatchString("em Ip", "Lorem Ipsum")
	return matched
}

func matchNot() bool {
	matched, _ := regexp.MatchString("Dolor", "Lorem Ipsum")
	return matched
}

func TestMatch(t *testing.T) {
	if match() == false {
		t.Error("ERROR: match")
	}
	if matchNot() == true {
		t.Error("ERROR: match not")
	}
}

func BenchmarkMatch(b *testing.B) {
	for n := 0; n < b.N; n++ {
		match()
	}
}

func BenchmarkMatchNot(b *testing.B) {
	for n := 0; n < b.N; n++ {
		matchNot()
	}
}

// BenchmarkContainsMethods benchmarks different methods to check substring presence.
func BenchmarkContainsMethods(b *testing.B) {
	b.Run("Strings.Contains", func(b *testing.B) {
		str := "Lorem Ipsum"
		substr := "em Ip"
		for n := 0; n < b.N; n++ {
			_ = strings.Contains(str, substr)
			_ = strings.Contains(str, "Dolor")
		}
	})

	b.Run("Bytes.Contains", func(b *testing.B) {
		str := []byte("Lorem Ipsum")
		substr := []byte("em Ip")
		for n := 0; n < b.N; n++ {
			_ = bytes.Contains(str, substr)
			_ = bytes.Contains(str, []byte("Dolor"))
		}
	})

	b.Run("RegexMatchString", func(b *testing.B) {
		re := regexp.MustCompile(`em Ip`)
		for n := 0; n < b.N; n++ {
			_ = re.MatchString("Lorem Ipsum")
			_ = re.MatchString("Dolor")
		}
	})

	b.Run("RegexMatch", func(b *testing.B) {
		for n := 0; n < b.N; n++ {
			_, _ = regexp.MatchString(`em Ip`, "Lorem Ipsum")
			_, _ = regexp.MatchString(`em Ip`, "Dolor")
		}
	})
}
```

```
$ go test -bench . -benchmem
goos: darwin
goarch: arm64
pkg: github.com/SimonWaldherr/golang-benchmarks/contains
cpu: Apple M2 Max
BenchmarkContains-12            	224095947	         5.426 ns/op	       0 B/op	       0 allocs/op
BenchmarkContainsNot-12         	150259542	         8.473 ns/op	       0 B/op	       0 allocs/op
BenchmarkContainsBytes-12       	100000000	        11.52 ns/op	       0 B/op	       0 allocs/op
BenchmarkContainsBytesNot-12    	100000000	        10.63 ns/op	       0 B/op	       0 allocs/op
BenchmarkCompileMatch-12        	17734885	        62.47 ns/op	       0 B/op	       0 allocs/op
BenchmarkCompileMatchNot-12     	37624876	        37.78 ns/op	       0 B/op	       0 allocs/op
BenchmarkMatch-12               	 1069442	      1101 ns/op	    1399 B/op	      17 allocs/op
BenchmarkMatchNot-12            	 1000000	      1244 ns/op	    1398 B/op	      17 allocs/op
BenchmarkContainsMethods/Strings.Contains-12         	87740680	        13.18 ns/op	       0 B/op	       0 allocs/op
BenchmarkContainsMethods/Bytes.Contains-12           	73549027	        18.07 ns/op	       0 B/op	       0 allocs/op
BenchmarkContainsMethods/RegexMatchString-12         	11953504	       108.6 ns/op	       0 B/op	       0 allocs/op
BenchmarkContainsMethods/RegexMatch-12               	  555619	      2380 ns/op	    2800 B/op	      34 allocs/op
PASS
ok  	github.com/SimonWaldherr/golang-benchmarks/contains	19.433s
```

### concurrency_counter

```go
package concurrency_counter

import (
	"fmt"
	"runtime"
	"sync"
	"sync/atomic"
	"testing"
)

// Benchmarks comparing common concurrency counter patterns.
// Variants:
// - sync.Mutex (write)
// - sync.RWMutex (write + read-heavy)
// - atomic.AddInt64
// - channels (various buffer sizes)
// - worker-pool (fixed workers consuming jobs)

func BenchmarkMutexParallel(b *testing.B) {
	var mu sync.Mutex
	var counter int64

	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			mu.Lock()
			counter++
			mu.Unlock()
		}
	})
}

func BenchmarkRWMutexWriteParallel(b *testing.B) {
	var mu sync.RWMutex
	var counter int64

	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			mu.Lock()
			counter++
			mu.Unlock()
		}
	})
}

func BenchmarkRWMutexReadParallel(b *testing.B) {
	var mu sync.RWMutex
	var counter int64

	// populate counter
	counter = 42

	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			mu.RLock()
			_ = counter
			mu.RUnlock()
		}
	})
}

func BenchmarkAtomicParallel(b *testing.B) {
	var counter int64

	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			atomic.AddInt64(&counter, 1)
		}
	})
}

func BenchmarkChannelBufferedSizes(b *testing.B) {
	sizes := []int{0, 1, 16, 1024}

	for _, sz := range sizes {
		b.Run(fmt.Sprintf("buf=%d", sz), func(b *testing.B) {
			ch := make(chan struct{}, sz)
			var wg sync.WaitGroup
			var counter int64

			wg.Add(1)
			go func() {
				for range ch {
					atomic.AddInt64(&counter, 1)
				}
				wg.Done()
			}()

			b.RunParallel(func(pb *testing.PB) {
				for pb.Next() {
					ch <- struct{}{}
				}
			})

			close(ch)
			wg.Wait()
		})
	}
}

func BenchmarkWorkerPool(b *testing.B) {
	workers := runtime.GOMAXPROCS(0)

	b.Run(fmt.Sprintf("workers=%d", workers), func(b *testing.B) {
		jobs := make(chan struct{}, 1024)
		var wg sync.WaitGroup
		var counter int64

		wg.Add(workers)
		for i := 0; i < workers; i++ {
			go func() {
				for range jobs {
					atomic.AddInt64(&counter, 1)
				}
				wg.Done()
			}()
		}

		b.RunParallel(func(pb *testing.PB) {
			for pb.Next() {
				jobs <- struct{}{}
			}
		})

		close(jobs)
		wg.Wait()
	})
}
```

```
$ go test -bench . -benchmem
goos: darwin
goarch: arm64
pkg: github.com/SimonWaldherr/golang-benchmarks/concurrency_counter
cpu: Apple M2 Max
BenchmarkMutexParallel-12           	12906538	        91.38 ns/op	       0 B/op	       0 allocs/op
BenchmarkRWMutexWriteParallel-12    	14970494	        90.46 ns/op	       0 B/op	       0 allocs/op
BenchmarkRWMutexReadParallel-12     	25601413	        52.91 ns/op	       0 B/op	       0 allocs/op
BenchmarkAtomicParallel-12          	42092708	        24.92 ns/op	       0 B/op	       0 allocs/op
BenchmarkChannelBufferedSizes/buf=0-12         	 3521269	       347.7 ns/op	       0 B/op	       0 allocs/op
BenchmarkChannelBufferedSizes/buf=1-12         	 5069686	       250.5 ns/op	       0 B/op	       0 allocs/op
BenchmarkChannelBufferedSizes/buf=16-12        	 9939571	       132.7 ns/op	       0 B/op	       0 allocs/op
BenchmarkChannelBufferedSizes/buf=1024-12      	11569461	       110.6 ns/op	       0 B/op	       0 allocs/op
BenchmarkWorkerPool/workers=12-12              	15736689	        65.98 ns/op	       0 B/op	       0 allocs/op
PASS
ok  	github.com/SimonWaldherr/golang-benchmarks/concurrency_counter	13.315s
```

### embed

```go
package embed

import (
	_ "embed"
	"io/ioutil"
	"os"
	"testing"
)

//go:embed example.txt
va