To integrate Google 2FA (Two-Factor Authentication) into your project, we can create a new folder within the `utils` directory to handle the 2FA functionality, as the reference code is already related to utility functions. This will allow you to manage 2FA operations (e.g., creating and validating 2FA) centrally.

### Project Structure Update:

```bash
/my_project
├── cmd/
├── config/
├── dto/
├── go.mod
├── go.sum
├── handler/
├── middleware/
├── model/
├── repository/
├── routes.md
├── scripts/
├── utils/
│   ├── 2fa/                # New folder for 2FA utilities
│   │   └── google_2fa.go    # Google 2FA related code
│   ├── response.go
│   ├── token.go
├── README.md
```

### Steps:

1. **Create a new folder for 2FA**: Inside `utils/`, create a folder called `2fa/` (or `google_2fa/` for clarity).

2. **Place 2FA-related code**: Add the provided code for creating and validating 2FA to a new file inside the `utils/2fa` folder (e.g., `google_2fa.go`).

3. **Update `README.md`**: Include instructions on how to use Google 2FA.

---

### `google_2fa.go` - Inside `utils/2fa/`

```go
package twofa

import (
	"bytes"
	"image/png"
	"os"
	"path/filepath"
	"strings"

	"github.com/pquerna/otp"
	"github.com/pquerna/otp/totp"
	"github.com/Marine-Drone-Tech/aioceaneye-api-go/config"
)

// Create2fa generates a TOTP key and QR code for the user.
func Create2fa(username, path string) (*otp.Key, error) {
	key, err := totp.Generate(totp.GenerateOpts{
		Issuer:      config.AppHost,  // Using your app's host as the issuer
		AccountName: username,        // Using username for account name
	})

	if err != nil {
		return nil, err
	}

	var buf bytes.Buffer
	img, err := key.Image(200, 200) // Create a QR code image
	if err != nil {
		return nil, err
	}
	png.Encode(&buf, img)

	// Save the QR code image to disk (in the storage/qrcode folder)
	dir, err := os.Getwd()
	if err != nil {
		return nil, err
	}

	fullPath := filepath.Join(dir, "storage", "qrcode", path+username+".png")
	if err := os.WriteFile(fullPath, buf.Bytes(), 0644); err != nil {
		return nil, err
	}

	return key, nil
}

// Validate2fa validates the OTP entered by the user.
func Validate2fa(otpInput, secret string) bool {
	trimmedToken := strings.TrimSpace(otpInput)

	// Validate the OTP token against the secret (TOTP)
	return totp.Validate(trimmedToken, secret)
}
```

---

### How to Use:

You can now use these functions to generate a 2FA key, create a QR code for the user to scan, and validate the 2FA token during the login process.

1. **Generate 2FA QR Code**: When a user sets up 2FA, you generate the 2FA key and create a QR code.
   
2. **Validate 2FA Token**: During login, after verifying the user's username and password, you can validate the OTP (one-time password) entered by the user against the secret stored in the database.

### Example of Integrating 2FA into User Login:

1. **In the `auth_handler.go` (or similar file)**:

```go
package handler

import (
	"encoding/json"
	"net/http"

	"github.com/Marine-Drone-Tech/aioceaneye-api-go/utils/twofa"
	"github.com/Marine-Drone-Tech/aioceaneye-api-go/model"
	"github.com/Marine-Drone-Tech/aioceaneye-api-go/service"
)

// UserLogin handles user login with 2FA verification
func (h *AuthHandler) UserLogin(w http.ResponseWriter, r *http.Request) {
	var userLoginRequest model.UserLoginRequest
	if err := json.NewDecoder(r.Body).Decode(&userLoginRequest); err != nil {
		http.Error(w, "Invalid request", http.StatusBadRequest)
		return
	}

	// Validate username and password (simplified)
	user, err := h.AuthService.ValidateUser(userLoginRequest.Username, userLoginRequest.Password)
	if err != nil {
		http.Error(w, "Invalid credentials", http.StatusUnauthorized)
		return
	}

	// After successful login, check if 2FA is enabled for the user
	if user.TwofaEnabled {
		// Generate a 2FA token and send it for verification
		// Assuming you have the secret stored securely in the database
		valid := twofa.Validate2fa(userLoginRequest.Totp, user.TwofaSecret)
		if !valid {
			http.Error(w, "Invalid 2FA token", http.StatusUnauthorized)
			return
		}
	}

	// Continue to generate JWT token or session
	// Respond with successful login (JWT or session cookie)
	w.WriteHeader(http.StatusOK)
}
```

2. **In the `User` Model (`user.go`)**, you can add the `TwofaEnabled` and `TwofaSecret` fields to track if 2FA is enabled for the user:

```go
type User struct {
	// other fields...
	TwofaEnabled bool   `gorm:"default:false" json:"twofa_enabled"`
	TwofaSecret  string `gorm:"size:255" json:"twofa_secret,omitempty"` // Store the 2FA secret
}
```

### Benefits of This Approach:

- **Centralized 2FA Logic**: By placing the 2FA logic in the `utils/2fa` folder, you keep your code organized and reusable across different parts of the application.
  
- **Security**: The secret for 2FA is stored securely (e.g., in the database) and can be validated during the login process, ensuring that only users with the correct OTP can access their account.

---

### Conclusion:

By adding the `2fa` folder to `utils/`, you keep your Google 2FA logic separate, making it reusable and maintainable. You can generate the 2FA secret and QR code when a user enables 2FA and validate the 2FA code during the login process, enhancing security.
