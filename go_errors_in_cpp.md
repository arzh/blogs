h1. Go like errors in C++

I'm starting this year off with working on a game that I have been thinking about all last year. I'm much more of a "tech" guy then a "game" guy so I plan on writting it all from scratch. I plan on using OpenGL as the render, which is the first thing I plan on writting. OpenGL is a C libary so its error handling is a bit interesting. Since I plan on writting all the API layer on top of OpenGL I wanted to have a common error handling system. Most C++ libaries that I have seen use exceptions for there errors. I hate exceptions. I hate the mechanism of blowing up the stack when an exception is thrown. I hate the code that you have to write to handle the expections. I hate that even small stuff can crash your application if you don't handle an exception. So for me exceptions are right out! I have enjoyed the concept of the [error type in go|http://blog.golang.org/error-handling-and-go] so I started to think about how I can use that concept in C++. 

The core concets of error that I wanted to maintain are; 
* An interface to get the error message
* The ability for the user to extend the error type while still using the same signature for the return value
* The end user of the error should not have to manage the memory of that error
* A simple way to check the error without having to use a function call
* Common way to check the type of an error and to get the error in that type.

To help follow the rest of the post I want to show you an example of the code I would like to write with this error (Very TDD of me):
```c++

class readFailure : public error {
public:
	const std::string file;

	readFailure(const std::string& _filename) :
		file(_filename) {}

	const std::string Error() const {
		return "Failed to read from file: " + file;
	}
};

error ReadFileToStr(const char* filename, std::string& contents) {
	// opening code
	if (!file.is_open()) {
		return error("Could not open file: " + filename); // std error
	}

	// reading code
	if (file.failed_to_read()) { // This isn't a real api
		return readFailure(filename); // custom error
	}

	return nil; // Success; no error
}

int main(int argc, char** argv) {

	std::string file;
	error err = ReadFileToStr("file.txt", file);
	
	if (err != nil) {
		if (err.IsType(readFailure)) {
			std::cerr << err;
			return 1;
		}
		std::cout << err;
	}

}
```

h3. Interface
This one is really easy to do:
```c++
class error {
public:
	virtual const std::string Error() const = 0;
};
```

h3. User errors
Not all errors are built the same. Some errors aren't show stoppers, like EOF errors. Others are not much more that an indicature of failure and a message. Giving the user the ability to create their own errors give them the power to design the API how they want to.

Because right now our `error` is an abstract class it is easy to implement a custom version.
```c++
class readFailure : public error {
public:
	const std::string file;

	readFailure(const std::string& _filename) :
		file(_filename) {}

	const std::string Error() const {
		return "Failed to read from file: " + file;
	}
};
```
In c++ you can't return instances of abstract classes, you can only return pointers to them. It requires usage code to be written like this:
```c++
error* ReadFile() {
	// important code
	if (file.failed_to_read()) {
		readFailure* e = new readFailure(filename);
		return e;
	}

	return nil;
}
```
I don't like this one bit. The user has to manage the memeory or the error, which is something I don't want. You also have to refrence the return as a pointer, and I don't like that.




