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

### Route Rules