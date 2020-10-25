# Part 1: Installation and the Basics

## Table of Contents

1.  [Installation](#installation)
2.  [Basics](#basics)
    1. [Packages](#packages)
	2. [Variables](#variables)
	3. [Flow control statements](#flow-control-statements)
	4. [Pointers](#pointers)
	5. [Structs](#structs)
	6. [Arrays](#arrays)
	7. [Slices](#slices)
	8. [Maps](#maps)
	9. [Functions](#functions)
	10. [Methods](#methods)
	11. [Files](#files)

## [Installation](#installation)

Install Go using ArchLinux' package manager.

```bash
$ pacman -S go
```

At the time of this writing, go is still version 1.x. 

```bash
$ go version
go version go1.15.2 linux/amd64
$ pacman -Q go
go 2:1.15.2-1
```

Normally, Go dependencies are installed under $HOME/go. This can be set by setting GOPATH. 

In the past, code also had to be written under GOPATH. However, with go modules, this is no longer necessary. I still have GOPATH pointing to $HOME/go, but I develop my code under $HOME/Code/Go.

## [Basics](#basics)

### [Packages](#packages)

Every Go program is made up of packages. The package that starts running programs is in package *main*. 

```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("Hello world!")
}
```

To run:

```go
go run main.go
```

To build:

```go
go install
```

The compiled executable will be placed under GOPATH/bin.

Imports are typically declared within parentheses. This is called a factored import statement.

```go
import (
	"fmt"
	"math"
)

func main() {
	var name = "Taro"
	var age = 37
	var length = 140.1

	fmt.Println(name, "is", age, "and", math.Floor(length), "and", math.Ceil(length))
	fmt.Printf("%T %T %T\n", name, age, length)
}
/*
Taro is 37 and 140 and 141
string int float64
*/
```

The name of methods must be capitalized to be accessible from other packages. Such methods are said to be *exported*.

```go
func main() {
	fmt.Println(math.Pi) // and not math.pi
}
```

To create our own package, create a file under GOPATH (for example, /home/taro/go/src/github.com/tarof429/go_crash_course/03_packages/strutil/reverse.go) and add our code. The function needs to be capitalized.

```go
package strutil

func Reverse(s string) string {
	runes := []rune(s)
	for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
		runes[i], runes[j] = runes[j], runes[i]
	}
	return string(runes)
}
```

Then to use it within our code:

```go
import (
	"fmt"

	"github.com/tarof429/go_crash_course/03_packages/strutil"
)

func main() {
	var greet = "hello"
	fmt.Println(strutil.Reverse(greet))
}
```

### [Variables](#variables)

The var statement can be used to declare variables. 

```go
func main() {
	var name string = "Taro"

	fmt.Println(name)
}
```

The go compiler will warn that the type is inferred to be string, so actually we don't need to specify the type in this case. Next we can create a variable of type int.

```go
func main() {
	var name = "Taro"
	var age = 37

	fmt.Println(name, "is", age)
}
// Taro is 37
```

To print variable types, use %T.

```go
fmt.Printf("%T %T\n", name, age)
// string int
```

The keyword *var* is required if declaring variables outside of a function.

```go
var remainder int

func getChange(coin int, value int) (change int, remainder int) {
	var minValue = 1

	if value < minValue {
		fmt.Println("No value!")
		return
	}

	change = coin / value
	remainder = coin % value

	return
}

func calculateChange(quarters int) (dimes int, nickels int, pennies int) {

	// The message that we want to print
	var message string

	// Get the number of dimes
	dimes, remainder = getChange(quarters, 10)

	// Get the number of nickels
	if remainder > 0 {
		nickels, remainder = getChange(remainder, 5)
	}

	// The remainder is the number of pennies that we have
	pennies = remainder

	if dimes > 0 && nickels > 0 && pennies > 0 {
		message = "All three coins!"
	} else {
		message = "Not all three coins!"
	}

	fmt.Println(message)

	return
}

func main() {

	dimes, nickels, pennies := calculateChange(45)

	fmt.Println("dimes, nickels, pennies: ", dimes, nickels, pennies)

	dimes, nickels, pennies = calculateChange(78)

	fmt.Println("dimes, nickels, pennies: ", dimes, nickels, pennies)

	dimes, nickels, pennies = calculateChange(7)

	fmt.Println("dimes, nickels, pennies: ", dimes, nickels, pennies)

	dimes, nickels, pennies = calculateChange(0)

	fmt.Println("dimes, nickels, pennies: ", dimes, nickels, pennies)
}
/* 
Output:
Not all three coins!
dimes, nickels, pennies:  4 1 0
All three coins!
dimes, nickels, pennies:  7 1 3
Not all three coins!
dimes, nickels, pennies:  0 1 2
Not all three coins!
dimes, nickels, pennies:  0 0 0
*/
```

#### Short variable declaration

Inside a function, *\:\=* can be used in place of using *var* to declare variables. The type is implicit.

```go
func main() {
	var i, j int = 1, 2
	k := 3 // shorthand
	c, python, java, golang := true, false, "no!", "yess!!"

	fmt.Println(i, j, k, c, python, java, golang)
}
```

#### Declaring variables within blocks

Multiple variables can be *factored* into blocks.

```go
var (
	status      bool
	message     string
	exitCode    int
	testsPassed int
	testsFailed int
)

func main() {

	status = true
	message = "All tests passed"
	exitCode = 0
	testsPassed = 30
	testsFailed = 0

	fmt.Println(status, message, exitCode, testsPassed, testsFailed)

	status = false
	message = "Tests failed"
	exitCode = 1
	testsPassed = 29
	testsFailed = 1

	fmt.Println(status, message, exitCode, testsPassed, testsFailed)
}
```

#### Zero Values

Variables have zero values which means a default value if none is assigned to them. That is, ints and floats default to 0, bool defaults to false, and a string defaults to an empty string.

```go
var (
	status      bool
	message     string
	exitCode    int
	testsPassed int
	testsFailed int
)

func main() {

	fmt.Println(status, message, exitCode, testsPassed, testsFailed)

}
// false  0 0 0
```

#### Constants

Constants can be defined using the *const* keyword. Its value cannot be changed and you will get a comple-time error if you attempt to do so.

```go
const pi = 3.14

func main() {
	radius := 7.0

	area := pi * radius * radius

	fmt.Println("Area of cirlce: ", area)

}
```

We can declare multiple constants within () much like import statements.

```go
const (
	Monday = 0
	Tuesday = 1
)
```

### [Flow control statements](#flow-control-statements)

#### For statement

Go has only one looping construct, the *for* loop. This is in contrast to other languages such as Python, which also has as a *while* loop. 

```go
package main

import "fmt"

func main() {
	for i := 0; i < 10; i++ {
		fmt.Println("Hello world!")
	}
}
```

Breaking down the for statement, we have three parts: init, condition, and post. Of these, only the middle (condition) is required, and semicoons are unnecessary.

```go
	i := 0
	for i < 10 {
		fmt.Println("Hello world!")
		i++
    }
```

Without the conditional statement, we can write an infinite loop.

```go
	i := 0
	for {
		fmt.Println("Hello world!")
		i++

		if i > 10 {
			break
		}
    }
```

Below is an implementation of fizzbuzz, which is described below (courtesy of https://wiki.c2.com/?FizzBuzzTest):

```go
/* 
Write a program that prints the numbers from 1 to 100. But for multiples of three print “Fizz” instead of the number and for the multiples of five print “Buzz”. For numbers which are multiples of both three and five print “FizzBuzz”.
*/
func main() {
	for i := 1; i <= 100; i++ {
		if i%15 == 0 {
			fmt.Println("fizzbuzz")
		} else if i%3 == 0 {
			fmt.Println("fizz")
		} else if i%5 == 0 {
			fmt.Println("buzz")
		} else {
			fmt.Println(i)
		}
	}
}
```

#### If statement

If statements evaluate some condition; the body needs to be within braces.

```go
func main() {
	var x = 10
	var y = 20

	if x < y {
		fmt.Println("x is less than y")
	}
}
```

Below is another example, as used within a function.

```go
func divide(x float64, y float64) string {
	if y != 0 {
		return fmt.Sprint(x / y)
	}
	return "Unable to divide by zero"
}
```

Multiple if else statements can be chained as in other languages.

```go
func menu(choice string) string {
	if choice == "Y" {
		return "You have entered Y"
	} else if choice == "N" {
		return "You have entered N"
	} else if choice == "Q" {
		return "You have entered Q"
	} else {
		return "Invalid choice"
	}
}
```

#### Switch statement

Switch statements can be used to evaluate a variable. The switch statement breaks automatically after the first match. The code below initializes and evaluates the arch variable.

```go
func findOS() {
	fmt.Print("Go runs on ")
	switch arch := runtime.GOARCH; arch {
	case "amd64":
		fmt.Println("AMD64.")
	case "sparc64":
		fmt.Println("Sparc64.")
	default:
		// others
		fmt.Printf("%s.\n", arch)
	}
}
```

Another example:

```go

func whenIsFriday() {

	currentday := time.Now().Weekday()

	switch string(currentday) {

	case "Friday":
		fmt.Println("Today is Friday!")
	case "Thursday":
		fmt.Println("Tomorrow is Friday!")
	case "Saturday":
		fmt.Println("Yesterday was Friday!")
	default:
		fmt.Println("Not today!")
	}
}
```

Another variation, where the evaluation is done in each case statement:

```go
func whenIsFriday() {

	currentday := time.Now().Weekday()

	switch {
	case string(currentday) == "Friday":
		fmt.Println("Today is Friday!")
	case string(currentday) == "Thursday":
		fmt.Println("Tomorrow is Friday!")
	case string(currentday) == "Saturday":
		fmt.Println("Yesterday was Friday!")
	default:
		fmt.Println("Not today!")
	}
}
```

#### Defer

A defer is a way to push statements onto a stack. Execution is delayed untill the code outside has completed.

```go
func willIGetAnInterview(day string) string {
	opportunity := "Unknown"

    defer fmt.Println("Checking with the recruiter...")
    
    defer fmt.Println("Checking what day it is...")

	switch day {
	case "Tuesday":
		opportunity = "Probably"
	default:
		opportunity = "Probably not"
	}

	return opportunity
}
/*
Checking what day it is...
Checking with the recruiter...
Probably
*/
```

The defer puts statements onto a stack and is an example of a LIFO. This means that the last statement put on the stack will be evaluated first.

### [Pointers](#pointers)

Pointers in Go are similar to pointers in C. If you want to change the value of a pointer you need to dereferrence it using *\**.

```go
func main() {
	num := 3
	p := &num
	*p = 19
	fmt.Println(num)
}
```

Below is an example where we use pointers with a function.

```go
func getMarried(pFirstName *string, pLastName *string, pSpousesLastName *string) {
	fmt.Println(pFirstName, pLastName, pSpousesLastName)

	*pLastName = *pSpousesLastName
}

func main() {
	firstName := "Mary"
	lastName := "Jane"
	spousesLastName := "James"

	getMarried(&firstName, &lastName, &spousesLastName)
	fmt.Println(&firstName, &lastName, &spousesLastName)
	fmt.Println(firstName, lastName)
}
/*
0xc000010210
0xc000010200 0xc000010210 0xc000010220
Mary James
[taro@zaxman golang]$ go run hello.go 
0xc000010200 0xc000010210 0xc000010220
0xc000010200 0xc000010210 0xc000010220
Mary James
*/
```

### [Structs](#structs)

A struct is an aggregate data type that groups together zero or more fields. As a simple example, we can consider a struct representing a phone.

```go
import (
	"fmt"
	"time"
)

// Phone represents a cell phone
type Phone struct {
	ID      int
	Brand   string
	Model   string
	Color   string
	Ranking int
	Seller  string
	price   float64
	DoI     time.Time
}

func main() {
	var iphone Phone
	iphone.ID = 100
	iphone.Brand = "Apple"
	iphone.Model = "IPhone SE"
	iphone.Color = "Silver"
	iphone.Ranking = 29
	iphone.Seller = "Apple"
	iphone.price = 399.0
	iphone.DoI = time.Date(2009, time.November, 10, 23, 0, 0, 0, time.UTC)

	// Print the value of the iPhone
	fmt.Println(iphone)

	// In December, Apple decided to discount the iPhone to help sales
	iphone.price = 350.0

	// At the same time, the color names were changed to make them more attractive
	// Here, we create a pointer to the iPhone and change its color via pointer.
	pColor := &iphone.Color
	*pColor = "Bright " + *pColor

	// Subsequently, sales increased! We create a pointer to the struct.
	// Accesing fields from the pointer only require the use of dot notation.
	pIphone := &iphone
	pIphone.Ranking = 25

	fmt.Println(iphone)

	// Create another phone from a rival maker
	var samsung = Phone{101, "Sansung", "Galaxy", "Black", 25, "Samsung",
		99.0, time.Date(2009, time.November, 10, 23, 0, 0, 0, time.UTC)}

	// Compare the iPhone with the Samsung. Comparing structs is supported.
	if iphone == samsung {
		fmt.Println("iPhone and Samsung are the same phone")
	} else if iphone.DoI == samsung.DoI {
		fmt.Println("iPhone and Samsung were released on the same date")
	}

	// Create an instance of a phone but defer assigning values to it
	var unknownBrand = Phone{}

	// Now lets set the brand of our unknown phone
	unknownBrand.Brand = "Unknown"
}
```

Now suppose we want to associate a phone with its owner, so we add some more fields to it.

```go
// Phone represents a cell phone
type Phone struct {
	ID      int
	Brand   string
	Model   string
	Color   string
	Ranking int
	Seller  string
	price   float64
	DoI     time.Time

	// Fields representing a dude
	FirstName string
	LastName  string
	Age       int
}
```

We would access these fields using the dot notation that we've already been using.

But maybe it makes more sense to put these fields in another struct of type *Dude*. This change makes accessing the fields in the Dude struct more verbose.

```go
import (
	"fmt"
	"time"
)

// Phone represents a cell phone
type Phone struct {
	ID      int
	Brand   string
	Model   string
	Color   string
	Ranking int
	Seller  string
	price   float64
	DoI     time.Time
	Dude    Dude
}

// Fields representing a dude
type Dude struct {
	FirstName string
	LastName  string
	Age       int
}

func main() {
	var iphone Phone
	iphone.ID = 100
	iphone.Brand = "Apple"
	iphone.Model = "IPhone SE"
	iphone.Color = "Silver"
	iphone.Ranking = 29
	iphone.Seller = "Apple"
	iphone.price = 399.0
	iphone.DoI = time.Date(2009, time.November, 10, 23, 0, 0, 0, time.UTC)
	iphone.Dude.FirstName = "Steve"
	iphone.Dude.LastName = "Jobs"
	iphone.Dude.Age = 41

	// Print the value of the iPhone
	fmt.Println(iphone)

	// In December, Apple decided to discount the iPhone to help sales
	iphone.price = 350.0

	// At the same time, the color names were changed to make them more attractive
	// Here, we create a pointer to the iPhone and change its color via pointer.
	pColor := &iphone.Color
	*pColor = "Bright " + *pColor

	// Subsequently, sales increased! We create a pointer to the struct.
	// Accesing fields from the pointer only require the use of dot notation.
	pIphone := &iphone
	pIphone.Ranking = 25

	fmt.Println(iphone)

	// Create another phone from a rival maker
	var samsung = Phone{101, "Sansung", "Galaxy", "Black", 25, "Samsung",
		99.0, time.Date(2009, time.November, 10, 23, 0, 0, 0, time.UTC), Dude{"John", "Tesh", 41}}

	// Compare the iPhone with the Samsung. Comparing structs is supported.
	if iphone == samsung {
		fmt.Println("iPhone and Samsung are the same phone")
	} else if iphone.DoI == samsung.DoI {
		fmt.Println("iPhone and Samsung were released on the same date")
	}

	// Create an instance of a phone but defer assigning values to it
	var unknownBrand = Phone{}

	// Now lets set the brand of our unknown phone
	unknownBrand.Brand = "Unknown"
}
```

Go lets us declare a field with no name. Such fields are called *anonymous fields*.

```go
import (
	"fmt"
	"time"
)

// Phone represents a cell phone
type Phone struct {
	ID      int
	Brand   string
	Model   string
	Color   string
	Ranking int
	Seller  string
	price   float64
	DoI     time.Time
	Dude
}

// Fields representing a dude
type Dude struct {
	FirstName string
	LastName  string
	Age       int
}

func main() {
	var iphone Phone
	iphone.ID = 100
	iphone.Brand = "Apple"
	iphone.Model = "IPhone SE"
	iphone.Color = "Silver"
	iphone.Ranking = 29
	iphone.Seller = "Apple"
	iphone.price = 399.0
	iphone.DoI = time.Date(2009, time.November, 10, 23, 0, 0, 0, time.UTC)
	iphone.FirstName = "Steve"
	iphone.LastName = "Jobs"
	iphone.Age = 41

	// Print the value of the iPhone
	fmt.Println(iphone)

	// In December, Apple decided to discount the iPhone to help sales
	iphone.price = 350.0

	// At the same time, the color names were changed to make them more attractive
	// Here, we create a pointer to the iPhone and change its color via pointer.
	pColor := &iphone.Color
	*pColor = "Bright " + *pColor

	// Subsequently, sales increased! We create a pointer to the struct.
	// Accesing fields from the pointer only require the use of dot notation.
	pIphone := &iphone
	pIphone.Ranking = 25

	fmt.Println(iphone)

	// Create another phone from a rival maker
	var samsung = Phone{101, "Sansung", "Galaxy", "Black", 25, "Samsung",
		99.0, time.Date(2009, time.November, 10, 23, 0, 0, 0, time.UTC),
		Dude{"John", "Tesh", 41}}

	// Compare the iPhone with the Samsung. Comparing structs is supported.
	if iphone == samsung {
		fmt.Println("iPhone and Samsung are the same phone")
	} else if iphone.DoI == samsung.DoI {
		fmt.Println("iPhone and Samsung were released on the same date")
	}

	// Create an instance of a phone but defer assigning values to it
	var unknownBrand = Phone{}

	// Now lets set the brand of our unknown phone
	unknownBrand.Brand = "Unknown"
}
```

Alternatively we can declare and initialize structs at the same time.

```go
import (
	"fmt"
	"time"
)

// Phone represents a cell phone
type Phone struct {
	ID      int
	Brand   string
	Model   string
	Color   string
	Ranking int
	Seller  string
	Price   float64
	DoI     time.Time
	Dude
}

// Fields representing a dude
type Dude struct {
	FirstName string
	LastName  string
	Age       int
}

func main() {

	iphone := Phone{
		ID:      100,
		Brand:   "Apple",
		Model:   "IPhone SE",
		Color:   "Silver",
		Ranking: 29,
		Seller:  "Apple",
		Price:   399.0,
		DoI:     time.Date(2009, time.November, 10, 23, 0, 0, 0, time.UTC),
		Dude:    Dude{FirstName: "Steve", LastName: "Jobs", Age: 41},
	}

	// Print the value of the iPhone
	fmt.Println(iphone)
}
```

### [Arrays](#arrays)

An array is a collection of of a type. The size needs to be specified and it cannot be changed.

```go
func main() {
	var colors [3]string
	colors[0] = "red"
	colors[1] = "blue"
	colors[2] = "yellow"

	for i := 0; i < len(colors); i++ {
		fmt.Println(colors[i])
	}
}
```
We can use the range function to iterate through an array without computing its length.

```go
	...
	for index, element := range colors {
		fmt.Println(index, element)
	}
```

or more succintly if we only care about the value:

```go
	for _, element := range colors {
		fmt.Println(element)
	}
```

We can create an assign values to an array at the same time, in what's called an `array literal`.

```go
func main() {
	var fruit [2]string = [2]string{"apple", "orange"}
	for _, v := range fruit {
		fmt.Println(v)
	}
}
```

We could omit the var keyword and use `:=` instead.
```go
func main() {
	fruit := [2]string{"apple", "orange"}
	fmt.Println(fruit)
}
```

#### [Slices](#slices)

A slice is an array without a predefined size. 

```go
func main() {
	ids := [6]int{3, 5, 7, 11, 13}

	for _, v := range ids {
		fmt.Println(v)
	}
}

```

Specifying the size of a slice is optional.

```go
func main() {
	fruit := []string{"apple", "orange"}
	fmt.Println(fruit)
}
```

Below is an example where we sum all the elements of the slice.

```go
func main() {
	sum := 0

	ids := []int{3, 5, 7, 11, 13}

	for _, v := range ids {
		sum += v
	}
	fmt.Println(sum)
}
```

Below is another example based on the colors example where we print a subset of the slice.

```go
func main() {
	colors := []string{
		"red",
		"blue",
		"yellow",
		"green",
		"orange"}

	top3Colors := colors[0:3]

	for index, element := range top3Colors {
		fmt.Println(index, element)
	}
}
/*
0 red
1 blue
2 yellow
*/
```

Now an interesting thing about slices is that modifing an element in a slice will modify an element in the original array! This is actually a major difference between go and python.

```go
	top3Colors[0] = "purple"

	fmt.Println(colors[0])
/*
purple
*/
```

```python
scores = [10, 20, 30, 40, 50]
top3Scores = scores[0:3]
top3Scores[0] = 90

print(scores[0])
# 10
```

Now if we want an array of elements but don't want to specify the size, we can use a slice literal. The code below *looks* like we're creating an array, and we are. However the actual array is hidden from us; we are accessing it using a slice literal.

```go
func main() {
	colors := []string{
		"red",
		"blue",
		"yellow"}

	for index, element := range colors {
		fmt.Println(index, element)
	}
}
```

We can use structs within slices, making it easy to create "dummy data":

```go
func main() {
	jobs := []struct {
		translator     string
		status         string
		completionDate string
	}{
		{"John Doe", "Completed", "3/1/2019"},
		{"Mary Jane", "Completed", "7/15/2019"},
		{"Harry Potter", "In Progress", ""},
	}
	fmt.Println(jobs)
}
// [{John Doe Completed 3/1/2019} {Mary Jane Completed 7/15/2019} {Harry Potter In Progress }]
```

A *slice default* refers to shortcuts when declaring a slice. For example, we can creat a slice of our slice above to only print John's jobs.

```go
	johnsJobs := jobs[:1]

	fmt.Println(johnsJobs)
```

Now we know that slices refer to arrays and changing elements of a slice changes also changes elements of the array. There are two important concepts to know about slices: its length and capacity. A slice's length is the number of elements in the slice. Its capacity refers to the number of elements in the referring array. However, the capacity can only see the elements to the right of its index. 

```go
func main() {
	s := []int{2, 3, 5, 7, 11, 13}
	printSlice(s) // 6, 6, [2, 3, 5, 7, 11, 13]

	// Slice the slice to give it zero length.
	s = s[:0]
	printSlice(s) // 0, 6, []

	// Extend its length.
	s = s[:4]
	printSlice(s) // 4, 6, [2, 3, 5, 7]

	// Drop its first two values.
	s = s[2:]
	printSlice(s)  // 2, 4, [5, 7]
}

func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}

```

Another example:

```go
func printSlice(s []string) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}

func main() {
	days := []string{"Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"}

	days = days[0:1]
	printSlice(days) // 1, 7, [Monday]

	days = days[1:3]
	printSlice(days) // 2, 6, [Tuesday, Wednesday]

	days = days[1:2]
	printSlice(days) // 1, 5, [Wednesday]
}
```

Just remember, the length is the number of elements in the slice, while capacity refers to the number of elements in the original array based on the first element in our slice.

A *nil slice* is a slice whose length and capacity is 0. What's the use of a nil slice? Well, if you want to remove all the elements in a slice, you can set its value to nil. 

```go
func main() {

	type shoppingCart struct {
		name     string
		price    float64
		quantity int
	}

	s := []shoppingCart{
		{"brocolli", 2.14, 1},
		{"carrots", 1.25, 12},
		{"oranges", 0.35, 8},
	}

	fmt.Println(s)

	s = nil

	if s == nil {
		fmt.Println("Emptied the shopping cart!")
	}
}
```

A slice can be created using *make*. Why use make? The idea is that you want an array, but don't want to fill it up with data immediately. What you want to do is to use make to create our slice. The *make* keyword allocates memory for an array.

```go
func main() {
	a := make([]int, 5)
	a[0] = 1
	a[2] = 4
	a[4] = 6

	fmt.Println(a)
}
// [1 0 4 0 6]
```

We can create a two dimensional array by creating what's called slices of slices.

```go

func main() {
	board := [][]string{
		[]string{"_", "e", "_"},
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
		[]string{"_", "c", "_"},
	}

	for i := 0; i < len(board); i++ {
		fmt.Printf("%s\n", strings.Join(board[i], " "))
	}

}

```

Arrays and slices normally do not grow in size. We can, however, use the append() function to add more elements to a slice!

```go
func main() {
	lines := make([]string, 1)
	lines[0] = "Hello world"
	lines[1] = "I love Go!" // This results in a panic

	fmt.Println(lines)
}
```

Instead of using make, use append to dynamically grow a slice.

```go
func main() {
	lines := []string{}
	lines = append(lines, "Hello world")
	lines = append(lines, "I love Go!")
	fmt.Println(lines)
}
// [Hello world I love Go!]
```

Earlier we saw an example of using range over a slice of an slice of colors. We can also use range directly on a slice.

```go
func main() {
	colors := []string{
		"red",
		"blue",
		"yellow",
		"green",
		"orange"}

	for index, element := range colors {
		fmt.Println(index, element)
	}
}
```

If we only need the index, we can omit the second variable.

```go
func main() {
	numbers := []int{
		2,
		3,
		4,
		6,
		8}

	for index := range numbers {
		numbers[index] *= 2
		fmt.Println(numbers[index])
	}
}

```

### [Maps](#maps)

Maps are key value pairs. In Go, we use the *make* function to create a map. Below is an example where we do a few operations on maps: adding, finding the size, iterating, and deleting an element.

```go
func main() {
	// Define map
	emails := make(map[string]string)

	emails["Taro"] = "taro@gmail.com"
	emails["Goro"] = "goro@gmail.com"

	for _, v := range emails {
		fmt.Println(v)
	}

	fmt.Println("Total number of emails:", len(emails))

	delete(emails, "Taro")

	fmt.Println("Total number of emails:", len(emails))

	for _, v := range emails {
		fmt.Println(v)
	}
}

```

Below is a example where the value is data type struct. Note that we have made a map as a global variable and this requires the var keyword.

```go
type plant struct {
	age      int
	pType    string
	location string
}

var m map[string]plant

func main() {
	m = make(map[string]plant)
	m["oak"] = plant{50, "tree", "backyard"}
	m["sage"] = plant{2, "shrub", "sideyard"}

	fmt.Println(m["oak"])
	fmt.Println(m["sage"])
}

```

We can also create a map and define the elements at the same time in what is called a *map literal*:

```go
func main() {
	// Define map
	emails := map[string]string{"Taro": "taro@gmail.com", "Goro": "goro@gmail.com"}

	for _, v := range emails {
		fmt.Println(v)
	}
}
```

Another example, where the value is a struct.

```go
type plant struct {
	age      int
	pType    string
	location string
}

var m = map[string]plant{
	"oak":  plant{50, "tree", "backyard"},
	"sage": plant{2, "shrub", "sideyard"},
}

func main() {
	fmt.Println(m["oak"])
	fmt.Println(m["sage"])
}
```

Below is a solution for the wordcount problem on go.com. Here we make use of the Fields method which splits a string based on whitespace.

```go
package main

import (
	"golang.org/x/tour/wc"
	"strings"
)

type Frequency map[string]int

func WordCount(s string) map[string]int {

	f := make(Frequency)
	
	fields := strings.Fields(s)
	
	for _, word := range(fields) {
		f[word]++
	}
	
	return f
}

func main() {
	wc.Test(WordCount)
}

```

Below is a solution for a wordcount problem on exercism. We use regexp to split the string based on multiple delimitters.

```go
// Frequency of words
type Frequency map[string]int

// WordCount counts the number of words in phrase and returns a map
func WordCount(phrase string) Frequency {

	// Allocate storage for our map
	f := make(Frequency)

	// Create a regex expression defining our delimiters
	t := regexp.MustCompile(`[ ,\n:!&@$%^:.]`)

	// Return all substrings
	v := t.Split(phrase, -1)

	// Iterate through the array and populate the map
	for _, element := range v {

		// Ignore empty strings
		if len(element) == 0 {
			continue
		}

		element = strings.ToLower(element)
		element = strings.Trim(element, "'")

		// Update the map
		f[element]++

	}

	return f
}
```

Below is a solution to a wordcount problem in *The Go Programming Language*. The problem asks for finding the number of duplicates per file. The solution uses a map which embeds a second map as the values. It sounds cumbersone, but to make it easy for us, we create a map per file and assign it to our primary map.

```go
package main

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"log"
	"os"
	"strings"
)

func main() {
	// Map a file to a map of lines and the number of times it appears in that file.
	counts := make(map[string]map[string]int)

	for _, filename := range os.Args[1:] {
		data, err := ioutil.ReadFile(filename)

		if err != nil {
			fmt.Fprintf(os.Stderr, "dup3: %v\n", err)
			continue
		}

		// Create a map for our file
		fileMap := make(map[string]int)

		for _, line := range strings.Split(string(data), "\n") {
			// Discard whitespaces
			line = strings.TrimSpace(line)

			// Ignore whitespaces
			if len(line) == 0 {
				continue
			}

			// Increment the number of times the line appeared in the file
			fileMap[line]++
		}

		// Remove all entries in the map which are not duplicates
		for k, v := range fileMap {
			if v < 2 {
				delete(fileMap, k)
			}
		}

		// Assign the map to our map
		counts[filename] = fileMap
	}

	// Use the json package to convert the Go struct to JSON (marshalling)
	data, err := json.MarshalIndent(counts, "", "\t")

	if err != nil {
		log.Fatal("JSON marshalling failed: ", err)
	}

	fmt.Printf("%s\n", data)
}

/*
{
        "bar.txt": {
                "green": 3
        },
        "hello.txt": {
                "foo": 2
        }
}
*/
```

### [Functions](#functions)

Below is an example of a function which takes an argument and returns a string. Notice that the type comes after the name of the variable.

```go
func greet(name string) string {
	return "Hello " + name
}

func main() {
	fmt.Println(greet("Taro"))
}
```

If a function has multiple arguments of the same type, a shortcut can be used to declare the type for both of them.

```go
func add(num1, num2 int) int {
	return num1 + num2
}

func main() {
	fmt.Println(add(3, 4))
}
// 7
```

Functions can return multiple results. Note that the return types need to be specified within parenthesis

```go

func swap(x, y int64) (int64, int64) {
	return y, x
}

func main() {
	x := int64(23)
	y := int64(97)

	x, y = swap(x, y)

	fmt.Println("x and y: ", x, y)
}

```

Return values can be given variable names. This can aid in documentaing the function. A *naked* return can be used in this case but is not recommended for long functions.

```go
func getChange(quarters int64) (dimes int64, nickels int64, pennies int64) {

	// Set initial values
	dimes, nickels, pennies = 0, 0, 0

	// Get the number of dimes
	dimes = quarters / 10
	remainder := quarters % 10

	// Get the number of nickels. If we don't have a remainder, return from this function.

	if remainder > 0 {
		nickels = remainder / 5
		remainder = remainder % 5
	}

	if remainder > 0 {
		pennies = remainder
	}

	return
```

*Variadic functions* are functions that have a varying number of arguments. This is similar to an argument that is a list. 

```go
func sum(vals ...int) int {
	sum := 0

	for _, val := range vals {
		sum += val
	}
	return sum
}

func main() {
	sum := sum(1, 2, 3)
	fmt.Println(sum)
}
```

### [Methods](#methods)

Methods are functions that have been added to structs. The first of these are called *value receivers* because they don't change the original value.

```go
import (
	"fmt"
	"strconv"
)

type name struct {
	firstName string
	lastName  string
	age       int
}

func (n name) greet() string {
	return "Hi, my name is " + n.firstName + " " + n.lastName + " and I am " + strconv.Itoa(n.age)
}

func main() {
	p := name{"Mary", "Pierce", 26}
	fmt.Println(p.greet())
}
```

If we want to modify the content of a struct, we must use a *pointer receiver*. What happens in this case is that we pass the address of the variable to the function and not simply its values. 

```go
func (n *name) birthday() {
	n.age++
}

func main() {
	p := name{"Mary", "Pierce", 26}
	fmt.Println(p.greet())
	p.birthday()
	fmt.Println(p.greet())
}
```

Below is an example of using pointer receivers in functions to change the value of the arguments.

```go
type elf64Half int64
type elf64Word int64
type elf64Off int64

// A structure to describe an elf32 header
type elf32Hddr struct {
	eIdent   []rune
	eType    elf64Half
	eMachine elf64Half
	eEntry   elf64Word
	ePhoff   elf64Off
}

// A function to swap the type of two elf32 headers
func swap(x *elf32Hddr, y *elf32Hddr) {
	x.eType, y.eType = y.eType, x.eType
}

func main() {
	x := elf32Hddr{[]rune("abcdefg"), 81, 44, 9, 0}
	y := elf32Hddr{[]rune("abcdefg"), 18, 44, 9, 0}

	fmt.Println("Before swap: ", x.eType, y.eType)

	swap(&x, &y)

	fmt.Println("After swap: ", x.eType, y.eType)
}
```

A type is said to implment an interface if it implements its methods. Below, we implement the greet() method on the person interface.

```go
import (
	"fmt"
	"math"
)

import (
	"fmt"
	"math"
)

type shape interface {
	area() float64
}

type circle struct {
	radius float64
}

type rectangle struct {
	x, y float64
}

func (c circle) area() float64 {
	return math.Pi * c.radius * c.radius
}

func (s rectangle) area() float64 {
	return s.x * s.y
}

func getArea(s shape) float64 {
	return s.area()
}

func main() {
	c := circle{7.0}
	fmt.Printf("Area: %f\n", math.Round(getArea(c)))

	s := rectangle{9.0, 3.0}
	fmt.Printf("Area: %f\n", getArea(s))
}


// 154
// 27
```

An "stringer" can be implemented for interfaces to provide a human-friendly string for printing to the console.


```go
type iPAddr [4]byte

func (p iPAddr) String() string {
	return fmt.Sprintf("%d.%d.%d.%d", p[0], p[1], p[2], p[3])
}

func main() {
	hosts := map[string]iPAddr{
		"loopback":  {127, 0, 0, 1},
		"googleDNS": {8, 8, 8, 8},
	}
	for name, ip := range hosts {
		fmt.Printf("%v: %v\n", name, ip)
	}
}
```

### [Files](#files)

To read a file, use *ioutil.readFile()*.

```go
package main

import (
	"fmt"
	"io/ioutil"
	"os"
	"strings"
)

func main() {
	data, err := ioutil.ReadFile("hello.txt")

	if err != nil {
		fmt.Fprintf(os.Stderr, "%v\n", err)
		return
	}

	// Because data is a byte slice, attempting to just print it will not result in
	// human-readable values.
	// fmt.Println(data)

	// The *split*() method can parse byte arrays
	for _, line := range strings.Split(string(data), "\n") {
		fmt.Println(line)
	}
}
/*
{
  "name": "my_package",
  "version": "1.0.0",
  "dependencies": {
    "my_dep": "^1.0.0",
    "another_dep": "~2.2.0"
  }
}
*/
```