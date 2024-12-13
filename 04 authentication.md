I'll create a comprehensive authentication module with multiple authentication methods. Let's build this step by step.

First, let's create the UI components for the authentication flow:



```tsx
import React, { useState } from 'react';
import { Mail, Phone, Key, Github, Twitter, Chrome } from 'lucide-react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Alert, AlertDescription } from '@/components/ui/alert';

const AuthModule = () => {
  const [email, setEmail] = useState('');
  const [phone, setPhone] = useState('');
  const [otp, setOtp] = useState('');
  const [authStep, setAuthStep] = useState('initial'); // initial, otp, success
  const [currentMethod, setCurrentMethod] = useState('email');
  const [message, setMessage] = useState('');

  const handleEmailSubmit = async (e) => {
    e.preventDefault();
    setMessage('Magic link sent to your email!');
    setAuthStep('success');
    // API call to send magic link would go here
  };

  const handlePhoneSubmit = async (e) => {
    e.preventDefault();
    setMessage('OTP sent to your phone!');
    setAuthStep('otp');
    // API call to send OTP would go here
  };

  const handleOTPSubmit = async (e) => {
    e.preventDefault();
    setMessage('Authentication successful!');
    setAuthStep('success');
    // API call to verify OTP would go here
  };

  const handleSocialLogin = (provider) => {
    // Social login logic would go here
    console.log(`Logging in with ${provider}`);
  };

  return (
    <div className="min-h-screen bg-gray-50 flex items-center justify-center p-4">
      <Card className="w-full max-w-md">
        <CardHeader>
          <CardTitle className="text-2xl text-center">Welcome</CardTitle>
        </CardHeader>
        <CardContent>
          {message && (
            <Alert className="mb-4">
              <AlertDescription>{message}</AlertDescription>
            </Alert>
          )}

          <Tabs defaultValue="email" className="mb-6">
            <TabsList className="grid grid-cols-2">
              <TabsTrigger 
                value="email" 
                onClick={() => setCurrentMethod('email')}
              >
                Email
              </TabsTrigger>
              <TabsTrigger 
                value="phone" 
                onClick={() => setCurrentMethod('phone')}
              >
                Phone
              </TabsTrigger>
            </TabsList>

            <TabsContent value="email">
              <form onSubmit={handleEmailSubmit} className="space-y-4">
                <div className="space-y-2">
                  <label className="text-sm font-medium">Email</label>
                  <div className="relative">
                    <Mail className="absolute left-3 top-3 h-4 w-4 text-gray-400" />
                    <Input
                      type="email"
                      placeholder="Enter your email"
                      className="pl-9"
                      value={email}
                      onChange={(e) => setEmail(e.target.value)}
                      required
                    />
                  </div>
                </div>
                <Button type="submit" className="w-full">
                  Send Magic Link
                </Button>
              </form>
            </TabsContent>

            <TabsContent value="phone">
              {authStep === 'initial' && (
                <form onSubmit={handlePhoneSubmit} className="space-y-4">
                  <div className="space-y-2">
                    <label className="text-sm font-medium">Phone Number</label>
                    <div className="relative">
                      <Phone className="absolute left-3 top-3 h-4 w-4 text-gray-400" />
                      <Input
                        type="tel"
                        placeholder="Enter your phone number"
                        className="pl-9"
                        value={phone}
                        onChange={(e) => setPhone(e.target.value)}
                        required
                      />
                    </div>
                  </div>
                  <Button type="submit" className="w-full">
                    Send OTP
                  </Button>
                </form>
              )}

              {authStep === 'otp' && (
                <form onSubmit={handleOTPSubmit} className="space-y-4">
                  <div className="space-y-2">
                    <label className="text-sm font-medium">Enter OTP</label>
                    <div className="relative">
                      <Key className="absolute left-3 top-3 h-4 w-4 text-gray-400" />
                      <Input
                        type="text"
                        placeholder="Enter OTP"
                        className="pl-9"
                        value={otp}
                        onChange={(e) => setOtp(e.target.value)}
                        required
                        maxLength={6}
                      />
                    </div>
                  </div>
                  <Button type="submit" className="w-full">
                    Verify OTP
                  </Button>
                </form>
              )}
            </TabsContent>
          </Tabs>

          <div className="relative my-6">
            <div className="absolute inset-0 flex items-center">
              <div className="w-full border-t border-gray-200" />
            </div>
            <div className="relative flex justify-center text-sm">
              <span className="bg-white px-2 text-gray-500">Or continue with</span>
            </div>
          </div>

          <div className="grid grid-cols-3 gap-3">
            <Button
              variant="outline"
              onClick={() => handleSocialLogin('google')}
              className="w-full"
            >
              <Chrome className="h-4 w-4" />
            </Button>
            <Button
              variant="outline"
              onClick={() => handleSocialLogin('github')}
              className="w-full"
            >
              <Github className="h-4 w-4" />
            </Button>
            <Button
              variant="outline"
              onClick={() => handleSocialLogin('twitter')}
              className="w-full"
            >
              <Twitter className="h-4 w-4" />
            </Button>
          </div>
        </CardContent>
      </Card>
    </div>
  );
};

export default AuthModule;

```

