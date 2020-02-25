# Passport-OTP Loopback

## Adding dependency

    npm i git+https://github.com/jagzmz/passport-otp.git

## Sample Configuration
    {
       "otp":{
          "defaultCountryCode":"+91",
          "resendAfter":0.5, //in minutes
          "defaultOTPVia":"email",
          "verificationRequired":false,
          "entryFlow":{
             "phoneVerificationRequired":false,
             "emailVerificationRequired":true
          }
       }
    }
## Default Configurations

    "otp":{
                "authScheme":"otp",
                "provider":"passport-otp",
                "module":"passport-otp",
                "callbackHTTPMethod":"post",
                "resendAfter":0.5,
                "digits":6,
                "defaultOTPVia":"email",
                "redirectEnabled":false,
                "verificationRequired":false,
                "entryFlow":{
                   "phoneVerificationRequired":false,
                   "emailVerificationRequired":false
                }
             }
## Config Description
Above are some config which can be used for this strategy, you do not need to use all of the configurations, these will be used by default, if you want to override them, you can do it by providing your own value for that config, and it will be overriden.
See [Overriding Default Configurations](#overriding-default-configurations)

|Config|Type|Description  | Default
|--|--|--|--|
| resendAfter|number | Time to wait for a new OTP request (in minutes).|0.5|
|digits|number|No. of OTP digits to generate. |6
|defaultOTPVia|string| Default configuration to send OTP through.|email
|redirectEnabled|boolean| Passport strategy usually make a redirect on successful/unsuccessful login. | false
|verificationRequired|boolean| If verification required is set to true, requests like `updateEmail, updatePassword, updatePhone` will require OTP verification, otherwise updates will be made by user's Access Token.|false
|entryFlow| object|These have two keys viz. `phoneVerificationRequired, emailVerificationRequired`, if either of these are not set it won't let user to login, or won't provide Access Token while using this strategy.|false
|defaultCountryCode|string| This country code will be used for all phone objects if countryCode is not provided explicitly.|false

## Overriding Default Configurations

You can override strategies default configuration according to your needs.
For this to be done, you need to just provide the key which needs to be overridden in your `authConfig.json`

**Example**: If you want a custom **resendAfter** interval for your OTP strategy, just provide 

    "otp":{
      // Resend Interval for OTP Request (in mins)
      resendAfter:1 	  
    }

Now **resendAfter** for OTP strategy will be set to **1 minute**.
