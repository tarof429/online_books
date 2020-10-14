# Part 2: Advanced

## Table of Contents

1. [Advanced](#advanced)
    1. [Goroutines](#goroutines)
	2. [Channels](#channels)
    3. [Closures](#closures)
	4. [Web](#web)
	5. [Json](#json)
	6. [Commands](#commands)
	7. [Testing](#testing)
	8. [Error handling](#errors)
	9. [Logging](#logging)
	10. [Networking](#networking)
	11. [Times and Dates](#timesanddates)
	12. [Internals](#internals)
	13. [Random](#random)
	14. [Data Structures](#datastructures)

---

## [Advanced](#advanced)

### [Goroutines](#goroutines)

A goroutine is a lightweight thread.

```go
func hello() {
	fmt.Println("Hello world goroutine")
}
func main() {
	go hello()
	fmt.Println("main function")
	time.Sleep(1 * time.Second)
}

/* output
main function
Hello world goroutine
*/
```

### [Channels](#channels)

Channels are a typed conduit for messages. The reader will block until the writer has finished writing, so explicit synchronization is not necessary.

```go
// Function to process messages in an array. We stop processing once we
// encounter the string 'stop' even if there are other messages
func processCmd(s []string, c chan int) {

	msgCount := 0

	for _, v := range s {
		if v == "stop" {
			break
		}
		msgCount++
	}
	c <- msgCount // Send msgCount to the channel
}

func main() {
	s := []string{"message a", "message b", "message c", "stop", "message d"}

	c := make(chan int)
	go processCmd(s, c)

	sum := <-c // Receive msgCount from the channel

	fmt.Println("message: ", sum)
}
```

Buffered channels set the length of the buffer length. Message counts exceeding this buffer length will result in a fatal error (however, goroutines don't seem to suffer this issue).

```go
// Function to process messages in an array. We stop processing once we
// encounter the string 'stop' even if there are other messages
func processCmd(s []string, c chan int) {

	msgCount := 0

	for index, v := range s {

		if index == len(s)-1 && v == "stop" {
			// If we've reached the last element and the message is "stop", don't count it
			break
		} else if v == "stop" {
			c <- msgCount // Send msgCount to the channel
			continue
		} else {
			msgCount++
		}
	}

	c <- msgCount // Send msgCount to the channel

	close(c)
}

func main() {
	s := []string{"message a", "message b", "message c", "stop", "message d", "stop", "message e"}

	c := make(chan int, 2)

	processCmd(s, c)
	//go processCmd(s, c) // doesn't have an issue with buffer length

	for v := range c {
		fmt.Println(v)
	}

}
```

We can check if the channel is closed by specifying a second variable when reading from channel. If ok is false, then there are no more values to receive and the channel is closed.

```go
// Function to count the length of a string, excluding blank spaces
func count(s string, c chan int) {

	counter := 0

	for _, r := range s {
		c := string(r)
		if c != " " {
			counter++
		}
	}

	c <- counter

	close(c)
}

func main() {
	s := "hello world"

	c := make(chan int)

	go count(s, c)

	count, ok := <-c // Receive msgCount from the channel

	// Check the channel for values. If there were any, then print the value
	if ok != false {
		fmt.Println(count)
	}
}
```

Below is an example of using a goroutine in a closure.

```go
func main() {
	s := "hello world"

	c := make(chan int)
	
	go func() {
		counter := 0

		for _, r := range s {
			c := string(r)
			if c != " " {
				counter++
			}
		}

		c <- counter
		close(c)
	}()

	count, ok := <-c // Receive msgCount from the channel

	// Check the channel for values. If there were any, then print the value
	if ok != false {
		fmt.Println(count)
	}

}
```

### [Closures](#closures)

Functions are first class citizens in go and can be passed and returned by functions like any other type.

```go
func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	sum := adder()
	for i := 0; i < 10; i++ {
		fmt.Println(sum(i))
	}
}

```

### [Web](#web)

A simple web server:

```go
package main

import (
	"fmt"
	"net/http"
)

func index(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello World")
}

func main() {
	http.HandleFunc("/", index)
	fmt.Println("Server startng...")
	http.ListenAndServe(":3000", nil)

}
```

### [JSON](#json)

Go has excellent support for encoding and decoding structs in JSON format. Suppose we want to generate a package.json to be consumed by npm, the package manager for node.js. 

```go
// Package represents the npm package and any dependencies
type Package struct {
	Name         string
	Version      string
	Dependencies []Dependency
}

// Dependency is a package on which a package depends on
type Dependency struct {
	Dependency string
	Version    string
}

func main() {
	var dependencies = []Dependency{{"my-dep", "^1.0.0"}, {"another_dep", "~2.2.0"}}
	p := Package{
		Name:         "my package",
		Version:      "1.0.0",
		Dependencies: dependencies,
	}

	fmt.Println(p)
}
/*
{my package 1.0.0 [{my-dep ^1.0.0}]}
*/
```

To generate JSON from Go, we can use the *Marshall()* method in the *json* package. All exported fields (fields that begin with a capital letter) will be marshalled. Any fields which are not exported will not be marshalled.

```go
import (
	"encoding/json"
	"fmt"
	"log"
)

// Package represents the npm package and any dependencies
type Package struct {
	Name         string
	Version      string
	Dependencies []Dependency
}

// Dependency is a package on which a package depends on
type Dependency struct {
	Dependency string
	Version    string
}

func main() {
	var dependencies = []Dependency{{"my-dep", "^1.0.0"}, {"another_dep", "~2.2.0"}}
	p := Package{
		Name:         "my package",
		Version:      "1.0.0",
		Dependencies: dependencies,
	}

	// Print the Package struct
	fmt.Println(p)

	// Use the json package to convert the Go struct to JSON (marshalling)
	data, err := json.Marshal(p)

	if err != nil {
		log.Fatal("JSON marshalling failed: ", err)
	}

	fmt.Printf("%s\n", data)
}
/*
{"Name":"my package","Version":"1.0.0","Dependencies":[{"Dependency":"my-dep","Version":"^1.0.0"},{"Dependency":"another_dep","Version":"~2.2.0"}]}
*/
```

Go supports the ability to *tag* struct fields to output correct JSON. This is useful if we want to provide an alternative name for the field. We can also use the *json.MarshallIndent()* to print a more human-friendly version of JSON.

```go
import (
	"encoding/json"
	"fmt"
	"log"
)

// Package represents the npm package and any dependencies
// Specify tags for the fields to output the correct json
type Package struct {
	Name         string       `json:"name"`
	Version      string       `json:"version"`
	Dependencies []Dependency `json:"dependencies"`
}

// Dependency is a package on which a package depends on
type Dependency struct {
	Dependency string `json:"dependency"`
	Version    string `json:"version"`
}

func main() {
	var dependencies = []Dependency{{"my-dep", "^1.0.0"}, {"another_dep", "~2.2.0"}}
	p := Package{
		Name:         "my package",
		Version:      "1.0.0",
		Dependencies: dependencies,
	}

	// Print the Package struct
	fmt.Println(p)

	// Use the json package to convert the Go struct to JSON (marshalling)
	data, err := json.MarshalIndent(p, "", "\t")

	if err != nil {
		log.Fatal("JSON marshalling failed: ", err)
	}

	fmt.Printf("%s\n", data)
}
/*
{
	"name": "my package",
	"version": "1.0.0",
	"dependencies": [
			{
					"dependency": "my-dep",
					"version": "^1.0.0"
			},
			{
					"dependency": "another_dep",
					"version": "~2.2.0"
			}
	]
}
*/
```

Below is some code to read package.json, convert it to a struct, then output it as JSON again. Note that we have made some adjustments compared to the previous code. Instead of using *Dependencies []Dependency* we use a map to define dependencies.

```go
package main

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"log"
	"os"
)

// Package represents the npm package and any dependencies
type Package struct {
	Name         string            `json:"name"`
	Version      string            `json:"version"`
	Dependencies map[string]string `json:"dependencies"`
}

func main() {
	data, err := ioutil.ReadFile("hello.json")

	if err != nil {
		fmt.Fprintf(os.Stderr, "%v\n", err)
		return
	}

	var p Package

	if err := json.Unmarshal(data, &p); err != nil {
		log.Fatalf("JSON unmarshalling failed: %s\n", err)
	}

	fmt.Println(p) // {my_package 1.0.0 map[another_dep:~2.2.0 my_dep:^1.0.0]}

	// Use the json package to convert the Go struct to JSON (marshalling)
	data, err = json.MarshalIndent(p, "", "\t")

	if err != nil {
		log.Fatal("JSON marshalling failed: ", err)
	}

	// Display the struct as a JSON object
	fmt.Printf("%s\n", data) 

	/*
	{
        "name": "my_package",
        "version": "1.0.0",
        "dependencies": {
                "another_dep": "~2.2.0",
                "my_dep": "^1.0.0"
        }
	}
	*/
}
```



### [Commands](#commands)

The *os* package has functions for executing commands and dealing with their input and output. Note that the *Command()* function takes a variable number of strings as arguments.

```go
package main

import (
	"fmt"
	"os/exec"
	"strings"
)

// ListLogFiles reads from /var/log
func ListLogFiles() (string, error) {

	// Pass a variable number of strings to exec.Command()
	out, err := exec.Command("ls", "-F", "/var/log").Output()

	if err != nil {
		return "", err
	}

	entries := strings.Split(string(out), "\n")

	var ret string

	// List files and skip over directories
	for _, entry := range entries {
		entry = strings.Trim(entry, "")
		if len(entry) > 0 && strings.LastIndex(entry, "/") == -1 {
			ret += entry + "\n"
		}
	}

	// Return the output and trim leading and trailing newlines
	return strings.Trim(ret, "\n"), nil

}

func main() {
	data, err := ListLogFiles()

	if err != nil {
		fmt.Println("Failed to read log file directory!")
	}
	fmt.Println(data)
}
```

### [Testing](#testing)

In Go, tests are written in a file with the _test.go suffix. Each test file contains functions in the form *func TestName(t *testing.T)*.

Below is a tst for a function that creates abbreviations.

```go
func Abbreviate(s string) string {
	abbreviation := ""

	// Splits a string around one or more whitespace
	fields := strings.Fields(s)

	for _, element := range fields {
		firstLetter := strings.Split(element, "")[0]
		abbreviation += strings.ToUpper(firstLetter)
	}
	return abbreviation
}

// abbreviate_test.go
...
import (
	"testing"
)

func TestBasic(t *testing.T) {
	output := Abbreviate("Portable Network Graphics")

	if output != "PNG" {
		t.Error(output + ` was not PNG`)
	}
}

func TestLowercaseWords(t *testing.T) {
	output := Abbreviate("Don't communicate by sharing memory, share memory by communicating.")

	if output != "DCBSMSMBC" {
		t.Error(output + ` was not DCBSMSMBC`)
	}
}

func TestPunctuation(t *testing.T) {
	output := Abbreviate("First In, First Out")

	if output != "FIFO" {
		t.Error(output + ` was not FIFO`)
	}
}

func TestAllCapsWord(t *testing.T) {
	output := Abbreviate("GNU Image Manipulation Program")

	if output != "GIMP" {
		t.Error(output + ` was not GIMP`)
	}
}
...
/*
$ go test -v
=== RUN   TestBasic
--- PASS: TestBasic (0.00s)
=== RUN   TestLowercaseWords
--- PASS: TestLowercaseWords (0.00s)
=== RUN   TestPunctuation
--- PASS: TestPunctuation (0.00s)
=== RUN   TestAllCapsWord
--- PASS: TestAllCapsWord (0.00s)
PASS
ok      gopl.io/ch11/word2  0.002s
*/
```

This looks all well, but it fails when the word is surrounded by underscores.

```go
func TestUnderscores(t *testing.T) {
	output := Abbreviate("Go is _not_ Java")

	if output != "GINJ" {
		t.Error(output + ` was not GINJ`)
	}
}
...
=== RUN   TestUnderscores
    TestUnderscores: hello_test.go:43: GI_J was not GINJ
--- FAIL: TestUnderscores (0.00s)
FAIL
```

This prompts us to use the FieldsFunc() function instead so we can implement the function that will be used to parse the string. This example also makes use of function variables.

```go
package word

import (
	"strings"
	"unicode"
)

func Abbreviate(s string) string {
	abbreviation := ""

	// See https://golang.org/pkg/strings/#FieldsFunc for the original example
	f := func(c rune) bool {
		return !unicode.IsLetter(c) && !unicode.IsNumber(c) && c != '\''
	}

	// Splits a string around one or more whitespace
	fields := strings.FieldsFunc(s, f)

	for _, element := range fields {
		firstLetter := strings.Split(element, "")[0]
		abbreviation += strings.ToUpper(firstLetter)
	}
	return abbreviation
}

```

### Error Handling

Many functions in the Go ecosystem return an error data type. For example, the code snippet below opens a file. If there was an error, then the function will exit with an error message.

```go
f, err := os.Open("hello.txt")
defer f.Close()

if err != nil {
	fmt.Println("Unable to open hello.txt")
	os.Exit(1)
}
```

Let's refactor this program and move the code that opens the file to a function.

```go
func openConfig(fname string) error {
	f, err := os.Open(fname)
	defer f.Close()
	return err
}

func main() {

	err := openConfig("config.yml")

	if err != nil {
		fmt.Printf("%T\n", err)
		fmt.Println(err.Error())
	}
}

// Outuput 
// $ go run hello.go 
// *os.PathError
// open config.yml: no such file or directory
```

Notice that the error is of type os.PathError. What if we want to return our own error? Go recommends that we use the error.New() method to return our custom error.

```go
func openConfig(fname string) error {
	f, err := os.Open(fname)
	defer f.Close()

	if err != nil {
		return errors.New("Unable to open config file")
	}
	return nil
}
// $ go run hello.go 
// *errors.errorString
// Unable to open config file
```

Note that when we do this, the error is of type *errors.errorString. This may be okay if we are not concerned about the type of error being returned, only the message in the error. How do we implement our own error? 

The answer lies in looking at the sorce code for errors, at https://golang.org/src/errors/errors.go. An implementation is shown below. As you can see, what we essentially did was implement the errorString struct, implemented the Error() method on the struct, and a helper function which reeturns the address of the struct with our customized message.

```go

// ConfigFileOpenError defines a config file open error
type ConfigFileOpenError struct {
	s string
}

// Create a function Error() string and associate it to the struct.
func (error *ConfigFileOpenError) Error() string {
	return error.s
}

// NewConfigFileOpenError returns an error when a config file can't be opened.
func NewConfigFileOpenError(text string) error {
	return &ConfigFileOpenError{text}
}

func openConfig(fname string) error {
	f, err := os.Open(fname)
	defer f.Close()

	if err != nil {
		return NewConfigFileOpenError("Unable to open config file")
	}
	return nil
}

func main() {

	err := openConfig("config.yml")

	if err != nil {
		fmt.Printf("%T\n", err)
		fmt.Println(err.Error())
	}
}
// $ go run hello.go 
// *main.ConfigFileOpenError
// Unable to open config file
```

### [Logging](#logging)

Let's go back the previous example and just print the error to the console.

```go
func main() {
	f, err := os.Open("hello.txt")
	defer f.Close()

	if err != nil {
		fmt.Println(err.Error())
	}
}
```

Another way to print the error message is to use the Printf() function in the log package.

```go
func main() {
	f, err := os.Open("hello.txt")
	defer f.Close()

	if err != nil {
		log.Printf(err.Error())
	}
}
// $ go run hello.go 
// 2020/07/25 14:09:28 open hello.txt: no such file or directory
```

But what if we want to redirect the log messages to a file? 

To do this, we can use the *New()* function in the *log* package. The three arguments are: io.Writer, prefix, and flag.

```go
const (
	LOGFILE = "hello.log"
)

func main() {

	f, err := os.OpenFile(LOGFILE, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)

	defer f.Close()

	if err != nil {
		fmt.Printf("Unable to open %s\n", LOGFILE)
		os.Exit(1)
	}

	logger := log.New(f, "logger ", log.Lshortfile)

	logger.Println("Started hello")
}
// $ cat hello.log 
// logger hello.go:26: Started hello
```

According to https://golang.org/pkg/log, log.Lshortfile displays the prefix, filename and line number, and the log message, which is what was printed. Another flag to experiment with is LstdFlags, which prints the date and time.

```go
...
logger := log.New(f, "logger ", log.LstdFlags|log.Lshortfile)
...

// $ cat hello.log
// logger 2020/07/25 17:51:44 hello.go:26: Started hello

```

Incidentally, the order in which the flags are listed does not change the format. 

The logger returned by *New() is not to be confused with more sophisticated loggers. Unlike *log4j*, as it does not have the ability to indicate log level, nor can it be configured using a configuration file. According tot he documentation, it does claim to have the distinction to be safe to use from multiple goroutines. Additionally, the log package has functions including *Fatalf()* which will exit the program after logging a message. Hence, 

```go
logger := log.New(f, "logger ", log.Lshortfile|log.LstdFlags)

logger.Fatalf("Exiting program")

logger.Println("Started hello")

// $ cat hello.log
// logger 2020/07/25 18:03:38 hello.go:26: Exiting program
```

Will generate a line in the log file with the message "Exiting program", but will not log the message "Starting hello".

### [Networking](#networking)

According to https://golang.org/pkg/net/, there are two main functions associated with the *net* package. The first is *Dial()*, which allows Go programs to connect to a server.

First, let's run nginx using docker. We'll configure it to listen to port 80.

```bash
$ docker pull nginx
...
$  docker run -t --name docker-nginx -p 80:80 nginx
```

Point your browser to http://localhost and verify that you are greeted with the default nginx web page.

Next, let's use the Dial() function to connect to this webserver. We will make use of the log package to log messages to the console.

```go
func main() {

	logger := log.New(os.Stdout, "logger ", log.LstdFlags|log.Lshortfile)

	conn, err := net.Dial("tcp", "localhost:80")
	if err != nil {
		logger.Fatalf("Unable to connect to web server")
	}

	fmt.Fprintf(conn, "GET / HTTP/1.0\r\n\r\n")

	defer conn.Close()

	reader := bufio.NewReader(conn)

	for {
		s, err := reader.ReadString('\n')

		if err != nil {
			break
		}
		fmt.Print(s)
	}
}
// Output:
// $ go run hello.go 
// HTTP/1.1 200 OK
// Server: nginx/1.19.1
// Date: Mon, 27 Jul 2020 19:16:08 GMT
// Content-Type: text/html
// Content-Length: 612
// Last-Modified: Tue, 07 Jul 2020 15:52:25 GMT
// Connection: close
// ETag: "5f049a39-264"
// Accept-Ranges: bytes
// 
// <!DOCTYPE html>
// <html>
// <head>
// <title>Welcome to nginx!</title>
// ...
```

As you can see, functions from the net package offers low level networking primitives. A program to create a server also illustrates this.

```go

func handleConnection(conn net.Conn) {
	defer conn.Close()

	var buf [512]byte
	for {
		n, err := conn.Read(buf[0:])
		if err != nil {
			conn.Close()
			return
		}

		s := string(buf[0:n])
		fmt.Println(s)

		tokens := strings.Split(s, "\n")

		for _, token := range tokens {
			if len(token) > 0 {
				fmt.Println("Token: ", token)
			}
		}

	}
}

func main() {

	logger := log.New(os.Stdout, "logger ", log.LstdFlags|log.Lshortfile)

	ln, err := net.Listen("tcp", ":8080")

	if err != nil {
		logger.Fatalf("Unable to create web server")
	}

	for {
		conn, err := ln.Accept()

		if err != nil {
			logger.Fatalf("Unable to listen")
		}
		go handleConnection(conn)
	}

}

// output
// $ go run hello.go 
// GET /hello-world HTTP/1.1
// Host: localhost:8080
// User-Agent: curl/7.71.1
// Accept: */*
```

If we want to be productive with network programming, we may want to look into the http package. To create a server, we will use the ListenAndServe() function to serve HTTP traffic.

```go
func main() {

	logger := log.New(os.Stdout, "logger ", log.LstdFlags|log.Lshortfile)

	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		logger.Fatalf("Unable to serve: %s\n", err.Error())
	}

}
```

The second argument to *ListenAndServe() is the handler, which is typically nil, in which case the DefaultServeMux is used.  

Alternatively, we could create an instance of an http.Server to serve requests.

```go
s := &http.Server{
	Addr:           ":8080",
	ReadTimeout:    10 * time.Second,
	WriteTimeout:   10 * time.Second,
	MaxHeaderBytes: 1 << 20,
}
logger.Fatal(s.ListenAndServe())
```

However, we are still not able to handle requests. What we need to do is implement our own handlers and invoke the *http.HandleFunc()* function on them.

```go
func handleRoot(w http.ResponseWriter, r *http.Request) {
	fmt.Println("Invoked handleRoot()")
}

func main() {

	logger := log.New(os.Stdout, "logger ", log.LstdFlags|log.Lshortfile)

	s := &http.Server{
		Addr:           ":8080",
		ReadTimeout:    10 * time.Second,
		WriteTimeout:   10 * time.Second,
		MaxHeaderBytes: 1 << 20,
	}

	http.HandleFunc("/", handleRoot)

	logger.Fatal(s.ListenAndServe())

}
```

If we don't need to configure the server, we could just invoke *ListenAndServe()* from the http package directly.

```go
func main() {

	logger := log.New(os.Stdout, "logger ", log.LstdFlags|log.Lshortfile)

	http.HandleFunc("/", handleRoot)

	logger.Fatal(http.ListenAndServe(":8080", nil))
}
```

In our handler, we could return an HTML response using the provided *http.ResponseWriter*.

```go
func handleRoot(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "<html><body>Hello world!</body></html>")
}
```

As demonstrated above, the HandleFunc function makes it easy to handle requests. Let's add another handler.

```go

func handleHello(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "<html><body>Hello, you have contacted the server</body></html>")
}

func main() {
...
  http.HandleFunc("/hello", handleHello)
  logger.Fatal(http.ListenAndServe(":8080", nil))
}
```

So far, our handlers returned HTML directly. But what if we want to server HTML files instead? To do this, we can creaste a FileServer.

```go
func main() {

	logger := log.New(os.Stdout, "logger ", log.LstdFlags|log.Lshortfile)

	http.Handle("/", http.FileServer(http.Dir(".")))

	logger.Fatal(http.ListenAndServe(":8080", nil))
}
```

This is fine if we just want to serve static files. But now what if we want the user to be able to supply paramenters? Go supports this through the use of templates.


```go
// User is a user
type User struct {
	Name string
}

func handleRoot(w http.ResponseWriter, r *http.Request) {

	tmpl, err := template.New("test").Parse("<html><body>Hello my name is {{.Name}}</body></html>")

	if err != nil {
		panic(err)
	}

	name := r.FormValue("name")

	user := User{name}

	err = tmpl.ExecuteTemplate(w, "rootTemplate", user)

}

func main() {

	logger := log.New(os.Stdout, "logger ", log.LstdFlags|log.Lshortfile)

	http.HandleFunc("/hello", handleRoot)

	logger.Fatal(http.ListenAndServe(":8080", nil))
}
// go run hello.go
// curl http://localhost:8080/hello?name=Taro
// <html><body>Hello my name is Taro</body></html>
```

Let's see how we can improve on this. We want to be able to process a template stored on the file system. To do this, we need to change our code to read the file and extract the bytes.

```go
// User is a user
type User struct {
	Name string
	Age  string
}

func handleRoot(w http.ResponseWriter, r *http.Request) {

	b, err := ioutil.ReadFile("index.html")

	if err != nil {
		panic(err)
	}

	t, err := template.New("rootTemplate").Parse(string(b))

	name := r.FormValue("Name")
	age := r.FormValue("Age")

	user := User{name, age}

	fmt.Println(user)

	err = t.ExecuteTemplate(w, "rootTemplate", user)

	if err != nil {
		fmt.Println(err.Error())
	}
}

func main() {

	logger := log.New(os.Stdout, "logger ", log.LstdFlags|log.Lshortfile)

	http.HandleFunc("/hello", handleRoot)

	logger.Fatal(http.ListenAndServe(":8080", nil))
}
//  curl "http://localhost:8080/hello?Name=Taro&Age=25"
// <!DOCTYPE html>
// <html>
//    <head>
//        <meta charset="UTF-8">
//        <title>Hello</title>
//    </head>
//    <body>
//        Hello Taro I am 25 years old
//    </body>
// </html>
```

### [Times and Dates](#timesanddates)

Most often when we want to deal with times and dates in Go, we need to find the current time. 

```go
func main() {

	fmt.Println(time.Now())
}
// 2020-07-28 09:33:44.445612865 -0700 PDT m=+0.000062648
```

To format this value, we need to call the *Format()* method. *The Format()* method takes a reference time as an argument. For example, the code below prints the current time and date but excludes timezone and milliseconds.

```go
func main() {

	t := time.Now()

	s := t.Format("15:04:05 Mon Jan 2 2006")

	fmt.Println(s)

}
// 10:14:54 Tue Jul 28 2020
```

One use of the time package is to measure the amount of time that has elapsed. This is achieved by invoking the *Sub()* method on the end time value.

```go

func main() {

	start := time.Now()

	time.Sleep(time.Second)

	end := time.Now()

	duration := end.Sub(start)

	fmt.Printf("%v\n", duration)
}
// 1.000189175s
```

Alternatively, Go provides a shortcut in the form of the *Since()* function which we can invoke on the Duration struct returned from *Now()* at the beginning of the program.

```go
duration := time.Since(start)
```

As seen above, the Duration is a floating point number and is not human friendly. We could try getting just the seconds from this value.

```go
...
fmt.Println(duration.Seconds())
...
// 1.00041425
```

But again we must deal with a floating point number. 

There are two aproaches to convert floating point numbers to a whole number. One is using the *Floor()* function in the math package.

```go
...
fmt.Println(math.Floor(duration.Seconds()))
...
// 1
```

Another is to use the FormatFloat() function in the *strconv* package.

```go
...
seconds := strconv.FormatFloat(duration.Seconds(), 'f', 0, 64)
fmt.Println(seconds)
...
// 1
```

Which is better? Maybe neither, but keep in mind that *math.Floor()* returns a float64 value, while *strconv.FormatFloat()* returns a string. 

### [Internals](#internals)

A Go program is compiled to a native executable file. However at runtime, the program is run within a Go runtime. This runtime is responsible for running applications as well as managing resources such as memory.

The Go garbage collector is responsible for managing memory. Specifically, the garbage collector periodically checks whether objects become out of scope and cannot be referenced any more. 

The *GOGC* environment variable controls the aggressiveness of the garbage collector. The default value of GOGC is 100, meaning the garbage collector will run every time the heap grows by 100%. It follows that if we set this value to 200, then the garbage collector will run every time the heap grows by 200%.

When the garbage collector runs, execution of the program is suspended. This adds latency to the program. The algorithm used by the garbage collector is called mark-and-sweep. All objects in the heap are visited and potentially marked for garbage collection.

The official name for the algorithm used in Go is the *tricolor mark-and-sweep algorithm*. The garbage collector runs in a low-latency goroutine to have minimal impact on a program's performance.

### [Random](#random)

Below is an adaptation of a code snipped from https://golang.org/pkg/math/rand/. It shows the use of several techniques in creating random numbers:

- Creating a stable random generator by using *rand.New()*
- Using a tabwriter to generated formattted output

```go
import (
	"fmt"
	"math/rand"
	"os"
	"text/tabwriter"
)

func main() {
	// Create the random seed generator. Using
	// a fixed seed ensures the same output on every run
	r := rand.New(rand.NewSource(99))

	// Create a tabwriter to help us write tabbed output
	w := tabwriter.NewWriter(os.Stdout, 1, 1, 1, ' ', 0)

	defer w.Flush()

	show := func(key string, value interface{}) {
		fmt.Fprintf(w, "%s\t%v\n", key, value)
	}

	show("Node", r.Intn(10))
	show("Count", r.Intn(10))
	show("Processes", r.Intn(10))
}
/*
Node      7
Count     3
Processes 0
*/
```

The random package has a function which can generate permutations of a number between 0 and n. If we are interested in generating a list of unique numbers within a range this can be useful.

```go

func main() {
	// Create the random seed generator. Use the current time
	// as the source to ensure that the generated numbers are different
	// every time the program is run.
	r := rand.New(rand.NewSource(time.Now().UnixNano()))

	// Create a tabwriter to help us write tabbed output
	w := tabwriter.NewWriter(os.Stdout, 1, 1, 1, ' ', 0)

	defer w.Flush()

	show := func(key string, value interface{}) {
		fmt.Fprintf(w, "%s\t%v\n", key, value)
	}

	ids := r.Perm(10)

	for i := range ids {
		show("ID:", ids[i])
	}

}
/*
ID: 4
ID: 2
ID: 7
ID: 3
ID: 6
ID: 5
ID: 1
ID: 8
ID: 0
ID: 9
*/
```

Suppose, however, that we want to generate a random ID consisting of values between 0-9 and the letters a through f. To do this, we should create a slice and populate each element one by one.

```go
func Init() {
	rand.Seed(time.Now().UnixNano())
}

func IDGenerator() string {
	const letters = "abcdef0123456789"

	b := make([]byte, 10)

	for i := range b {
		b[i] = letters[rand.Intn(len(letters))]
	}
	return string(b)

}

func main() {

	// Create a tabwriter to help us write tabbed output
	w := tabwriter.NewWriter(os.Stdout, 1, 1, 1, ' ', 0)

	defer w.Flush()

	show := func(key string, value interface{}) {
		fmt.Fprintf(w, "%s\t%v\n", key, value)
	}

	Init()

	i := 0

	for i < 10 {
		show("ID:", IDGenerator())
		i++
	}

}

```

[Data Structures](#datastructures)

Below is an example of a circular linked list. The factory function returns an empty LinkedList by value. What seems to be important here is that factory fuctions cannot be methods. When adding a node, we make sure that the new node's next value points to the head of the list instead of null. 

```go
type Node struct {
	data int
	next *Node
}

type LinkedList struct {
	head *Node
	tail *Node
}

func NewList() LinkedList {
	var list LinkedList
	return list
}

func (list *LinkedList) Add(data int) {
	node := Node{data: data}

	if list.head == nil {
		list.head = &node
		node.next = &node
		list.tail = &node
	} else {
		node.next = list.head
		list.tail.next = &node
		list.tail = &node

	}
}

func (list *LinkedList) List() {

	cur := list.head

	headAddr := list.head

	for cur != nil {
		fmt.Println(cur.data)
		cur = cur.next

		if cur == headAddr {
			break
		}

	}
}

func (list *LinkedList) Visit() {

	cur := list.head

	for cur != nil {
		fmt.Println(cur.data)
		cur = cur.next
		time.Sleep(100 * time.Millisecond)
	}
}

func main() {
	list := NewList()
	list.Add(2)
	list.Add(3)
	list.Add(4)
	list.Visit()
}
```

## References

https://www.amazon.com/Isomorphic-Go-isomorphic-applications-programming/dp/1788394186/ref=sr_1_7?dchild=1&keywords=golang&qid=1593625550&sr=8-7

https://www.amazon.com/Go-Practice-Techniques-Matt-Butcher/dp/1633430073/ref=sr_1_15?dchild=1&keywords=golang&qid=1593625550&sr=8-15

[Mastering Go](https://www.amazon.com/dp/B07WC24RTQ/)

https://dave.cheney.net/tag/gogc
