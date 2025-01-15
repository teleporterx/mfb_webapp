# User Auth. Module

> The **User Authentication Module** is responsible for handling user authentication. It should be able to register new users and log in existing users.

## Endpoints & Security (Overview):
All endpoints will follow in the pattern `/v1` in line with the API versioning best practice.

**Endpoints:**
1. **POST /v1/auth/register**: Register a new user.
2. **POST /v1/auth/login**: Log in an existing user.

**Security:**

**JWT authentication** for user session management after login.

### 1. User Registration (POST /v1/auth/register)
**Purpose:** Allow a user to create an account by providing email and password.

**Request:**
```json
{
  "email": "user@example.com",
  "password": "securepassword"
}
```

**Response 200 (Success):**
```json
{
  "message": "User registered successfully"
}
```
**Response 400 (Bad Request: User already exists):**
```json
{
  "error": "User already exists"
}
```
**Response 400 (Bad Request: Invalid Email):**
```json
{
  "detail": "Invalid email format"
}
```
**Response 400 (Bad Request: Invalid password):**
```json
{
  "detail": "Password must be at least 8 characters long, include at least one uppercase letter, one lowercase letter, and one number"
}
```

#### /v1/auth/models.py: SNIP (registration request model)
```python
from pydantic import BaseModel, Field, field_validator
from fastapi import HTTPException
from api.v1.auth.req_validate import validate_password, validate_email

class UserRegistrationRequest(BaseModel):
    email: str = Field(..., description="The user's email address")
    password: str = Field(..., description="The user's password")

    @field_validator('email')
    def validate_email_field(cls, value):
        if not validate_email(value):
            raise HTTPException(status_code=400, detail='Invalid email format')
        return value

    @field_validator('password')
    def validate_password_field(cls, value):
        if not validate_password(value):
            raise HTTPException(status_code=400, detail='Password must be at least 8 characters long, include at least one uppercase letter, one lowercase letter, and one number')
        return value
```

#### /v1/auth/auth_routes.py: SNIP (registration)
```python
@auth_router.post("/register")
async def register_user(request: UserRegistrationRequest):
    try:
        # Check if the email is already registered
        existing_user = await mongo_service.find_one("mfb_webapp", "user_data", {"email": request.email})
        if existing_user:
            raise HTTPException(status_code=400, detail="Email already registered")
        
        # Hash the password (use a hashing library like bcrypt, or Argon2 here)
        hashed_password = request.password

        # Create the user data object
        user_data = {
            "email": request.email,
            "password": hashed_password,
            # Add any other necessary fields
        }

        # Save to the database
        await mongo_service.insert_one("mfb_webapp", "user_data", user_data)
        return {"message": "User registered successfully."}
    
    except HTTPException as http_exc:
        raise http_exc  # Re-raise HTTP exceptions (e.g., email already registered)
    
    except Exception as e:
        # Catch any general exception, e.g., MongoDB connection issues
        raise HTTPException(status_code=500, detail=f"[Is MongoDB offline?] Internal server error: {str(e)}")
```

#### /v1/auth/req_validate.py: SNIP (request validation)

```python
"""
Propagates errors to auth_routes invoked by UserRegistrationRequest
"""
import re

def validate_email(email: str) -> bool:
    # Simple email validation regex
    email_regex = r'^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$'
    return re.fullmatch(email_regex, email) is not None

def validate_password(password: str) -> bool:
    # Example of password validation: At least 8 characters, 1 uppercase, 1 lowercase, 1 number
    min_length = 8
    if len(password) < min_length:
        return False
    if not any(char.isdigit() for char in password):
        return False
    if not any(char.isupper() for char in password):
        return False
    return True
```

### 2. User Login (POST /v1/auth/login)
**Purpose:** Allow an existing user to log in by providing email and password.

**Request:** 
```json
{
    "email": "alal@s.com",
    "password": "Scaa2sas@#s"
}
```

