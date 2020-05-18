---
title: Spring REST Error Handling Example
time: 2019-11-09
categories: spring
tags: spring
---

# Spring REST Error Handling Example
---
In this article, we will show you error handling in Spring Boot REST application.

## 1. /error
1.1 By default, Spring Boot provides a BasicErrorController controller for /error mapping that handles all errors, and getErrorAttributes to produce a JSON response with details of the error, the HTTP status, and the exception message.

```
{	
	"timestamp":"2019-02-27T04:03:52.398+0000",
	"status":500,
	"error":"Internal Server Error",
	"message":"...",
	"path":"/path"
}
```

BasicErrorController.java
```
package org.springframework.boot.autoconfigure.web.servlet.error;

//...

@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {

	//...

	@RequestMapping
	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
		Map<String, Object> body = getErrorAttributes(request,
				isIncludeStackTrace(request, MediaType.ALL));
		HttpStatus status = getStatus(request);
		return new ResponseEntity<>(body, status);
	}
```
In the IDE, puts a breakpoint in this method, you will understand how Spring Boot generates the default JSON error response.

## 2. Custom Exception
In Spring Boot, we can use @ControllerAdvice to handle custom exceptions.

2.1 A custom exception.

BookNotFoundException.java
```
package com.mkyong.error;

public class BookNotFoundException extends RuntimeException {

    public BookNotFoundException(Long id) {
        super("Book id not found : " + id);
    }

}
```

A controller, if a book id is not found, throws the above BookNotFoundException

BookController.java
```
package com.mkyong;

//...

@RestController
public class BookController {

    @Autowired
    private BookRepository repository;
	
    // Find
    @GetMapping("/books/{id}")
    Book findOne(@PathVariable Long id) {
        return repository.findById(id)
                .orElseThrow(() -> new BookNotFoundException(id));
    }

	//...
}
```

By default, Spring Boot generates the following JSON error response, http 500 error.
```
curl localhost:8080/books/5

{
	"timestamp":"2019-02-27T04:03:52.398+0000",
	"status":500,
	"error":"Internal Server Error",
	"message":"Book id not found : 5",
	"path":"/books/5"
}
```

 2.2 If a book id not found, it should return a 404 error instead of 500, we can override the status code like this :

 CustomGlobalExceptionHandler.java
```
package com.mkyong.error;

import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.context.request.WebRequest;
import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;

import javax.servlet.http.HttpServletResponse;
import javax.validation.ConstraintViolationException;
import java.io.IOException;
import java.util.Date;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;


@ControllerAdvice
public class CustomGlobalExceptionHandler extends ResponseEntityExceptionHandler {

    // Let Spring BasicErrorController handle the exception, we just override the status code
    @ExceptionHandler(BookNotFoundException.class)
    public void springHandleNotFound(HttpServletResponse response) throws IOException {
        response.sendError(HttpStatus.NOT_FOUND.value());
    }

	//...
}
```

2.3 It returns a 404 now.
```
curl localhost:8080/books/5

{
	"timestamp":"2019-02-27T04:21:17.740+0000",
	"status":404,
	"error":"Not Found",
	"message":"Book id not found : 5",
	"path":"/books/5"
}
```


2.4 Furthermore, we can customize the entire JSON error response :

CustomErrorResponse.java
```
package com.mkyong.error;

import com.fasterxml.jackson.annotation.JsonFormat;

import java.time.LocalDateTime;

public class CustomErrorResponse {

    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd hh:mm:ss")
    private LocalDateTime timestamp;
    private int status;
    private String error;

    //...getters setters
}
```

```
package com.example.demo.exceptionHandler;

import com.example.demo.errorresponse.CustomErrorResponse;
import com.example.demo.exception.BookNotFoundException;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.context.request.WebRequest;
import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;

import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.time.LocalDateTime;

/**
 * @author: Lancer
 * @date: 2019/11/9
 */
@ControllerAdvice
public class CustomGlobalExceptionHandler extends ResponseEntityExceptionHandler {

    // Let Spring BasicErrorController handle the exception, we just override the status code
//    @ExceptionHandler(BookNotFoundException.class)
    public void springHandleNotFound(HttpServletResponse response) throws IOException {
        System.out.println("hello");
        response.sendError(HttpStatus.NOT_FOUND.value());

    }

    //...
    @ExceptionHandler(BookNotFoundException.class)
    public ResponseEntity<CustomErrorResponse> customHandleNotFound(Exception ex, WebRequest request) {

        CustomErrorResponse errors = new CustomErrorResponse();

        errors.setTimestamp(LocalDateTime.now());
        errors.setError(ex.getMessage());
        errors.setStatus(HttpStatus.NOT_FOUND.value());

        return new ResponseEntity<>(errors, HttpStatus.NOT_FOUND);
    }
}
```

