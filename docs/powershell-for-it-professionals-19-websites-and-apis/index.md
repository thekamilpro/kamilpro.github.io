# PowerShell for IT Professionals [#19] – Websites and APIs

{{<youtube dzaqNeblPF4>}}

PowerShell is a great server automation tool, but what about Internet and any other web served services?

As it turns out, PowerShell is great in scrapping websites and consuming APIs &#8211; and it&#8217;s been one of the main development areas of the tool in the last couple of years.

In this lesson we are going to see how to use PowerShell to download files, scrap websites, discover links. 

We will then use an API of [PasswordPusher (pwpush.com)][1] to generate expiring links to our secret passwords &#8211; all from within PowerShell.

## Script

```powershell
Set-Location C:\temp2; clear
$PSVersionTable
### Invoke-WebRequest
# Downloading files
$uri = "https://github.com/notepad-plus-plus/notepad-plus-plus/releases/download/v7.9.1/npp.7.9.1.Installer.x64.exe"
$here = Get-Location

Invoke-WebRequest -Uri $uri
Invoke-WebRequest -Uri $uri -OutFile .\notepad1.exe
ls

$wc = &#91;System.Net.Webclient]::New()
$wc.DownloadFile($uri, "$here\notepad2.exe")
ls

Start-BitsTransfer -Source $uri -Destination "$here\notepad3.exe"
ls

$Website = Invoke-WebRequest -Uri "bbc.com"
$Website
$Website.StatusCode; $Website.StatusDescription; $Website.Headers
$Website.RawContent 
$Website.RawContent | Out-File website.html
explorer .
# Websites
$Website.Links
$Website.Links | where {$_.class -eq "top-list-item__link"}
$Website.Links | where {$_.class -eq "top-list-item__link"} | select data-bbc-title,href

# Resolving shortcut urls
$Website.Images
$Website.images | where {$_.src -like "http*"}
$Website.images | where {$_.src -like "http*"} | ForEach-Object  {Start-BitsTransfer -Source $_.src -Destination "$($_.alt).jpg" }
ls
explorer .

# Short urls
$uri = "https://bit.ly/38JsRDB"
$response = Invoke-WebRequest -Uri $uri
$response.BaseResponse.RequestMessage.RequestUri.AbsoluteUri

# Api
$Uri = "https://pwpush.com/p.json"

# Json
$json = '{
    "password": {
      "expire_after_views": 5,
      "expire_after_days": 2,
      "payload": "mysecretpassword"
    }
  }'

$request = Invoke-WebRequest -Uri $uri -Method "Post" -ContentType "application/json" -Body $json
$request
$request.Content
$request.Content | ConvertFrom-Json

$body = @{
    password = @{
    "payload" = "mysecretpassword"
    "expire_after_days" = 2
    "expire_after_views" = 5
    }
}
$body
$body.password 
$Body | ConvertTo-Json
$Body

$body = @{
    password = @{
    "payload" = "mysecretpassword"
    "expire_after_days" = 2
    "expire_after_views" = 5
    }
} | ConvertTo-Json

Invoke-WebRequest -Uri $uri -Method "Post" -ContentType "application/json" -Body $body

###Invoke-RestMethod

$Uri = "https://pwpush.com/p.json"
 
$params = @{
    uri = $uri
    ContentType = "application/json"
    Method = "Post"
    body = $body
}
$password = Invoke-RestMethod @params
$password

# Let's confirm it worked
$uri = "https://pwpush.com/p/{0}.json" -f $password.url_token
Invoke-RestMethod -Uri $uri
```

 [1]: https://pwpush.com/
