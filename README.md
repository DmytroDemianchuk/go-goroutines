# Go Goroutines

Example that shows principles of goroutines in golang

```
package main

import "fmt"

func main() {
	for i := 0; i < 5; i++ {
		go func() {
			fmt.Println(i)
		}()
	}
}

```

People often think that we have a loop executing, and in a separate goroutine, we run a function that will print numbers from zero to four. But here we make a mistake because we are not running the function, we are just putting it in the scheduler queue. When the queue gets to it - only the scheduler knows, we cannot influence this. So, five iterations of the loop will pass faster than the scheduler can launch one of the goroutines. And even if one goroutine starts, the other four won't have time, so the loop will end and after that, we will have nothing. Since main is the entry point of the program, when it ends, the whole process ends. If the process ends, all its child elements (goroutines) also end. So our program will finish, and the other goroutines won't print anything.


```

func main() {
	for i := 0; i < 5; i++ {
		go func() {
			fmt.Println(i)
		}()
	}

	time.Sleep(time.Second)
}

func main() {
	for i := 0; i < 5; i++ {
		go func() {
			fmt.Println(i)
		}()
	}

	time.Sleep(time.Minute)
}

```

We can use time.Sleep, but in some cases, it may be insufficient or excessive. To make the program wait until all goroutines finish, we use WaitGroup. WaitGroup has three basic methods:

- Add - increases the counter by a specified number.
- Done - decreases the counter by one.
- Wait - blocks until the counter becomes zero.

```
func main() {
	wg: sync.WaitGroup{}

	for i := 0; i < 5; i++ {

		go func() {
			wg.Add(1)

			fmt.Println(i)
			wg.Done()
		}()
	}

	wg.Wait()
}
```

It may seem that we haven't started WaitGroup here and increased the counter since it started, finished the goroutine, wrote Done. The reason is that we don't know when the anonymous functions in the goroutines will start. The loop will pass faster than any goroutine can execute. The counter will be zero because no one has increased it, and Wait will not block, and everything will finish

```
func main() {
	wg: sync.WaitGroup{}
	wg.Add(1)

	for i := 0; i < 5; i++ {
		go func() {
			fmt.Println(i)
			wg.Done()
		}()
	}

	wg.Wait()
}
```

We can imagine a situation where we know how many iterations there will be (in this case, 5), and in that case, the code works. But this is only for our case, often the code logic can be more complex with different rules, exit points, etc

```
func main() {
	wg: sync.WaitGroup{}
	wg.Add(1)

	for i := 0; i < 5; i++ {
		go func() {
			fmt.Println(i)
			if {
				return 
			}

			if {
				return
			}

			wg.Done()
		}()
	}

	wg.Wait()
}
```

For example, like here. If we leave it like this, we will encounter a deadlock, because we exit here and just don't reach Done. It turns out that the goroutine finishes, but the counter doesn't decrease by one, and we get a block. The runtime will say that the counter is not zero, but all the goroutines have finished, and panic will occur and there will be a deadlock

```

func main() {
	wg: sync.WaitGroup{}

	wg.Add(5)
	for i := 0; i < 5; i++ {
		go func() {
			defer wg.Done()
			fmt.Println(i)
		}()
	}

	wg.Wait()
}
```


defer comes to our rescue. The defer statement is executed at the end of the function, regardless of whether the function finished with panic or something else. defer will be called. But there is a case where defer will not run. For example, if we call os.Exit(1) or log.Fatal()

```
func main() {
	wg: sync.WaitGroup{}

	wg.Add(5)
	for i := 0; i < 5; i++ {
		go func() {
			defer wg.Done()
			fmt.Println(i)

			os.Exit(1)
			Log.Fatal()
		}()
	}

	wg.Wait()
}
```

For example, when developing console utilities, we can wrap metrics collection, logging, etc. in defer. And in general, if we collect statistics, in such cases, defer will not be executed


```
func main() {
	wg: sync.WaitGroup{}

	wg.Add(5)
	for i := 0; i < 5; i++ {
		go func() {
			fmt.Println(i)
			defer wg.Done()

		}()
	}

	wg.Wait()
}
```
In this case, defer will be executed only if it is on the stack.
What is a stack ?

```

func main() {
	wg: sync.WaitGroup{}

	wg.Add(5)
	for i := 0; i < 5; i++ {
		go func() {
			defer.fmt.Println(i) --- другий 
			defer wg.Done() ---- перший 

		}()
	}

	wg.Wait()
}

```

