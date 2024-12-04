
### Key Changes:
1. **Reading the `.env` file**: You are using `fmt.Fscan` to read the content of the file. This is not the best approach, especially when dealing with large files or needing precise control over the content. Instead, I recommend using `os.ReadFile` to read the file content.
2. **Appended data**: When appending new data (`JWT_SECRET_KEY`), you should ensure no redundant newline characters are added unnecessarily.
3. **Appending after checking**: Before appending `JWT_SECRET_KEY`, we will check if it already exists, and only append it if it doesn't.

### Updated Code:

```go
package main

import (
	"crypto/rand"
	"encoding/base64"
	"fmt"
	"log"
	"os"
	"strings"
)

func main() {
	// Generate 32 random bytes for the secret key
	bytes := make([]byte, 32)
	_, err := rand.Read(bytes)
	if err != nil {
		log.Fatal(err)
	}

	// Encode the bytes to base64 to get a string
	secretKey := base64.StdEncoding.EncodeToString(bytes)
	fmt.Println("Generated Secret Key: ", secretKey)

	// Define the path to the .env file
	envFilePath := "config/.env"

	// Read the content of the .env file
	fileContent, err := os.ReadFile(envFilePath)
	if err != nil {
		log.Fatal("Error reading .env file: ", err)
	}

	// Check if JWT_SECRET_KEY is already in the file
	if strings.Contains(string(fileContent), "JWT_SECRET_KEY") {
		fmt.Println("JWT_SECRET_KEY already exists in .env file. No changes made.")
		return
	}

	// Open the .env file for reading and appending
	file, err := os.OpenFile(envFilePath, os.O_RDWR|os.O_APPEND, 0644)
	if err != nil {
		log.Fatal("Error opening .env file for appending: ", err)
	}
	defer file.Close()

	// Check if there's already a newline at the end of the file, add one if necessary
	if !strings.HasSuffix(string(fileContent), "\n") {
		_, err = file.WriteString("\n")
		if err != nil {
			log.Fatal("Error adding newline to .env file: ", err)
		}
	}

	// Append the JWT_SECRET_KEY to the .env file
	_, err = file.WriteString("JWT_SECRET_KEY=" + secretKey + "\n")
	if err != nil {
		log.Fatal("Error writing JWT_SECRET_KEY to .env file: ", err)
	}

	// Successfully added the JWT_SECRET_KEY to .env
	fmt.Println("JWT_SECRET_KEY has been saved to .env file.")
}
```

### Explanation of Changes:
1. **Reading the `.env` file**: Using `os.ReadFile` ensures that we can easily work with the entire content of the `.env` file as a string, which allows us to search for the `JWT_SECRET_KEY` easily using `strings.Contains`.
2. **Check for `JWT_SECRET_KEY`**: We search for `JWT_SECRET_KEY` in the file content before appending it. This avoids adding it multiple times.
3. **File Opening**: I moved the file opening logic after checking if the key already exists in the `.env` file, ensuring we only open the file when necessary.
4. **Newline Handling**: Before appending, we check if there's a newline at the end of the file. If there isn't one, we explicitly add it to avoid accidental overwriting or malformed `.env` file entries.
5. **Appending the Key**: Finally, the key is appended at the end, followed by a newline to ensure proper formatting.

### Result in `.env` file:

```
DB_PORT=3306
DB_NAME=db_aioceaneye_api_go
JWT_SECRET_KEY=WQwR9xn8LRycqVBWewctrktKOMqCVL8R/8QCbGK2a44=
```

This ensures that `JWT_SECRET_KEY` is correctly appended without creating unnecessary issues or formatting problems in your `.env` file.