Now, let's create the backend API for handling authentication:



```python
from fastapi import FastAPI, HTTPException, Depends
from fastapi.security import OAuth2PasswordBearer
from pydantic import BaseModel, EmailStr
from typing import Optional
import jwt
import boto3
from datetime import datetime, timedelta
import pyotp
import requests

app = FastAPI()

# Configuration
SECRET_KEY = "your-secret-key"
AWS_REGION = "your-aws-region"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# AWS Services
sns = boto3.client('sns', region_name=AWS_REGION)
ses = boto3.client('ses', region_name=AWS_REGION)

# Models
class UserBase(BaseModel):
    email: Optional[EmailStr] = None
    phone: Optional[str] = None

class Token(BaseModel):
    access_token: str
    token_type: str

# Magic Link Authentication
def generate_magic_link(email: str) -> str:
    expiration = datetime.utcnow() + timedelta(minutes=15)
    token = jwt.encode(
        {"email": email, "exp": expiration},
        SECRET_KEY,
        algorithm=ALGORITHM
    )
    return f"https://yourapp.com/verify?token={token}"

@app.post("/auth/magic-link")
async def send_magic_link(email: EmailStr):
    magic_link = generate_magic_link(email)
    
    # Send email using AWS SES
    try:
        ses.send_email(
            Source="noreply@yourapp.com",
            Destination={"ToAddresses": [email]},
            Message={
                "Subject": {"Data": "Your Magic Link"},
                "Body": {
                    "Text": {"Data": f"Click here to login: {magic_link}"}
                }
            }
        )
        return {"message": "Magic link sent successfully"}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

# OTP Authentication
def generate_otp() -> str:
    totp = pyotp.TOTP(pyotp.random_base32())
    return totp.now()

@app.post("/auth/send-otp")
async def send_otp(phone: str):
    otp = generate_otp()
    
    # Store OTP securely (implement secure storage)
    # Send OTP via SNS
    try:
        sns.publish(
            PhoneNumber=phone,
            Message=f"Your OTP is: {otp}"
        )
        return {"message": "OTP sent successfully"}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

# WhatsApp Authentication
def send_whatsapp_otp(phone: str, otp: str):
    # Using WhatsApp Business API
    WHATSAPP_API_URL = "https://graph.facebook.com/v13.0/YOUR_PHONE_NUMBER_ID/messages"
    WHATSAPP_TOKEN = "your-whatsapp-token"
    
    payload = {
        "messaging_product": "whatsapp",
        "to": phone,
        "type": "template",
        "template": {
            "name": "otp_template",
            "language": {
                "code": "en"
            },
            "components": [
                {
                    "type": "body",
                    "parameters": [
                        {
                            "type": "text",
                            "text": otp
                        }
                    ]
                }
            ]
        }
    }
    
    headers = {
        "Authorization": f"Bearer {WHATSAPP_TOKEN}",
        "Content-Type": "application/json"
    }
    
    response = requests.post(WHATSAPP_API_URL, json=payload, headers=headers)
    if response.status_code != 200:
        raise HTTPException(status_code=500, detail="Failed to send WhatsApp message")

@app.post("/auth/whatsapp-otp")
async def send_whatsapp_otp_route(phone: str):
    otp = generate_otp()
    await send_whatsapp_otp(phone, otp)
    return {"message": "WhatsApp OTP sent successfully"}

# Social Login
@app.get("/auth/google")
async def google_login():
    return {"url": "https://accounts.google.com/o/oauth2/v2/auth?..."}

@app.get("/auth/github")
async def github_login():
    return {"url": "https://github.com/login/oauth/authorize?..."}

@app.get("/auth/twitter")
async def twitter_login():
    return {"url": "https://twitter.com/i/oauth2/authorize?..."}

# Verification endpoints
@app.post("/auth/verify-magic-link")
async def verify_magic_link(token: str):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        email = payload.get("email")
        if email is None:
            raise HTTPException(status_code=400, detail="Invalid token")
        # Create session/JWT token
        return {"access_token": create_access_token(email)}
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=400, detail="Token has expired")
    except jwt.JWTError:
        raise HTTPException(status_code=400, detail="Invalid token")

@app.post("/auth/verify-otp")
async def verify_otp(phone: str, otp: str):
    # Verify OTP (implement secure verification)
    # Create session/JWT token
    return {"access_token": create_access_token(phone)}

def create_access_token(data: str) -> str:
    expires = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode = {"sub": data, "exp": expires}
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

```

