I'll create a comprehensive authentication module with multiple authentication methods. Let's build this step by step.

First, let's create the UI components for the authentication flow:



```tsx
import React, { useState } from 'react';
import { Mail, Phone, Key, Github, Twitter, Google } from 'lucide-react';
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
              <Google className="h-4 w-4" />
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