terminal
```
curl localhost:8080/books/5
{
	"timestamp":"2019-02-27 12:40:45",
	"status":404,
	"error":"Book id not found : 5"
}
```

## 3. JSR 303 Validation error

3.1 For Spring @valid validation errors, it will throw handleMethodArgumentNotValid

CustomGlobalExceptionHandler.java
```
package com.mkyong.error;

import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.context.request.WebRequest;
import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;

import javax.servlet.http.HttpServletResponse;
import javax.validation.ConstraintViolationException;
import java.io.IOException;
import java.util.Date;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

@ControllerAdvice
public class CustomGlobalExceptionHandler extends ResponseEntityExceptionHandler {

    //...

    // @Validate For Validating Path Variables and Request Parameters
    @ExceptionHandler(ConstraintViolationException.class)
    public void constraintViolationException(HttpServletResponse response) throws IOException {
        response.sendError(HttpStatus.BAD_REQUEST.value());
    }

    // error handle for @Valid
    @Override
    protected ResponseEntity<Object>
    handleMethodArgumentNotValid(MethodArgumentNotValidException ex,
                                 HttpHeaders headers,
                                 HttpStatus status, WebRequest request) {

        Map<String, Object> body = new LinkedHashMap<>();
        body.put("timestamp", new Date());
        body.put("status", status.value());

        //Get all fields errors
        List<String> errors = ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(x -> x.getDefaultMessage())
                .collect(Collectors.toList());

        body.put("errors", errors);

        return new ResponseEntity<>(body, headers, status);

    }

}

```


## 4. ResponseEntityExceptionHandler
4.1 If we are not sure, what exception was thrown by the Spring Boot, puts a breakpoint in this method for debugging.

ResponseEntityExceptionHandler.java
```
package org.springframework.web.servlet.mvc.method.annotation;

//...
public abstract class ResponseEntityExceptionHandler {

	@ExceptionHandler({
			HttpRequestMethodNotSupportedException.class,
			HttpMediaTypeNotSupportedException.class,
			HttpMediaTypeNotAcceptableException.class,
			MissingPathVariableException.class,
			MissingServletRequestParameterException.class,
			ServletRequestBindingException.class,
			ConversionNotSupportedException.class,
			TypeMismatchException.class,
			HttpMessageNotReadableException.class,
			HttpMessageNotWritableException.class,
			MethodArgumentNotValidException.class,
			MissingServletRequestPartException.class,
			BindException.class,
			NoHandlerFoundException.class,
			AsyncRequestTimeoutException.class
		})
	@Nullable
	public final ResponseEntity<Object> handleException(Exception ex, WebRequest request) throws Exception {
		HttpHeaders headers = new HttpHeaders();

		if (ex instanceof HttpRequestMethodNotSupportedException) {
			HttpStatus status = HttpStatus.METHOD_NOT_ALLOWED;
			return handleHttpRequestMethodNotSupported((HttpRequestMethodNotSupportedException) ex, headers, status, request);
		}
		else if (ex instanceof HttpMediaTypeNotSupportedException) {
			HttpStatus status = HttpStatus.UNSUPPORTED_MEDIA_TYPE;
			return handleHttpMediaTypeNotSupported((HttpMediaTypeNotSupportedException) ex, headers, status, request);
		}
		//...
	}

	//...

}

```

## 5. DefaultErrorAttributes
5.1 To override the default JSON error response for all exceptions, create a bean and extends DefaultErrorAttributes

CustomErrorAttributes.java
```
package com.mkyong.error;

import org.springframework.boot.web.servlet.error.DefaultErrorAttributes;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.WebRequest;

import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Map;

@Component
public class CustomErrorAttributes extends DefaultErrorAttributes {

    private static final DateFormat dateFormat = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss");

    @Override
    public Map<String, Object> getErrorAttributes(WebRequest webRequest, boolean includeStackTrace) {

        // Let Spring handle the error first, we will modify later :)
        Map<String, Object> errorAttributes = super.getErrorAttributes(webRequest, includeStackTrace);

        // format & update timestamp
        Object timestamp = errorAttributes.get("timestamp");
        if (timestamp == null) {
            errorAttributes.put("timestamp", dateFormat.format(new Date()));
        } else {
            errorAttributes.put("timestamp", dateFormat.format((Date) timestamp));
        }

        // insert a new key
        errorAttributes.put("version", "1.2");

        return errorAttributes;

    }

}
```

Now, the date time is formatted and a new field – version is added to the JSON error response.

```
curl localhost:8080/books/5

{
	"timestamp":"2019/02/27 13:34:24",
	"status":404,
	"error":"Not Found",
	"message":"Book id not found : 5",
	"path":"/books/5",
	"version":"1.2"
}

curl localhost:8080/abc

{
	"timestamp":"2019/02/27 13:35:10",
	"status":404,
	"error":"Not Found",
	"message":"No message available",
	"path":"/abc",
	"version":"1.2"
}
```

