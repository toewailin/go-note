### Updated Code (`cmd/generate_rsa/main.go`)

```go
package main

import (
	"crypto/rand"
	"crypto/rsa"
	"crypto/x509"
	"encoding/pem"
	"fmt"
	"log"
	"os"
)

// Function to generate RSA private and public keys
func generateRSAKeys(bits int) (*rsa.PrivateKey, *rsa.PublicKey, error) {
	// Generate the private key
	privateKey, err := rsa.GenerateKey(rand.Reader, bits)
	if err != nil {
		return nil, nil, err
	}

	// Extract the public key from the private key
	publicKey := &privateKey.PublicKey

	return privateKey, publicKey, nil
}

// Function to save the private key to a PEM file
func savePrivateKeyToFile(privateKey *rsa.PrivateKey, filename string) error {
	// Create a PEM block with the private key
	privFile, err := os.Create(filename)
	if err != nil {
		return err
	}
	defer privFile.Close()

	privBytes := x509.MarshalPKCS1PrivateKey(privateKey)

	err = pem.Encode(privFile, &pem.Block{
		Type:  "RSA PRIVATE KEY",
		Bytes: privBytes,
	})
	return err
}

// Function to save the public key to a PEM file
func savePublicKeyToFile(publicKey *rsa.PublicKey, filename string) error {
	// Create a PEM block with the public key
	pubFile, err := os.Create(filename)
	if err != nil {
		return err
	}
	defer pubFile.Close()

	pubBytes, err := x509.MarshalPKIXPublicKey(publicKey)
	if err != nil {
		return err
	}

	err = pem.Encode(pubFile, &pem.Block{
		Type:  "RSA PUBLIC KEY",
		Bytes: pubBytes,
	})
	return err
}

func main() {
	// Generate the RSA keys (private and public)
	privateKey, publicKey, err := generateRSAKeys(2048) // 2048-bit key size
	if err != nil {
		log.Fatalf("Error generating RSA keys: %v", err)
	}

	// Define paths where the keys will be saved
	privateKeyPath := "config/rsa_private.pem"
	publicKeyPath := "config/rsa_public.pem"

	// Save the private key to the specified file
	err = savePrivateKeyToFile(privateKey, privateKeyPath)
	if err != nil {
		log.Fatalf("Error saving private key to file: %v", err)
	}

	// Save the public key to the specified file
	err = savePublicKeyToFile(publicKey, publicKeyPath)
	if err != nil {
		log.Fatalf("Error saving public key to file: %v", err)
	}

	// Successfully saved the keys
	fmt.Println("RSA key pair generated and saved successfully!")
	fmt.Printf("Private key saved to: %s\n", privateKeyPath)
	fmt.Printf("Public key saved to: %s\n", publicKeyPath)
}
```

### Explanation of Changes:
1. **Path Update**: 
   - The private and public keys will now be saved in the `config/` directory as `rsa_private.pem` and `rsa_public.pem`, respectively.
   - The file paths are now defined as `config/rsa_private.pem` and `config/rsa_public.pem`.

2. **Directory Structure**:
   - The `generate_rsa` Go file will remain in `cmd/generate_rsa/main.go`.
   - Make sure the `config` directory exists in the project root. If it doesn't exist, you can create it.

3. **Saving Keys**:
   - The keys will be saved in the specified paths under the `config` directory.

### Directory Structure:

After running the code, the directory structure will look like this:

```
/project-root
  /cmd
    /generate_rsa
      main.go  <-- This file
  /config
    rsa_private.pem  <-- Generated private key
    rsa_public.pem   <-- Generated public key
  /go.mod
  /go.sum
```

### Example of Running the Program:

1. **Navigate to the `cmd/generate_rsa` directory**:
    ```bash
    cd cmd/generate_rsa
    ```

2. **Run the `main.go` to generate the keys**:
    ```bash
    go run main.go
    ```

3. **After running the program, you will see the output**:
    ```bash
    RSA key pair generated and saved successfully!
    Private key saved to: config/rsa_private.pem
    Public key saved to: config/rsa_public.pem
    ```

### Example of Content in `rsa_private.pem`:

```
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAyptLQfWjx0FEjOcM5FChJlEHYl7J03rPRrZ8Jc2cWxPEnnp1
VbV3bErnN0S1D7DtOSQXBdlCEJzO+Vwv9IcUwFZ2ceI1IXhClsk9MzHODZm38Dbw
ZK2W1Y0IWShzz0tRUavF7hzpKh8wZ6soD0dqx7g91yFbUlE8ukPEWbbR0jW5d+zM
lAFZoswF79B0oGBvKqI4pQzHpvDNxzAtozdlUwrz6m8vnlctJo5TZo0CkCZzydIh
VtJwU5Za9RhmrntQxBfp7HkUebzBCkE5Jj04WV0jWYt3Rg18p5AGP6cLRwIuAKfE
3ObTOakqJfD6nlYxsR9xWjxuVNNkBYbVs6Gb/wIDAQABAoIBAQCNyqW9gAwzfsqM
...
-----END RSA PRIVATE KEY-----
```

### Example of Content in `rsa_public.pem`:

```
-----BEGIN RSA PUBLIC KEY-----
MIIBCgKCAQEAyptLQfWjx0FEjOcM5FChJlEHYl7J03rPRrZ8Jc2cWxPEnnp1VbV3
bErnN0S1D7DtOSQXBdlCEJzO+Vwv9IcUwFZ2ceI1IXhClsk9MzHODZm38DbwZK2W
1Y0IWShzz0tRUavF7hzpKh8wZ6soD0dqx7g91yFbUlE8ukPEWbbR0jW5d+zMlAFZo
swF79B0oGBvKqI4pQzHpvDNxzAtozdlUwrz6m8vnlctJo5TZo0CkCZzydIhVtJwU5
Za9RhmrntQxBfp7HkUebzBCkE5Jj04WV0jWYt3Rg18p5AGP6cLRwIuAKfE3ObTOa
kqJfD6nlYxsR9xWjxuVNNkBYbVs6Gb/wIDAQAB
-----END RSA PUBLIC KEY-----
```

With this setup, you now have a script that generates and stores RSA keys in a specific directory (`config/`), which can then be used for JWT signing, encryption, or other cryptographic purposes.
