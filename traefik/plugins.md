# Traefik Plugins
The below are some basic config examples for common middleware plugins -  

## Adding plugins to apps (per middleware)
Just edit the labels on the given app, to include the relevant plugin-name@docker into its middleware label, and that particular app will then begin using the plugin, based on the plugin settings defined at 'traefik level'.  

## Add to all apps as part of a chain
Edit the Traefik docker-compose labels, editing the chain based middleware with a comma separated list of middlewares - 
```yaml
      # Middleware - Secure Chain      
      - "traefik.http.middlewares.secured.chain.middlewares=redirect-http-to-https@internal,cloudflare-real-ip,default-headers,geo-block,ipallowlist"
```
Then, assuming `secured@docker` is added as the sole middleware on each app, it will filter down to each.

## Plugins
 - [Geo-Block](###Geo-Block)
 - [Cloudflare Real IP](###Cloudflare-Real-IP)

### Geo-Block
https://plugins.traefik.io/plugins/62d6ce04832ba9805374d62c/geo-block  

#### Add plugin via Traefik Plugin Registry
Add the following in your docker-compose to the `command:` section of the file -  
```yaml
    # Plugins - geo-block
      - "--experimental.plugins.geo-block.modulename=github.com/PascalMinder/geoblock"
      - "--experimental.plugins.geo-block.version=v0.2.8"
```
Next, add the following to the `labels:` section -  
```yaml
      # Middleware/Plugin - GeoIP # https://plugins.traefik.io/plugins/62d6ce04832ba9805374d62c/geo-block
      - "traefik.http.middlewares.geo-block.plugin.geo-block.silentStartUp=false"
      - "traefik.http.middlewares.geo-block.plugin.geo-block.allowLocalRequests=true"
      - "traefik.http.middlewares.geo-block.plugin.geo-block.logLocalRequests=false"
      - "traefik.http.middlewares.geo-block.plugin.geo-block.logAllowedRequests=false"
      - "traefik.http.middlewares.geo-block.plugin.geo-block.logApiRequests=false"
      - "traefik.http.middlewares.geo-block.plugin.geo-block.api=https://get.geojs.io/v1/ip/country/{ip}"
      - "traefik.http.middlewares.geo-block.plugin.geo-block.apiTimeoutMs=750"
      - "traefik.http.middlewares.geo-block.plugin.geo-block.cacheSize=15"
      - "traefik.http.middlewares.geo-block.plugin.geo-block.forceMonthlyUpdate=false"
      - "traefik.http.middlewares.geo-block.plugin.geo-block.allowUnknownCountries=false"
      - "traefik.http.middlewares.geo-block.plugin.geo-block.unknownCountryApiResponse=nil"
      - "traefik.http.middlewares.geo-block.plugin.geo-block.blackListMode=true"
      - "traefik.http.middlewares.geo-block.plugin.geo-block.addCountryHeader=false"
      - "traefik.http.middlewares.geo-block.plugin.geo-block.countries=CN,RU,KP,IR"
      - "traefik.http.middlewares.geo-block.plugin.geo-block.allowedIPAddresses=CIDR1,CIDR2,CIDR3"
```
Make sure you modify the bottom two lines to list the countries (based on [ISO code](https://en.wikipedia.org/wiki/List_of_ISO_3166_country_codes)) you want blocked, and the CIDR blocks for your home network too (if your server has a routable connection back to your home network, if not, your home external IP should suffice).  
&nbsp;  
Lastly, edit both the secured-chain middleware to add GeoBlock to that should you ever need it too - 
```yaml
      - "traefik.http.middlewares.secured.chain.middlewares=ipallowlist,default-headers,geo-block"
```
& the router for the Traefik dashboard to add it so that its in use on the dashboard too - 
```yaml
      - "traefik.http.routers.traefik-dashboard.middlewares=ipallowlist,geo-block@docker"
```
Following this, deploy Traefik again/update the stack.  
In the log, you should see simlar to this, showing the plugin downloaded, and then configured itself - 
```
INFO: GeoBlock: 2024/12/21 12:43:03 Denied request status code: 403
INFO: GeoBlock: 2024/12/21 12:43:03 countries: [CN RU KP IR]
INFO: GeoBlock: 2024/12/21 12:43:03 add country header: false
INFO: GeoBlock: 2024/12/21 12:43:03 blacklist mode: true
INFO: GeoBlock: 2024/12/21 12:43:03 unknown country api response: nil
INFO: GeoBlock: 2024/12/21 12:43:03 allow unknown countries: false
INFO: GeoBlock: 2024/12/21 12:43:03 force monthly update: false
INFO: GeoBlock: 2024/12/21 12:43:03 cache size: 15
INFO: GeoBlock: 2024/12/21 12:43:03 ignore API timeout: false
INFO: GeoBlock: 2024/12/21 12:43:03 API timeout: 750
INFO: GeoBlock: 2024/12/21 12:43:03 API uri: https://get.geojs.io/v1/ip/country/{ip}
INFO: GeoBlock: 2024/12/21 12:43:03 use custom HTTP header field for country lookup: false
INFO: GeoBlock: 2024/12/21 12:43:03 log api requests: false
INFO: GeoBlock: 2024/12/21 12:43:03 log allowed requests: false
INFO: GeoBlock: 2024/12/21 12:43:03 log local requests: false
INFO: GeoBlock: 2024/12/21 12:43:03 allow local IPs: true
```
#### Testing where an IP is according to the API  
Not directly related to the plugin per-say, but if you want to test, open a web browser and enter the following URL -  
`https://get.geojs.io/v1/ip/country/IP_Address` - where `IP_Address` is literally an IP address to check.  

The website will then show a basic two letter ISO code for that particular IP.  

### Cloudflare Real-IP
[https://plugins.traefik.io/plugins/62d6ce04832ba9805374d62c/geo-block](https://plugins.traefik.io/plugins/62e97498e2bf06d4675b9443/real-ip-from-cloudflare-proxy-tunnel)

#### Add plugin via Traefik Plugin Registry
Add the following in your docker-compose to the `command:` section of the file -  
```yaml
    # Plugins - geo-block
      - "--experimental.plugins.cloudflare-real-ip.modulename=github.com/BetterCorp/cloudflarewarp"
      - "--experimental.plugins.cloudflare-real-ip.version=v1.3.3"
```
Next, add the following to the `labels:` section -  
```yaml
      # Middleware/Plugin - Cloudflare-Real-IP #
      - "traefik.http.middlewares.cloudflare-real-ip.plugin.cloudflare-real-ip.disableDefault=false"
      #- "traefik.http.middlewares.cloudflare-real-ip.plugin.cloudflare-real-ip.trustip=" 
```
_Trust IPS not required if disableDefault is false - we will allocate Cloud Flare IPs automatically_  
&nbsp;  
# Configure middlewares for usage
Next, edit both the secured-chain middleware to add GeoBlock to that should you ever need it too - 
```yaml
      - "traefik.http.middlewares.secured.chain.middlewares=ipallowlist,default-headers,geo-block,cloudflare-real-ip"
```
& the router for the Traefik dashboard to add it so that its in use on the dashboard too - 
```yaml
      - "traefik.http.routers.traefik-dashboard.middlewares=ipallowlist,geo-block@docker,cloudflare-real-ip@docker"
```
&nbsp;  
Following this, deploy Traefik again/update the stack.  

#### Alternative
Go-to your `traefik.yaml` and add 
```yaml
entryPoints:
...
  https:
    address: ":443"
    forwardedHeaders:
      insecure: false # true will trust all forwarded header sources
      trustedIPs:
        # Start of Cloudflare public IP list for HTTP requests, remove this if you don't use it; https://www.cloudflare.com/de-de/ips/
        - 103.21.244.0/22
        - 103.22.200.0/22
        - 103.31.4.0/22
        - 104.16.0.0/13
        - 104.24.0.0/14
        - 108.162.192.0/18
        - 131.0.72.0/22
        - 141.101.64.0/18
        - 162.158.0.0/15
        - 172.64.0.0/13
        - 173.245.48.0/20
        - 188.114.96.0/20
        - 190.93.240.0/20
        - 197.234.240.0/22
        - 198.41.128.0/17
        - 2400:cb00::/32
        - 2606:4700::/32
        - 2803:f800::/32
        - 2405:b500::/32
        - 2405:8100::/32
        - 2a06:98c0::/29
        - 2c0f:f248::/32
        # End of Cloudlare public IP list
        # Internal IP ranges
        - 192.168.0.0/24
        - 192.168.200.0/24
        - 192.168.201.0/24
        - 192.168.66.0/24
        - 192.168.30.0/24
        - 192.168.20.0/24
        # Docker IP Range
        - 172.10.0.0/24
```
You should now see client IPs in your logs.