I'll create the backend using Python with FastAPI. We'll set up authentication endpoints with proper error handling and security measures.



```python
# app/main.py
from fastapi import FastAPI, HTTPException, Depends, status
from fastapi.security import OAuth2PasswordBearer
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, EmailStr
from typing import Optional
import jwt
import secrets
import smtplib
from email.mime.text import MIMEText
from datetime import datetime, timedelta
import redis
import os
import boto3
from botocore.exceptions import ClientError

app = FastAPI()

# CORS configuration
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # In production, replace with specific origins
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Configuration
SECRET_KEY = os.getenv("SECRET_KEY", secrets.token_urlsafe(32))
ACCESS_TOKEN_EXPIRE_MINUTES = 30
REDIS_URL = os.getenv("REDIS_URL", "redis://localhost:6379")

# Initialize Redis for OTP storage
redis_client = redis.from_url(REDIS_URL)

# Initialize AWS SNS client for SMS
sns_client = boto3.client(
    'sns',
    aws_access_key_id=os.getenv('AWS_ACCESS_KEY_ID'),
    aws_secret_access_key=os.getenv('AWS_SECRET_ACCESS_KEY'),
    region_name=os.getenv('AWS_REGION')
)

# Pydantic models
class TokenResponse(BaseModel):
    access_token: str
    token_type: str = "bearer"

class EmailRequest(BaseModel):
    email: EmailStr

class PhoneRequest(BaseModel):
    phone: str

class OTPVerifyRequest(BaseModel):
    phone: str
    otp: str

class MagicLinkVerifyRequest(BaseModel):
    token: str

# Helper functions
def create_access_token(data: dict, expires_delta: timedelta = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm="HS256")
    return encoded_jwt

def generate_otp():
    return ''.join(secrets.choice('0123456789') for _ in range(6))

def send_magic_link_email(email: str, token: str):
    sender_email = os.getenv("SMTP_USER")
    sender_password = os.getenv("SMTP_PASSWORD")
    
    magic_link = f"https://yourapp.com/verify?token={token}"
    
    msg = MIMEText(f"""
    Hello!
    
    Click the following link to sign in:
    {magic_link}
    
    This link will expire in 15 minutes.
    
    If you didn't request this link, please ignore this email.
    """)
    
    msg['Subject'] = 'Your Sign-in Link'
    msg['From'] = sender_email
    msg['To'] = email
    
    try:
        with smtplib.SMTP(os.getenv("SMTP_HOST"), int(os.getenv("SMTP_PORT"))) as server:
            server.starttls()
            server.login(sender_email, sender_password)
            server.send_message(msg)
        return True
    except Exception as e:
        print(f"Failed to send email: {e}")
        return False

# Routes
@app.post("/auth/magic-link", response_model=dict)
async def send_magic_link(request: EmailRequest):
    try:
        # Generate token
        token = create_access_token(
            {"sub": request.email},
            timedelta(minutes=15)
        )
        
        # Send email
        if send_magic_link_email(request.email, token):
            return {"message": "Magic link sent successfully"}
        else:
            raise HTTPException(
                status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
                detail="Failed to send magic link"
            )
    except Exception as e:
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=str(e)
        )

@app.post("/auth/send-otp", response_model=dict)
async def send_otp(request: PhoneRequest):
    try:
        # Generate OTP
        otp = generate_otp()
        
        # Store OTP in Redis with 5-minute expiration
        redis_client.setex(f"otp:{request.phone}", 300, otp)
        
        # Send SMS via AWS SNS
        response = sns_client.publish(
            PhoneNumber=request.phone,
            Message=f"Your OTP is: {otp}. Valid for 5 minutes.",
            MessageAttributes={
                'AWS.SNS.SMS.SenderID': {
                    'DataType': 'String',
                    'StringValue': 'YourApp'
                }
            }
        )
        
        return {"message": "OTP sent successfully"}
    except ClientError as e:
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=f"Failed to send OTP: {str(e)}"
        )

@app.post("/auth/verify-otp", response_model=TokenResponse)
async def verify_otp(request: OTPVerifyRequest):
    stored_otp = redis_client.get(f"otp:{request.phone}")
    
    if not stored_otp:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="OTP expired or not found"
        )
    
    if stored_otp.decode() != request.otp:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Invalid OTP"
        )
    
    # Delete OTP after successful verification
    redis_client.delete(f"otp:{request.phone}")
    
    # Generate access token
    access_token = create_access_token(
        data={"sub": request.phone}
    )
    
    return TokenResponse(access_token=access_token)

@app.post("/auth/verify-magic-link", response_model=TokenResponse)
async def verify_magic_link(request: MagicLinkVerifyRequest):
    try:
        # Verify token
        payload = jwt.decode(request.token, SECRET_KEY, algorithms=["HS256"])
        email = payload.get("sub")
        
        if not email:
            raise HTTPException(
                status_code=status.HTTP_400_BAD_REQUEST,
                detail="Invalid token"
            )
        
        # Generate new access token
        access_token = create_access_token(
            data={"sub": email}
        )
        
        return TokenResponse(access_token=access_token)
    except jwt.ExpiredSignatureError:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Token has expired"
        )
    except jwt.JWTError:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Invalid token"
        )

# OAuth2 scheme for token authentication
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

@app.get("/auth/me")
async def get_current_user(token: str = Depends(oauth2_scheme)):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        return {"user": payload.get("sub")}
    except jwt.JWTError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials"
        )

```

