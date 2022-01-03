# Resolve IP and DNS with Powershell tool

I&#8217;ve been asked to prepare a simple tool which would:

  * Resolve IP address or DNS
  * Return all results to table
  * Import addresses from a file

The reason behind was to check how IP/DNS records change over the period of time, therefore there was a need for something like that:

  * A file with all addresses in question
  * Tool which could import all these addresses and resolve them
  * Save the results into a new file

Once having at least a couple of files, the results could be compared.

There are tools like ping, nslookup or Powershell&#8217;s Resolve-DnsName but they simply can&#8217;t meet all the requiments (or at least not to my knowledge).  So I simply needed to prepare the dedicated tool.

And thus I&#8217;ve created Resolve-IPDNS, the source code at the moment of writing this post looks like this:

<pre class="lang:ps decode:true" title="Resolve-IPDNS">function Resolve-IPDNS {
&lt;#
.SYNOPSIS
Resolves IPv4 address to DNS, or DNS to IPv4.  
.DESCRIPTION
This command uses either IPv4 or DNS name to resolve either. It puts all the data into
a structured table, which can be easilly exported if needed. By default it uses 
1.1.1.1 DNS server.
.PARAMETER IPDNS
One or more IPs or DNS names. Values can be piped.
.PARAMETER DNSServer
Specifies which DNS server will resolve all the queries.
.EXAMPLE
Resolve-IPDNS -IPDNS bbc.com,8.8.8.8
Resolves "bbc.com" and "8.8.8.8" returning their retrospective addresses. 
.EXAMPLE
(Get-Content C:\temp\test.txt) | Resolve-IPDNS
This example will try to resolve all entries from the file
.NOTES
Createad by Kamil Procyszyn
last updated April 2018
https://kamilpro.com
#&gt;    
    
    [CmdletBinding()]
    param(
        [Parameter(Mandatory = $true,ValueFromPipeline = $true)]
        [string[]]$IPDNS,

        [string]$DNSServer = "1.1.1.1"

    )#param

    BEGIN {}

    Process {
        foreach ($addrs in $IPDNS ) {
        
            #Resolve IP or DNS
            IF ($addrs -as [ipaddress]) {
                $resolve_params = @{
                    'Name'   = $addrs
                    'Server' = $DNSServer
                    'Type'   = 'PTR'
                }
            }
            ELSE {
                $resolve_params = @{
                    'Name'   = $addrs
                    'Server' = $DNSServer
                    'Type'   = 'A'
                }
            }
          
            ForEach ($resolve in Resolve-DnsName @resolve_params) {

                #Create props
                IF ($addrs -as [ipaddress]) {
                    $props = @{
                        'IP'  = $resolve.Name
                        'DNS' = $resolve.NameHost
                    }
                }
                ELSE {
                    $props = @{
                        'IP'  = $resolve.IPAddress
                        'DNS' = $resolve.Name
                    }
                
                }#IF
       
                #Output data
                $obj = New-Object -TypeName psobject -Property $props
                Write-Output $obj
                
            }#Foreach $resolve in dns-resolve
        
        }#foreach $addrs in $ipdns

    }#Process
    END {}
}#function

Resolve-IPDNS</pre>

Let&#8217;s break down the function now:

<pre class="lang:ps decode:true ">foreach ($addrs in $IPDNS</pre>

Since $IPDNS can take multiple values, I want to process the function for every address it is given.

<pre class="lang:ps decode:true">#Resolve IP or DNS
            IF ($addrs -as [ipaddress]) {
                $resolve_params = @{
                    'Name'   = $addrs
                    'Server' = $DNSServer
                    'Type'   = 'PTR'
                }
            }
            ELSE {
                $resolve_params = @{
                    'Name'   = $addrs
                    'Server' = $DNSServer
                    'Type'   = 'A'
                }
            }

ForEach ($resolve in Resolve-DnsName @resolve_params)</pre>

The code above prepares the parameters which are going to be parsed to Resolve-DnsName . It establishes if the value is the IP address or DNS name &#8220;(IF ($addrs -as [ipaddress])&#8221; and then specifies the appropiate query type (Either A record for DNS or PTR record for IP).

It&#8217;s not really necessary, but gives more control over function.

<pre class="lang:ps decode:true">#Create props
                IF ($addrs -as [ipaddress]) {
                    $props = @{
                        'IP'  = $resolve.Name
                        'DNS' = $resolve.NameHost
                    }
                }
                ELSE {
                    $props = @{
                        'IP'  = $resolve.IPAddress
                        'DNS' = $resolve.Name
                    }
                
                }#IF</pre>

Now, the script prepares properties for the new custom object. Since Powershell uses different property names for different queries, it&#8217;s essential to expand the right ones. As you can see above, for the IP address the IP and DNS properties are called Name and NameHost respectively, while for DNS they&#8217;re called IPAddress and Name, respectively.

By creating properties that way, I&#8217;m able to create an object with two consistent collums: IP and DNS which you can see bellow:

<pre class="lang:ps decode:true">$obj = New-Object -TypeName psobject -Property $props
 Write-Output $obj</pre>

This is it, Powershell will output the nicely formated data onto your screen now:

<pre class="lang:ps decode:true">IP                   DNS
--                   ---
1.1.1.1.in-addr.arpa 1dot1dot1dot1.cloudflare-dns.com
151.101.0.81         bbc.com
151.101.64.81        bbc.com
151.101.128.81       bbc.com
151.101.192.81       bbc.com
151.101.1.67         cnn.com
151.101.65.67        cnn.com
151.101.129.67       cnn.com
151.101.193.67       cnn.com
8.8.8.8.in-addr.arpa google-public-dns-a.google.com</pre>

As you can see, it can even deal with round robing entries!

You can obviously pipe the data into the function and export it to e.g. csv.

You can always find up-to-date version of the tool here: <https://github.com/kprocyszyn>

Main photo by: [Jungwoo Hong][1] on [Unsplash][2]

 [1]: https://unsplash.com/photos/cYUMaCqMYvI?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText
 [2]: https://unsplash.com/search/photos/point-sign?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText

