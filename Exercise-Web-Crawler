// https://tour.golang.org/concurrency/10
// This code shows a good example on how to wait 
// until all chidren routines done

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

type URLs struct {
	List map[string]bool
	mux  sync.Mutex
}

func (urls URLs) Exists(url string) bool {
	urls.mux.Lock()
	defer urls.mux.Unlock()
	return urls.List[url]
}

func (urls URLs) Set(url string) {
	urls.mux.Lock()
	urls.List[url] = true
	urls.mux.Unlock()
}

// Crawl uses fetcher to recursively crawl
// pages starting with url, to a maximum of depth.
func Crawl(url string, depth int, fetcher Fetcher, fetched *URLs, res chan string) {
	defer close(res)

	if depth <= 0 {
		return
	}

	if fetched.Exists(url) {
		return
	}
	fetched.Set(url)

	body, urls, err := fetcher.Fetch(url)
	if err != nil {
		fmt.Println(err)
		return
	}
	
	res <- fmt.Sprintf("found: %s %q\n", url, body)

	// A parent routine will wait all children routine until they finished.
	// Otherwise the main func will just exit
	childrenRes := make([]chan string, len(urls))
	for i, _ := range childrenRes {
		childrenRes[i] = make(chan string)
	}
	for i, u := range urls {
		go Crawl(u, depth-1, fetcher, fetched, childrenRes[i])
	}
	for i := range childrenRes {
		for r := range childrenRes[i] {
			res <- r
		}
	}
	return
}

func main() {
	fetched := &URLs{List: make(map[string]bool)}
	res := make(chan string)

	go Crawl("http://golang.org/", 4, fetcher, fetched, res)

	for r := range res {
		fmt.Printf(r)
	}
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
	"http://golang.org/": &fakeResult{
		"The Go Programming Language",
		[]string{
			"http://golang.org/pkg/",
			"http://golang.org/cmd/",
		},
	},
	"http://golang.org/pkg/": &fakeResult{
		"Packages",
		[]string{
			"http://golang.org/",
			"http://golang.org/cmd/",
			"http://golang.org/pkg/fmt/",
			"http://golang.org/pkg/os/",
		},
	},
	"http://golang.org/pkg/fmt/": &fakeResult{
		"Package fmt",
		[]string{
			"http://golang.org/",
			"http://golang.org/pkg/",
		},
	},
	"http://golang.org/pkg/os/": &fakeResult{
		"Package os",
		[]string{
			"http://golang.org/",
			"http://golang.org/pkg/",
		},
	},
}