**Response 200 (Success):**
```json
{
  "message": "Login successful", "access_token": access_token, "user": user_data
}
```
**Response 400 (Bad Request: Invalid Email):**
```json
{
  "detail": "Invalid email format"
}
```
**Response 400 (Bad Request: Invalid password):**
```json
{
  "detail": "Password must be at least 8 characters long, include at least one uppercase letter, one lowercase letter, and one number"
}
```

#### /v1/auth/auth_routes.py: SNIP (login)
```python
@auth_router.post("/login")
async def login_user(request: UserRegistrationRequest): # Reuse the same request model to reduce redundancy
    try:
        # Check if the email is registered
        user_data = await mongo_service.find_one("mfb_webapp", "user_data", {"email": request.email})
        if not user_data:
            raise HTTPException(status_code=401, detail="Invalid email")
        
        # Verify the password
        if not AuthSecurity.verify_password(request.password, user_data['password']):
            raise HTTPException(status_code=401, detail="Invalid password")
        
        # Generate JWT token
        # Debug here for JWT token generation issues (usually .env related)
        access_token =AuthSecurity.create_access_token(data={"email": request.
        email})

        #`user_data` contains the user document retrieved from the database which includes a sensitive field
        if "password" in user_data:
            user_data["password"] = "[REDACTED]"
        return {"message": "Login successful", "access_token": access_token, "user": user_data}
    
    except HTTPException as http_exc:
        raise http_exc  # Re-raise HTTP exceptions
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Internal server error: {str(e)}")
```

The *Registration Request Model* can be reused here to minimize redundant code.

### 3. Authentication Security:
To ensure a simple yet robust authentication process, we will implement a JWT auth system for sesssion management. This will be done in the `auth_security.py`

**auth_security.py**
```python
# /api/v1/auth/auth_security.py

from fastapi import HTTPException
from passlib.context import CryptContext
from datetime import datetime, timedelta, timezone
import jwt

from api.v1.config import CONFIG

SECRET_KEY = CONFIG['JWT_SECRET_KEY']  # Make sure to set this in your config
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30 # Token expiration time

# Create a password hashing context using bcrypt or Argon2 (depending on your choice)
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

class AuthSecurity:
    @staticmethod
    def hash_password(password: str) -> str:
        """
        Hash the given password.

        Args:
            password (str): The plaintext password to hash.

        Returns:
            str: The hashed password.
        """
        return pwd_context.hash(password)

    @staticmethod
    def verify_password(plain_password: str, hashed_password: str) -> bool:
        """
        Verify the given password against the hashed password.

        Args:
            plain_password (str): The plaintext password.
            hashed_password (str): The hashed password.

        Returns:
            bool: True if the password matches, False otherwise.
        """
        return pwd_context.verify(plain_password, hashed_password)
    
    @staticmethod
    def create_access_token(data: dict, expires_delta: timedelta = None):
        to_encode = data.copy()
        if expires_delta:
            expire = datetime.now(timezone.utc) + expires_delta  # Updated line
        else:
            expire = datetime.now(timezone.utc) + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)  # Updated line
        to_encode.update({"exp": expire})
        encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
        return encoded_jwt

    @staticmethod
    def decode_access_token(token: str):
        try:
            decoded_data = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
            return decoded_data
        except jwt.ExpiredSignatureError:
            raise jwt.ExpiredSignatureError("Token has expired.")
        except jwt.InvalidTokenError:
            raise jwt.InvalidTokenError("Invalid token.")

    @staticmethod
    def get_current_user(token: str):
        """
        Decodes the JWT token and returns the user information if valid.
        
        Args:
            token (str): The JWT token.
        
        Returns:
            dict: Decoded user information from the token.
        
        Raises:
            HTTPException: If token is expired or invalid.
        """
        try:
            decoded_data = AuthSecurity.decode_access_token(token)
            return decoded_data  # Return the decoded token's payload (user information)
        except Exception as e:
            raise HTTPException(status_code=401, detail=str(e))  # Unauthorized or invalid token
```

The `get_current_user` function is used to verify the JWT token and return the user information if valid. If the token is expired or invalid, it raises an `HTTPException` with a `401` status code and a message indicating token expiry.

This way the front end can manage the user's session and restrict access to the pages beyond login.