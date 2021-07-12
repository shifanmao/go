# Golang Tour Solutions

Solutions to Go tour: https://tour.golang.org/list

##### Table of Contents  
1. [Basics](#basics)
2. [Methods and interfaces](#methods)
3. [Cocurrency](#cocurrency)

# Basics <a name="basics"></a>

## [Exercise: Loops and Functions](https://tour.golang.org/flowcontrol/8)
>As a way to play with functions and loops, let's implement a square root function: given a number x, we want to find the number z for which z² >is most nearly x.
>
>Computers typically compute the square root of x using a loop. Starting with some guess z, we can adjust z based on how close z² is to x, >producing a better guess:
>
>z -= (z*z - x) / (2*z)
>Repeating this adjustment makes the guess better and better until we reach an answer that is as close to the actual square root as can be.
>
>Implement this in the func Sqrt provided. A decent starting guess for z is 1, no matter what the input. To begin with, repeat the calculation >10 times and print each z along the way. See how close you get to the answer for various values of x (1, 2, 3, ...) and how quickly the guess >improves.
>
>Hint: To declare and initialize a floating point value, give it floating point syntax or use a conversion:
>
>z := 1.0
>z := float64(1)
>Next, change the loop condition to stop once the value has stopped changing (or only changes by a very small amount). See if that's more or >fewer than 10 iterations. Try other initial guesses for z, like x, or x/2. How close are your function's results to the math.Sqrt in the >standard library?
```golang
package main

import (
	"fmt"
)

func Sqrt(x float64) float64 {
	z := 1.0
	for i := 1; i <= 10; i++ {
		z -= (z*z - x) / (2*z)
		fmt.Printf("Iteraction %v, z=%g\n", i, z)
	}
	return z
}

func main() {
	fmt.Println(Sqrt(2))
	fmt.Println(Sqrt(3))
	fmt.Println(Sqrt(4))
}
```

## [Exercise: Slices](https://tour.golang.org/moretypes/18)
>Implement Pic. It should return a slice of length dy, each element of which is a slice of dx 8-bit unsigned integers. When you run the >program, it will display your picture, interpreting the integers as grayscale (well, bluescale) values.
>
>The choice of image is up to you. Interesting functions include (x+y)/2, x*y, and x^y.
>
>(You need to use a loop to allocate each []uint8 inside the [][]uint8.)
>
>(Use uint8(intValue) to convert between types.)
```golang
package main

import (
	"golang.org/x/tour/pic"
)

func Pic(dx, dy int) [][]uint8 {
	var res [][]uint8
	for y:=0; y<=dy-1; y++ {
		row := make([]uint8, dx)
		for x:=0; x<=dx-1; x++ {
			row[x] = uint8(x^y)
		}
		res = append(res, row)
	}
	return res
}

func main() {
	pic.Show(Pic)
}
```

## [Exercise: Maps](https://tour.golang.org/moretypes/23)
>Implement WordCount. It should return a map of the counts of each “word” in the string s. The wc.Test function runs a test suite against the >provided function and prints success or failure.
>
>You might find strings.Fields helpful.
```golang
package main

import (
	"golang.org/x/tour/wc"
	"strings"
)

func WordCount(s string) map[string]int {
	res := make(map[string]int)
	for _, word := range strings.Fields(s) {
		res[word] ++
	}
	return res
}

func main() {
	wc.Test(WordCount)
}
```

## [Exercise: Fibonacci closure](https://tour.golang.org/moretypes/26)
>Let's have some fun with functions.
>
>Implement a fibonacci function that returns a function (a closure) that returns successive fibonacci numbers (0, 1, 1, 2, 3, 5, ...).
```golang
package main

import "fmt"

// fibonacci is a function that returns
// a function that returns an int.
func fibonacci() func() int {
	pp, p := 0, 1
	return func() int {
		res := pp
		pp, p = p, pp+p
		return res
	}
}

func main() {
	f := fibonacci()
	for i := 0; i < 10; i++ {
		fmt.Println(f())
	}
}
```

# Methods and interfaces <a name="methods"></a>

## [Exercise: Stringers](https://tour.golang.org/methods/18)
>Make the IPAddr type implement fmt.Stringer to print the address as a dotted quad.
>
>For instance, IPAddr{1, 2, 3, 4} should print as "1.2.3.4".
```golang
package main

import (
	"fmt"
	"strings"
	"strconv"
)

type IPAddr [4]byte

func (ip IPAddr) String() string {
	var sb strings.Builder
	sb.WriteString(strconv.Itoa(int(ip[0])))
	for i := 1; i<len(ip); i++ {
		sb.WriteString(".")
		sb.WriteString(strconv.Itoa(int(ip[i])))
	}
	return sb.String()
}

func main() {
	hosts := map[string]IPAddr{
		"loopback":  {127, 0, 0, 1},
		"googleDNS": {8, 8, 8, 8},
	}
	for name, ip := range hosts {
		fmt.Printf("%v: %v\n", name, ip)
	}
}
```

## [Exercise: Errors](https://tour.golang.org/methods/20)
>Copy your Sqrt function from the earlier exercise and modify it to return an error value.
>
>Sqrt should return a non-nil error value when given a negative number, as it doesn't support complex numbers.
>
>Create a new type
>
>type ErrNegativeSqrt float64
>and make it an error by giving it a
>
>func (e ErrNegativeSqrt) Error() string
>method such that ErrNegativeSqrt(-2).Error() returns "cannot Sqrt negative number: -2".
>
>Note: A call to fmt.Sprint(e) inside the Error method will send the program into an infinite loop. You can avoid this by converting e first: >fmt.Sprint(float64(e)). Why?
>
>Change your Sqrt function to return an ErrNegativeSqrt value when given a negative number.
```golang
package main

import (
	"fmt"
)

type ErrNegativeSqrt float64

func (e ErrNegativeSqrt) Error() string {
	return fmt.Sprintf("cannot Sqrt negative number: %f", e)
}

func Sqrt(x float64) (float64, error) {
	if x < 0 {
		return 0, ErrNegativeSqrt(x)
	}
	z := 1.0
	for i := 1; i <= 10; i++ {
		z -= (z*z - x) / (2*z)
	}
	return z, nil
}

type myFloat float64

func main() {
	fmt.Println(Sqrt(2))
	fmt.Println(Sqrt(-2))
}
```


## [Exercise: Readers](https://tour.golang.org/methods/22)
> Implement a Reader type that emits an infinite stream of the ASCII character 'A'.
```golang
package main

import "golang.org/x/tour/reader"

type MyReader struct{}

// TODO: Add a Read([]byte) (int, error) method to MyReader.
func (mr MyReader) Read(b []byte) (n int, err error) {
	for i := range b {
		b[i] = byte('A')
	}
	return len(b), nil
}

func main() {
	reader.Validate(MyReader{})
}
```

## [Exercise: rot13Reader](https://tour.golang.org/methods/23)
>A common pattern is an io.Reader that wraps another io.Reader, modifying the stream in some way.
>
>For example, the gzip.NewReader function takes an io.Reader (a stream of compressed data) and returns a *gzip.Reader that also implements io.Reader (a stream >of the decompressed data).
>
>Implement a rot13Reader that implements io.Reader and reads from an io.Reader, modifying the stream by applying the rot13 substitution cipher to all >alphabetical characters.

>The rot13Reader type is provided for you. Make it an io.Reader by implementing its Read method.
```golang
package main

import (
	"io"
	"os"
	"strings"
	// "fmt"
	"unicode"
)

type rot13Reader struct {
	r io.Reader
}

func rotate13(ch byte) byte {
	if unicode.IsUpper(rune(ch)) {
		if ch <= byte('M') {
			return ch+13
		} else {
			return ch-13
		}
	} else {
		if ch <= byte('m') {
			return ch+13
		} else {
			return ch-13
		}
	}
}

func (rot *rot13Reader) Read(b []byte) (n int, err error) {
	a, err := rot.r.Read(b)
	for i, ch := range b {
		b[i] = rotate13(ch)
	}
	return a, err
}

func main() {
	s := strings.NewReader("Lbh penpxrq gur pbqr!")
	r := rot13Reader{s}
	io.Copy(os.Stdout, &r)
}
```


## [Exercise: Images](https://tour.golang.org/methods/25)
>Remember the picture generator you wrote earlier? Let's write another one, but this time it will return an implementation of image.Image instead of a slice of >data.
>
>Define your own Image type, implement the necessary methods, and call pic.ShowImage.
>
>Bounds should return a image.Rectangle, like image.Rect(0, 0, w, h).
>
>ColorModel should return color.RGBAModel.
>
>At should return a color; the value v in the last picture generator corresponds to color.RGBA{v, v, 255, 255} in this one.

```golang
package main

import (
	"golang.org/x/tour/pic"
	"image"
	"image/color"
)

type Image struct {
	w, h int
}

func (im Image) ColorModel() color.Model {
	return color.RGBAModel
}

func (im Image) Bounds() image.Rectangle {
	return image.Rect(0, 0, im.w, im.h)
}

func (im Image) At(x, y int) color.Color {
	v := uint8(x^y)
	return color.RGBA{v, v, 255, 255}
}

func main() {
	m := Image{100, 100}
	pic.ShowImage(m)
}
```

# Cocurrency <a name="cocurrency"></a>

## [Exercise: Equivalent Binary Trees](https://tour.golang.org/concurrency/7)

```golang
package main

import (
	"golang.org/x/tour/tree"
	"fmt"
)

// Walk walks the tree t sending all values
// from the tree to the channel ch.
func Walk(t *tree.Tree, c chan int) {
	defer close(c)
	var walk func(t *tree.Tree)
	walk = func(t *tree.Tree) {
		if t == nil {
			return
		}
		walk(t.Left)
		c <- t.Value
		walk(t.Right)
	}
	walk(t)
}

// Same determines whether the trees
// t1 and t2 contain the same values.
func Same(t1, t2 *tree.Tree) bool {
	c1 := make(chan int)
	c2 := make(chan int)
	go Walk(t1, c1)
	go Walk(t2, c2)
	for {
		v1, ok1 := <-c1
		v2, ok2 := <-c2
		if ok1 != ok2 || v1 != v2 {
			return false
		}
		if !ok1 {
			return true
		}
	}
	return true
}

func main() {
	c := make(chan int)
	go Walk(tree.New(1), c)
	for i := range c {
		fmt.Println(i)
	}
	
	if Same(tree.New(1), tree.New(1)) != true {
		panic("Wrong Same output for tree.New(1) vs. tree.New(1)")
	}
	if Same(tree.New(1), tree.New(2)) != false {
		panic("Wrong Same output for tree.New(1) vs. tree.New(2)")
	}
}
```

## [Exercise: Web Crawler](https://tour.golang.org/concurrency/10)

> In this exercise you'll use Go's concurrency features to parallelize a web crawler.
> 
> Modify the Crawl function to fetch URLs in parallel without fetching the same URL twice.
> 
> Hint: you can keep a cache of the URLs that have been fetched on a map, but maps alone are not safe for concurrent use!


```golang
package main

import (
	"fmt"
	"sync"
)

type Fetcher interface {
	// Fetch returns the body of URL and
	// a slice of URLs found on that page.
	Fetch(url string) (body string, urls []string, err error)
}

type SafeSet struct {
	mu sync.Mutex
	v map[string]bool
}

func (c *SafeSet) Add(key string) {
	c.mu.Lock()
	c.v[key] = true
	c.mu.Unlock()
}

func (c *SafeSet) Has(key string) bool {
	c.mu.Lock()
	defer c.mu.Unlock()
	_, ok := c.v[key]
	return ok
}

var wg sync.WaitGroup

// Crawl uses fetcher to recursively crawl
// pages starting with url, to a maximum of depth.
func Crawl(url string, depth int, fetcher Fetcher, s SafeSet) {
	defer wg.Done()

	s.Add(url) // This has to be here to prevent duplicate crawling
	
	if depth <= 0 {
		return
	}
	body, urls, err := fetcher.Fetch(url)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("found: %s %q\n", url, body)
	
	for _, u := range urls {
		if !s.Has(u) {
			wg.Add(1)
			go Crawl(u, depth-1, fetcher, s)
		}
	}
	return
}

func main() {
	s := SafeSet{v: make(map[string]bool)}
	wg.Add(1)
	Crawl("https://golang.org/", 4, fetcher, s)
	wg.Wait()
}

// fakeFetcher is Fetcher that returns canned results.
type fakeFetcher map[string]*fakeResult

type fakeResult struct {
	body string
	urls []string
}

func (f fakeFetcher) Fetch(url string) (string, []string, error) {
	if res, ok := f[url]; ok {
		return res.body, res.urls, nil
	}
	return "", nil, fmt.Errorf("not found: %s", url)
}

// fetcher is a populated fakeFetcher.
var fetcher = fakeFetcher{
	"https://golang.org/": &fakeResult{
		"The Go Programming Language",
		[]string{
			"https://golang.org/pkg/",
			"https://golang.org/cmd/",
		},
	},
	"https://golang.org/pkg/": &fakeResult{
		"Packages",
		[]string{
			"https://golang.org/",
			"https://golang.org/cmd/",
			"https://golang.org/pkg/fmt/",
			"https://golang.org/pkg/os/",
		},
	},
	"https://golang.org/pkg/fmt/": &fakeResult{
		"Package fmt",
		[]string{
			"https://golang.org/",
			"https://golang.org/pkg/",
		},
	},
	"https://golang.org/pkg/os/": &fakeResult{
		"Package os",
		[]string{
			"https://golang.org/",
			"https://golang.org/pkg/",
		},
	},
}

```
