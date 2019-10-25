# APIGEE Training Readme

##### Thomas Hegarty for MS3

The purpose of this project is to illustrate core concepts used in this APIGEE test proxy developed in a day and a half for MS3's APIGEE training hackathon.


### Caching

Caching in APIGEE is really simple, especially for caching backend service responses. The ResponseCache element layout in the project is as follows:
```
<ResponseCache async="false" continueOnError="false" enabled="true" name="Response-Cache-1">
    <DisplayName>cache-url-data</DisplayName>
    <Properties/>
    <CacheKey>
        <Prefix/>
        <KeyFragment ref="request.uri" type="string"/>
    </CacheKey>
    <Scope>Exclusive</Scope>
    <ExpirySettings>
        <ExpiryDate/>
        <TimeOfDay/>
        <TimeoutInSec ref="">60</TimeoutInSec>
    </ExpirySettings>
    <SkipCacheLookup/>
    <SkipCachePopulation/>
</ResponseCache>
```
The important elements are easy to track: most importantly, you can see the `CacheKey` element, where `KeyFragment` is setting what is saved as key for lookups in the future (in this instance the request uri). Then, the `ExpirySettings` determine how long cached info is saved (in mine, I chose to remove elements after 60 seconds). Putting one component at the pre-flow and one at the post will automatically check for the entry at the beginning and then stash it after it's been retrieved at the end.

### Route Rules/CORS Headers

An important difference between this project demo and the modules and labs is the way CORS headers are implemented. In the lab, Verify APIKey elements were placed in the proxy pre-flow variables and then a `DefaultFaultRule` at the beginning of the proxy flow with two elements: a step to set CORS response headers and a conditional step to set the status code to 200 if `request.verb equals "OPTIONS"`. This is bad practice -- every time the portal sends a request to the proxy with `OPTIONS` it will not include the apikey if placed in the header, then the `Verify-APIKey` step will fail every time and the only reason CORS headers will be returned correctly is because the program glosses over the fact that `Verify-APIKey` failed completely. The way it's implemented in this demo is much more easily expanded and also doesn't hack its way through errors. The key functionality is route rules:
```
<RouteRule name="GoNowhere">
    <Condition>request.verb=="OPTIONS"</Condition>
</RouteRule>
<RouteRule>
    <Condition>request.queryparam.test equals "second"</Condition>
    <TargetEndpoint>test-second</TargetEndpoint>
</RouteRule>
<RouteRule>
    <TargetEndpoint>default</TargetEndpoint>
</RouteRule>
```
These three route rules mean that in the case of an options call, the target is never called and CORS headers are hit without trying to hit any backend services, while the `request.queryparam.test equals "second"` just shows that it is possible to change the target endpoint from `https://api.openweathermap.org/data/2.5/weather` to `http://testarget.com/api` based on request parameters. This functionality I believe is extremely important -- it means that APIGEE could theoretically be used to build a lightweight experience layer api to call out to different services based on parameters passed to the single centralized proxy url.