Now, let's create the environment setup and requirements file:

```text
# requirements.txt
fastapi==0.68.0
uvicorn==0.15.0
python-jose[cryptography]==3.3.0
passlib==1.7.4
pydantic==1.8.2
email-validator==1.1.3
python-multipart==0.0.5
redis==4.0.2
boto3==1.24.0
python-dotenv==0.19.0

# .env
SECRET_KEY=your-secret-key-here
REDIS_URL=redis://localhost:6379
AWS_ACCESS_KEY_ID=your-aws-access-key
AWS_SECRET_ACCESS_KEY=your-aws-secret-key
AWS_REGION=your-aws-region
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your-app-specific-password

```

To set up and run the backend:

1. **Install Dependencies**:
```bash
pip install -r requirements.txt
```

2. **Set Up Redis**:
   - Install Redis on your system
   - Or use Redis Cloud for production

3. **AWS Setup**:
   - Create an AWS account
   - Set up IAM user with SNS permissions
   - Configure AWS credentials

4. **Environment Setup**:
   - Copy the `.env` template
   - Fill in your actual credentials
   - Never commit actual credentials to version control

5. **Run the Server**:
```bash
uvicorn main:app --reload
```

Key Features:
1. Magic Link Authentication
2. OTP via SMS
3. Token-based authentication
4. Redis for OTP storage
5. AWS SNS integration for SMS
6. Email integration for magic links
7. Security best practices
8. Error handling
9. CORS support
10. Environment configuration

Would you like me to:
1. Add additional authentication methods?
2. Implement database integration for user management?
3. Add more security features?
4. Create API documentation?
5. Add rate limiting?
6. Implement logging and monitoring?
7. Add unit tests?

Let me know what aspect you'd like to explore further!




To use this authentication system:

1. **Setup Requirements**:
   ```bash
   pip install fastapi python-jose[cryptography] emails passlib python-multipart boto3 pyotp requests
   ```

2. **AWS Configuration**:
   - Set up AWS credentials
   - Configure SES for email sending
   - Configure SNS for SMS
   - Set up appropriate IAM roles and permissions

3. **Environment Variables**:
   ```bash
   export SECRET_KEY="your-secret-key"
   export AWS_ACCESS_KEY_ID="your-access-key"
   export AWS_SECRET_ACCESS_KEY="your-secret-key"
   export AWS_REGION="your-region"
   ```

4. **Social Login Setup**:
   - Create Google OAuth credentials
   - Set up GitHub OAuth App
   - Configure Twitter Developer App
   - Update the respective OAuth URLs and credentials

5. **WhatsApp Integration**:
   - Set up WhatsApp Business API
   - Configure webhook endpoints
   - Update the WhatsApp token and phone number ID

Would you like me to:
1. Add any additional authentication methods?
2. Implement specific security features?
3. Create more detailed setup instructions?
4. Add rate limiting and security measures?
5. Create the database schema for user management?

Let me know what aspect you'd like to explore further!