We can do this, and then we will have defer executed in order. But this looks strange and unreasonable

```
func main() {
	wg: sync.WaitGroup{}

	wg.Add(5)
	for i := 0; i < 5; i++ {
		go func(wg *sync.WaitGroup) {
			defer wg.Done() 
			fmt.Println(i) 

		}(&wg)
	}

	wg.Wait()
}

func warszaw(wg *sync.WaitGroup) {

}
```

We can also pass arguments, but the thing is, we are dealing with closures. Imagine that we don't have an anonymous function but an external function where the logic is executed, then yes, we will need to pass WaitGroup. But WaitGroup must be passed by pointer. If we did not pass by pointer, it would just copy, and the counter hidden under it would also be copied. And all our modifications, calls to Done, etc. would apply to the copy, and the original on which we are waiting Wait would not be aware of these changes

```
func main() {
	wg: sync.WaitGroup{} --- зовнішя чатсина фун3цї

	wg.Add(5)
	for i := 0; i < 5; i++ {
		go func(wg *sync.WaitGroup) { --- в середині однієї функції
			defer wg.Done()  ---- перемінна
			fmt.Println(i) 

		}()
	}

	wg.Wait()
}
```



Go Goroutines
Example
go
Копіювати код
package main

import "fmt"

func main() {
    for i := 0; i < 5; i++ {
        go func() {
            fmt.Println(i)
        }()
    }
}
People often think that we have a loop executing, and in a separate goroutine, we run a function that will print numbers from zero to four. But here we make a mistake because we are not running the function, we are just putting it in the scheduler queue. When the queue gets to it - only the scheduler knows, we cannot influence this. So, five iterations of the loop will pass faster than the scheduler can launch one of the goroutines. And even if one goroutine starts, the other four won't have time, so the loop will end and after that, we will have nothing. Since main is the entry point of the program, when it ends, the whole process ends. If the process ends, all its child elements (goroutines) also end. So our program will finish, and the other goroutines won't print anything.
go
Копіювати код
func main() {
    for i := 0; i < 5; i++ {
        go func() {
            fmt.Println(i)
        }()
    }

    time.Sleep(time.Second)
}
We can use time.Sleep, but in some cases, it may be insufficient or excessive. To make the program wait until all goroutines finish, we use WaitGroup. WaitGroup has three basic methods:
Add - increases the counter by a specified number.
Done - decreases the counter by one.
Wait - blocks until the counter becomes zero.
go
Копіювати код
func main() {
    var wg sync.WaitGroup

    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func() {
            fmt.Println(i)
            wg.Done()
        }()
    }

    wg.Wait()
}
It may seem that we haven't started WaitGroup here and increased the counter since it started, finished the goroutine, wrote Done. The reason is that we don't know when the anonymous functions in the goroutines will start. The loop will pass faster than any goroutine can execute. The counter will be zero because no one has increased it, and Wait will not block, and everything will finish.
go
Копіювати код
func main() {
    var wg sync.WaitGroup

    wg.Add(1)

    for i := 0; i < 5; i++ {
        go func() {
            fmt.Println(i)
            wg.Done()
        }()
    }

    wg.Wait()
}
We can imagine a situation where we know how many iterations there will be (in this case, 5), and in that case, the code works. But this is only for our case, often the code logic can be more complex with different rules, exit points, etc.
go
Копіювати код
func main() {
    var wg sync.WaitGroup

    wg.Add(5)

    for i := 0; i < 5; i++ {
        go func() {
            fmt.Println(i)
            if true {
                return // here
            }

            if true {
                return
            }

            wg.Done()
        }()
    }

    wg.Wait()
}
For example, like here. If we leave it like this, we will encounter a deadlock, because we exit here and just don't reach Done. It turns out that the goroutine finishes, but the counter doesn't decrease by one, and we get a block. The runtime will say that the counter is not zero, but all the goroutines have finished, and panic will occur and there will be a deadlock.
go
Копіювати код
func main() {
    var wg sync.WaitGroup

    wg.Add(5)

    for i := 0; i < 5; i++ {
        go func() {
            defer wg.Done()
            fmt.Println(i)
        }()
    }

    wg.Wait()
}
defer comes to our rescue. The defer statement is executed at the end of the function, regardless of whether the function finished with panic or something else. defer will be called. But there is a case where defer will not run. For example, if we call os.Exit(1) or log.Fatal().
go
Копіювати код
func main() {
    var wg sync.WaitGroup

    wg.Add(5)

    for i := 0; i < 5; i++ {
        go func() {
            fmt.Println(i)
            os.Exit(1)
            log.Fatal()
            defer wg.Done()
        }()
    }

    wg.Wait()
}
For example, when developing console utilities, we can wrap metrics collection, logging, etc. in defer. And in general, if we collect statistics, in such cases, defer will not be executed.
go
Копіювати код
func main() {
    var wg sync.WaitGroup

    wg.Add(5)

    for i := 0; i < 5; i++ {
        go func() {
            fmt.Println(i)
            defer wg.Done()
        }()
    }

    wg.Wait()
}
In this case, defer will be executed only if it is on the stack.
What is a stack?
go
Копіювати код
func main() {
    var wg sync.WaitGroup

    wg.Add(5)

    for i := 0; i < 5; i++ {
        go func() {
            defer fmt.Println(i) // second
            defer wg.Done() // first
        }()
    }

    wg.Wait()
}
We can do this, and then we will have defer executed in order. But this looks strange and unreasonable.
go
Копіювати код
func main() {
    var wg sync.WaitGroup

    wg.Add(5)

    for i := 0; i < 5; i++ {
        go func(wg *sync.WaitGroup) {
            defer wg.Done()
            fmt.Println(i)
        }(&wg)
    }

    wg.Wait()
}
We can also pass arguments, but the thing is, we are dealing with closures. Imagine that we don't have an anonymous function but an external function where the logic is executed, then yes, we will need to pass WaitGroup. But WaitGroup must be passed by pointer. If we did not pass by pointer, it would just copy, and the counter hidden under it would also be copied. And all our modifications, calls to Done, etc. would apply to the copy, and the original on which we are waiting Wait would not be aware of these changes.
go
Копіювати код
func main() {
    var wg sync.WaitGroup

    wg.Add(5)

    for i := 0; i < 5; i++ {
        go func(wg *sync.WaitGroup) {
            defer wg.Done()
            fmt.Println(i)
        }(&wg)
    }

    wg.Wait()
}
But since our function is called inside another function, we don't need to pass all this because we have a closure. A closure is when a variable declared outside the function is used inside the function. And this variable defer wg.Done() by language standards is captured inside by pointer, which, in turn, does not create any copy, and this suits us very well. But in the case of fmt.Println(i) it does not suit us. i is a loop counter that changes in the queue of all iterations. This is the same variable that has the same address. Since we are dealing with a closure here, we have the value i passed by pointer in the pipeline. But there's a nuance - the goroutines will start after all iterations, and the value at the address i is the last one. And then most goroutines will print the last value of the counter. The last value of the loop in this case will not always be 4. When the counter reaches 4, it increments the iteration, then it increments, becomes 5, and only after it becomes 5 do we understand that the loop condition is not met i < 5, and we exit the loop. But i remains 5. So most goroutines will print 5. Of course, some goroutine will print 3 or something else because it can be faster than the loop iteration ends. We have two ways to solve this problem: make a copy of the variable.


```
func main() {
    var wg sync.WaitGroup

    wg.Add(5)

    for i := 0; i < 5; i++ {
        go func(i int) {
            defer wg.Done()
            fmt.Println(i)
        }(i)
    }

    wg.Wait()
}

```
We accept i into the anonymous function and print it. Or create a local variable with the same name and copy this value.


```
func main() {
    var wg sync.WaitGroup

    wg.Add(5)

    for i := 0; i < 5; i++ {
        i := i
        go func() {
            defer wg.Done()
            fmt.Println(i)
        }()
    }

    wg.Wait()
}

```


Inside the loop iteration, create a local variable with the same name and copy the value, and refer to this local variable

----
```
func main() {
    var wg sync.WaitGroup

    wg.Add(5)

    for i := 0; i < 5; i++ {
        go func(i int) {
            defer wg.Done()
            fmt.Println(i)
        }(i)
    }

    wg.Wait()
}
```

Here is the working version that will print numbers from 0 to 4 in the correct order. In this case, there will be no overhead on waiting, Wait will wait until all goroutines complete their work

	