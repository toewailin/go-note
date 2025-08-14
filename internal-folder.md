Sure! Let’s go through an example where we have two projects:

1. Current Project: myapp (the Go project where you’re managing your code).


2. External Project: externalapp (another Go project that will use public APIs from myapp).



Folder Structure for myapp (Current Project):

In this structure, we will place some parts of the code under the internal directory to ensure they are not accessible outside the myapp project.

myapp/
│
├── cmd/
│   └── main.go
├── internal/
│   ├── db/
│   │   └── connection.go  // Private logic for database connection
│   ├── utils/
│   │   └── helpers.go     // Private utility functions
│   └── auth/
│       └── auth.go        // Private logic for user authentication
├── pkg/
│   └── api/
│       └── api.go         // Public API logic, exposed to external projects
└── go.mod

Explanation:

cmd/main.go: This file contains the entry point for the myapp application. It can access both internal and public packages.

internal/db/connection.go: This package contains code related to connecting to a database, and it’s only accessible within myapp.

internal/utils/helpers.go: Contains utility functions like logging, string manipulation, etc., but only accessible within myapp.

internal/auth/auth.go: Authentication-related logic (such as user login, password validation), which is private to myapp.

pkg/api/api.go: This is a public package, which can be accessed by external applications. This might expose API routes or handlers that external systems can call.


Folder Structure for externalapp (External Project):

The external project (externalapp) wants to interact with the myapp project. However, it cannot directly access any internal package from myapp (e.g., internal/db, internal/auth, or internal/utils), but it can use the public API exposed in pkg/api.

externalapp/
│
├── cmd/
│   └── main.go
└── go.mod

cmd/main.go: This is the entry point for externalapp, where it will interact with the exposed public packages from myapp.


How externalapp Can Interact with myapp:

The external project can only import the public package from myapp. Specifically, it can import myapp/pkg/api and use the functionality exposed in that package.

Code Example:

1. myapp/pkg/api/api.go (Public API exposed for externalapp):

package api

import (
    "fmt"
    "myapp/internal/db"  // Accesses internal DB logic
)

func GetDatabaseStatus() string {
    // Uses internal db package to get connection status
    return db.Connect()
}

Here, pkg/api exposes the GetDatabaseStatus function which internally uses the internal/db package.

Note: The internal/db package is not accessible directly by external projects like externalapp.


2. myapp/internal/db/connection.go (Private DB connection logic):

package db

func Connect() string {
    // Simulated DB connection logic
    return "Connected to the database successfully!"
}

The Connect function is private to myapp and is not available for use outside the project.


3. externalapp/cmd/main.go (Using myapp/pkg/api):

package main

import (
    "fmt"
    "myapp/pkg/api"  // External project imports public API package
)

func main() {
    // The external app uses the public API exposed by `myapp`
    status := api.GetDatabaseStatus()
    fmt.Println("Database Status:", status)
}

In this file, externalapp imports myapp/pkg/api and calls the GetDatabaseStatus function.

The externalapp cannot access myapp/internal/db or myapp/internal/auth because those are internal to myapp.


What Happens When External Project Tries to Access Internal Package:

If externalapp tried to import anything from the internal folder (e.g., myapp/internal/db), it would result in a compile-time error.

For example, if externalapp/cmd/main.go tries to do this:

package main

import (
    "fmt"
    "myapp/internal/db"  // ERROR: Cannot import internal package
)

func main() {
    status := db.Connect()  // This line will cause a compile error
    fmt.Println("Database Status:", status)
}

This will give an error like:

cannot import package myapp/internal/db (package is internal to myapp)

Summary of Access Control:

1. Inside myapp:

cmd can access both public (pkg/api) and private (internal/db, internal/auth, etc.) packages.

pkg can only access public APIs and expose functionality to external applications.

internal is private to myapp and cannot be accessed from outside.



2. In externalapp:

externalapp/cmd can only import myapp/pkg/api, the public part of the myapp project.

It cannot access any internal packages like myapp/internal/db, myapp/internal/utils, or myapp/internal/auth.




This architecture keeps your internal logic private, ensuring that external projects only interact with the parts of your application that are explicitly meant to be public, thus improving maintainability, security, and encapsulation.

