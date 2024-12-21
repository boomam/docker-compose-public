# Traefik Plugins
The below are some basic config examples for common middleware plugins -  
## Adding plugins to apps
Just edit the labels on the given app, to include the relevant plugin-name@docker into its middleware label, and that particular app will then begin using the plugin, based on the plugin settings defined at 'traefik level'.  

## Geo-Block
https://plugins.traefik.io/plugins/62d6ce04832ba9805374d62c/geo-block  

### Add plugin via Traefik Plugin Registry
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
### Testing where an IP is according to the API  
Not directly related to the plugin per-say, but if you want to test, open a web browser and enter the following URL -  
`https://get.geojs.io/v1/ip/country/IP_Address` - where `IP_Address` is literally an IP address to check.  

The website will then show a basic two letter ISO code for that particular IP.  
