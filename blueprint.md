FORMAT: 1A
HOST: https://api.isthereanydeal.com

# IsThereAnyDeal.com API

This API provides programatic access to features found on [IsThereAnyDeal](https://isthereanydeal.com).

> In this API we will be refering to all apps, browser extensions or any other software as an *app*.

# Documentation source

You can find source file for this documentation over at [GitHub](https://github.com/tfedor/IsThereAnyDealApiDocs).
Feel free to help us improve it.

# Do's and don'ts

Whole API is provided in good faith. We trust you won't misuse the API to damage IsThereAnyDeal.com, its users or its reputation.

* You **may** use this API for commercial purposes **IF** the resulting app is available to public. If you want to use this API privately contact us.
* You **should** provide a link to IsThereAnyDeal.com or mention IsThereAnyDeal.com API.
* You **must not** make an impression that you are affiliated with IsThereAnyDeal.com, unless agreed otherwise. If you are interested in making an official app, please contact us.
* You **must not** change provided data in any way. This means that you can't remove affiliate tags from the URLs, change prices and so on. You **may** use only part of data provided and enrich them from different sources (e.g. you may add images to games).  
* You **must not** make an app that could be considered a competition to IsThereAnyDeal.com.
* You **must not** use this API to directly or indirectly help the competition of IsThereAnyDeal.com.

All data are provided *as-is*. We reserve the right to deny you access to the API at any point without notice. If you are not sure about any of the points, please contact us.

# Contact

* General API contact: [api@isthereanydeal.com](mailto:api@isthereanydeal.com)
* Bug reports: [issue tracker](https://bitbucket.org/tfedor/isthereanydeal.com/issues)

# Endpoints

All regular API endpoints will list current version, type and scope, if endpoint requires it.

## How to get keys

1. [Register your app](https://isthereanydeal.com/dev/app/)
2. Upon registration you will be able to manage your OAuth credentials and API keys.

> Both API keys and OAuth credentials uniquely identify your app
 
## Access levels

There are three levels of API endpoints that we provide:

* *public* - unrestricted access without any prerequisites
* *protected* - access is restricted by use of *API key* or *OAuth token*, depending on whether the API manipulates or reads user's data
* *private* - explicit permission is required, [contact us](#introduction/contact)

Misuse of any endpoint may lead to restricting all API access.

An endpoint may be of multiple types, e.g. `public/protected`. That means that you can get some basic info from
the endpoint but to get more detailed information you will need to use an API key or OAuth token.

## Versioning

All API endpoints are versioned. Version is denoted by letter `v` and number, like `v01` for version 1. Zeroes are optional and it doesn't matter whether you use `v1` or `v0001`. We prefer two digit format.

Non-breaking changes may be made without changing the version number. If possible, always use the latest version.

Older versions may become:
* *deprecated* - there is a newer version of the endpoint or a replacement; you should switch as soon as possible
* *unsupported* - the endpoint is no longer supported and may be removed at any time

Before being removed, endpoint will go through deprecated and unsupported phase. No guarantees are made for the duration of these phases.

 
## Debugging

For all endpoints that return json you may use `?pretty` parameter to get prettified output.

## Regions

Some endpoints may require you to specify *region*.
Accuracy of the response may be further improved by also supplying *country code*.
  
If you don't specify the country, *some* country from given region will be used.

## Common response fields 

A response may contain one or more fields beginning by dot `.`. These are not part of a data request but instead may contain
information about the endpoint or the specific response.

* `.deprecated` denotes that the endpoint is deprecated. You should always check for the existence of this field
* `.unsupported` denotes that the endpoint is unsupported. You should always check for the existence of this field
* `.meta` field describes the output, e.g. if the request was loaded from cache

# Group OAuth authentication

OAuth authentication is required for protected and private endpoints that manipulate users' data. We use OAuth 2.
A lot has been written about OAuth, but in a short and simplified way it works something like this:

1. Your app will redirect user to an authorization page with a defined scope and grant type
2. User allows access for your app
3. Request is redirected back to a `redirect URI` with an *authorization code* which you will use to get token or with *token* itself, depending on the grant type you have used.
4. With **token** acquired you may finally access API endpoints 

## Scope

OAuth requests requires you to define `scope` when you ask a user to authorize the app.
Scope tells the user what permissions are you requesting from them.

> You should always define the minimal scope your app will really need

## Grant types

Currently we allow following grant types:

* Implicit
* Authorization code
* Refresh token

For more information about grant types take a look at e.g. [A guide to OAuth 2.0 grants by Alex Bilbie](http://alexbilbie.com/2013/02/a-guide-to-oauth-2-grants/).  
            
## Token lifetime

Unlike API keys, OAuth tokens will **expire**. Following table lists for how long the tokens are valid.

| Key                | Use      | Lifetime       |
|--------------------|:--------:|:--------------:|
| Authorization code | once     | 1 minute       |
| Access token       | multiple | 1 year         |
| Refresh token      | once     | 1 year         |

## Authorize [/oauth/authorize/]

### Authorize app [GET /oauth/authorize/{?response_type,client_id,state,scope,redirect_uri}]

Ask user to authorize your app.

Redirect user to this URL and let him decide whether he will authorize your app.
After user's action, the response will be sent back to `redirect_uri` you set for your app or you sent as a parameter.

Response depends on the grant type you are using.
If you are using `implicit` grant type you will receive token in a URL fragment,
with `authorization code` grant type you will get one-time use code to request token.

**Always** define **minimal** scope your app will really need. If you're not using endpoints to user's Collection data,
there's no point to request Collection scopes. If your app grows in the future you can let user re-authorize your app.

*See also:* [some reading about `state` parameter](http://www.thread-safe.com/2014/05/the-correct-use-of-state-parameter-in.html)

+ Parameters
    + client_id (required) - OAuth client ID
    + response_type: `code` (enum[string], required)
        + Members
            + `code` - authorization code
            + `token` - token as part of URL fragment (implicit grant type)
    + state (required) - State for CSRF check
    + scope: `wait_read wait_write` (required) - Space separated list of scopes your app wants to access
    + redirect_uri (optional) - URL redirect where the response should be sent after user's action
        Must match `redirect_uri` you set up for your app.
        If you have listed multiple `redirect_uri`'s for your app then this parameter is **required**.
    
+ Response 200
    
    If user authorizes your app response will be sent back to `redirect_uri`.
    When using `authorization_code` grant the request will include a `code` and `state` parameter.
    In case of `implicit` grant, the request will contain token data in an URL fragment.
    
    + Body
    
+ Response 400
    
    Authorization server returns 400 if there's an error in `client ID` or `redirect URI`   
    
    + Body
    
            {
                "error": "invalid_client",
                "error_description": "The client id supplied is invalid"
            }
    
+ Response 302
    
    If there was an error in other parameters, request will be taken back to `redirect URI` with `error`, `error_description` and `state` parameters in URL.
    
    Please note that our implementation is stricter than the standard, since we always require you to specify `scope`
    
    + Body



## Token [/oauth/token/]

There are 2 types of tokens: 

* *access tokens*, used to access API endpoints
* *refresh tokens*, used to generate new access token

### Request access token [POST]

+ Request (application/x-www-form-urlencoded)

    | Attribute     | Required | Value | Description |
    |---------------|:--------:|-------|-------------|
    | grant_type    | &#10003; | `authorization_code` | |
    | code          | &#10003; | | Authorization code generated after authorize request |
    | redirect_uri  |          | | **Required** only if authorization request was issued with `redirect_uri` parameter. In such case values must be identical. |
    | client_id     | &#10003; | | OAuth client ID |
    | client_secret | &#10003; | | OAuth client secret |

    + Body 
    
            grant_type=authorization_code&code={code}&client_id={client_id}&client_secret={client_secret}

+ Response 200
       
    + Body
    
            {
                "access_token": "c629cdcefbe459393ac1337bc2282e91880c0b0c",
                "expires_in": 3600,
                "refresh_token": "ae6ef5b68693368013e0333b499555c987c577a5",
                "scope": "wait_read wait_write",
                "token_type": "Bearer"
            }
            
+ Response 400

    + Body
    
            {
                "error": "invalid_grant",
                "error_description": "Authorization code doesn't exist or is invalid for the client"
            }            

### Request refresh token [POST]

+ Request (application/x-www-form-urlencoded)

    | Attribute     | Required | Value | Description |
    |---------------|:--------:|-------|-------------|
    | grant_type    | &#10003; | `refresh_token` | |
    | refresh_token | &#10003; | | Refresh token |
    | client_id     | &#10003; | | OAuth client ID |
    | client_secret | &#10003; | | OAuth client secret |

    + Body 
    
            grant_type=refresh_token&refresh_token={refresh_token}&client_id={client_id}&client_secret={client_secret}

+ Response 200

    + Body
            
            {
                "access_token": "c6f36addc6aec47496683d46f03d7f118511a209",
                "expires_in": 3600,
                "refresh_token": "fcd196bbd4aefd9cdd213e1033fc8c7603a96274",
                "scope": "wait_read wait_write",
                "token_type": "Bearer"
            }

# Group User

## User Info [/v01/user/info?access_token={access_token}]
```Version: v01 | Type: protected | Scope: user_info```

Get public info of authenticated user.

### Get user info [GET]


+ Parameters
    
    + access_token (string) - Access token received after OAuth authentication
    
    
+ Response 200

        {
            "data":
            {
                "username": "tomas3333"
            }
        }



# Group Web

## Regions [/v01/web/regions/]
```Version: v01 | Type: public```

Lists all covered regions with the country codes associated to the region and currency information.
Optionally also displays English country name.

### List covered regions [GET]

+ Parameters
    
    + optional (enum[string], optional) - Separate multiple values with comma
        + Members
            + `names`


+ Response 200

        {
            "data":
                {
                    "eu1":{"countries":["AL","AD","AT","BE","DK","FI","FR","DE","IE","LI","LU","MK","NL","SE","CH"],"currency":{"code":"EUR","html":"&euro;","sign":"\u20ac"}},
                    "eu2":{"countries":["BA","BG","HR","CY","CZ","GR","HU","IT","MT","MC","ME","NO","PL","PT","RO","SM","RS","SK","SI","ES","VA","EE","LV","LT"],"currency":{"code":"EUR","html":"&euro;","sign":"\u20ac"}},
                    "uk":{"countries":["GB"],"currency":{"code":"GBP","html":"&pound;","sign":"\u00a3"}},
                    "us":{"countries":["US","CA"],"currency":{"code":"USD","html":"$","sign":"$"}},
                    "br":{"countries":["BR"],"currency":{"code":"USD","html":"$","sign":"$"}},
                    "au":{"countries":["AU"],"currency":{"code":"USD","html":"$","sign":"$"}}
                }
        }

## Stores in region [/v02/web/stores/{?region,country,optional}]
```Version: v02 | Type: public```

Lists all covered stores for given *region*. Optionally also loads current *deals* and *catalog* game count
and unix timestamp of last update.

> Note that IDs of stores don't change even if the store is re-branded. This means that you can hardcode ID of the store
  in your app if you need to use them.

### List covered stores for given region [GET]

+ Parameters
    
    + region: `us` (required) - Region name as used by ITAD
    + country: `US` (optional) - Country code to improve accuracy
    + optional (enum[string], optional) - Separate multiple values with comma
        + Members
            + `deals`
            + `catalog`

+ Response 200

    + Body

            {
                ".meta":{"region":"us","country":"US"},
                "data":
                    [
                        {"id":"steam","title":"Steam","color":"#9ffc3a"},
                        {"id":"gamersgate","title":"GamersGate","color":"#fc5d5d"},
                        {"id":"gamesplanet","title":"GamesPlanet UK","color":"#f6a740"},
                        {"id":"greenmangaming","title":"GreenMan Gaming","color":"#21a930"},
                        {"id":"gog","title":"GOG","color":"#f16421"},
                        {"id":"dotemu","title":"DotEmu","color":"#f6931c"},
                        {"id":"amazonus","title":"Amazon","color":"#fcc588"},
                        {"id":"nuuvem","title":"Nuuvem","color":"#b5e0f4"}
                    ]
                }

## Covered Stores [/v01/web/stores/all/]
```Version: v01 | Type: public```

Similar to previous endpoint, this one lists stores covered by ITAD. Difference is that this endpoint does not consider
region and always returns all stores covered. Since this is global for whole ITAD, you can't list last update timestamp
or counts of games on the store. 

> Note that IDs of stores don't change even if the store is re-branded. This means that you can hardcode ID of the store
  in your app if you need to use them.

### List all covered stores [GET]

+ Response 200

    + Body

            {
                "data":
                    [
                        {"id":"steam","title":"Steam","color":"#9ffc3a"},
                        {"id":"gamersgate","title":"GamersGate","color":"#fc5d5d"},
                        {"id":"gamesplanet","title":"GamesPlanet UK","color":"#f6a740"},
                        {"id":"greenmangaming","title":"GreenMan Gaming","color":"#21a930"},
                        {"id":"gog","title":"GOG","color":"#f16421"},
                        {"id":"dotemu","title":"DotEmu","color":"#f6931c"},
                        {"id":"amazonus","title":"Amazon","color":"#fcc588"},
                        {"id":"nuuvem","title":"Nuuvem","color":"#b5e0f4"}
                    ]
            }

# Group Game

## Identifier [/v02/game/plain/{?key,shop,game_id,url,title,optional}]
```Version: v02 | Type: protected```

ITAD identifies games by *plain* (string). This endpoint will try to fetch a *plain* for you by store game ID, URL
or title. The more information you provide the better the chance to successfully fetch plain.

Identification is done in following order:
 
1. by ID
2. by URL
3. by title

The `.meta` part of the response includes `match` (tells you which part of the process succeded)
and `active` (true if we have pricing info for the game).

Optionally this endpoint can also return game's title.

### Identify by ID

To uniquely identify the game on different stores and map it to our `plain` we use *store/game ID* combination.
Ideally, game ID is the same the given store uses in their system. This method is the fastest and the most accurate.

Most of the time the ID the store uses to identify the game can be guessed from URL or found in page source. 
However, internally, IDs on ITAD may differ from those used by stores (either because they are not known or IDs
provided in feeds differ from what is shown in the store). If you are not sure what IDs we use for given store, you can
contact us for more information. We don't list it for each store since the situation may change.

Requires `shop` and `game_id` parameters.

### Identify by URL

The most unreliable method. Since URLs often change by adding various parameters
(e.g. region, country, campaign info) it's not the best idea to try to identify the game solely by URL.
 
Identification by URL will remove the host part of the URL and try to compare the rest with the end of the URL
in ITAD's database.

Requires `shop` and `url` parameters.

### Identify by Title

Identification by title tries to match provided title against our database.
Should provide reasonable results.

Requires `title` parameter.

### Get plain [GET]

+ Parameters
    
    + key (required) - Your API key
    + shop: `steam` (optional) - Required for ID and URL
    + game_id: `app/377160` (optional)
    + url (optional)
    + title (optional)
    + optional (enum[string], optional) - Separate multiple values with comma
        + Members
            + `title`

+ Response 200

        {".meta":{"match":"id","active":true},"data":{"plain":"dishonored"}}
        
+ Response 200

        {".meta":{"match":false,"active":false},"data":[]}
       

## Multiple plains by ID [/v01/game/plain/id/{?key,shop,ids}]
```Version: v01 | Type: protected```

If you need to get more `plain`s at once, you can use this endpoint.
Its limitation is that it works only with IDs and one shop at once.

> This endpoint will provide ONLY `plain` for given `id` (or `null` if not found), no additional info like title or whether the entry has active deals.

### Get plain [GET]

Instead of `ids` parameter you may also send JSON array in a `POST` body.

+ Parameters
    
    + key (required)
    + shop: `steam` (required)
    + ids: `app/377160,app/96100,sub/28187,sub/1245` (optional) - List of IDs separated by comma

+ Response 200

        {
            "data": {
                "app\/377160": "falloutiv",
                "app\/96100": "defygravity",
                "sub\/28187": "elderscrollsvskyrimlegendaryedition",
                "sub\/1245": null
            }
        }

## Get all plains [/v01/game/plain/list/{?key,shops}]
```Version: v01 | Type: protected```

Get list of all `plain`s currently known by ITAD.
You can uses `shops` parameter to restrict list to one or more shops.

> Result might be quite large, please do cache the list on your end

### Get all plains [GET]

+ Parameters
    
    + key (required)
    + shops: `discord` (required) List of shop IDs separated by comma

+ Response 200

        {
            "data":
            {
                "discord":
                {
                    "451550535720501248": "deadcells",
                    "466696214818193408": "pillarsofeternityiideadfire",
                    "469679019701567528": "omensight",
                    "471033395590201365": "spellforceiii",
                    "471033751535616000": "starbound",
                    "471376328319303681": "intobreach",
                    "471501020006973451": "moonlighter",
                    "471501184939851806": "frostpunk",
                    "472483394085715979": "bannersagaiii",
                    "474762737331732495": "hollowknight",
                    "474762950108905472": "thisispoliceii",
                    "486981988109254667": "kingofhat",
                    "487031053454802946": "atsundown",
                    "487272772393762826": "madmachines",
                    "488422292137836574": "badnorth",
                    "488607666231443456": "minionmasters",
                    "488811098242547739": "battlechasersnightwar",
                    "489184797936058380": "sinnersacrificeforredemption",
                    "489420695454089217": "descenders",
                    "489926636943441932": "subnautica",
                    "492418279717994505": "mastersofanima",
                    "494870847777931268": "forsakenremastered",
                    "498690780836134912": "crosscode",
                    "500428425362931713": "celeste",
                    "503982482664849408": "callofcthulhu",
                    "508008071411400724": "parkasaurus",
                    "508057374875975682": "visage",
                    "510199020782747732": "kynseed"
                }
            }
        }
        


## Get id/plain map [/v01/game/map/{?key,shop}]
```Version: v01 | Type: protected```

Get map of `plain -> shop id` or `shop id -> plain` for a selected store.

> Result might be quite large, please do cache the list on your end

### Get map [GET]

+ Parameters
    
    + key (required)
    + shop: `steam` (required) Shop ID
    + type (enum[string],optional)
        
            + Values
                + `plain:id`
                + `id:plain`

+ Response 200

        {
            "data": {
                "app\/1064990": [
                    "thunderpaw"
                ],
                "app\/1081000": [
                    "trovesweetsixteenpack"
                ],
                "app\/1256": [
                    "killingflooroutbreakcharacterpack"
                ],
                "app\/202970": [
                    "callofdutyblackopsii"
                ],
                "app\/204306": [
                    "awesomenautsofficerlonestar"
                ],
                "app\/208541": [
                    "dungeondefendersseriesevherodlc"
                ],
                "app\/208670": [
                    "bladesoftime"
                ],
            
                ...    
            }
        }
       

## Prices [/v01/game/prices/{?key,plains,region,country,shops,exclude,added}]
```Version: v01 | Type: protected ```

Get all current prices for one or more selected games. Use `region` and `country` to get more accurate results.

> Prices will be sorted from lowest to highest.

### Get current prices [GET]

+ Parameters
    + key (required) - Your API key
    + plains: `endorlight,storyofsurvivor` (required)
    + region: `eu2` (optional)
    + country: `SK` (optional)
    + shops: `steam,indiegamestand,amazonus` (optional) - list of store ids to include in results, separated by comma
    + exclude: `voidu,itchio` (optional) - list of store ids to exclude from results, separated by comma
    + added: 0 (optional) - timestamp, only get prices added since this time

+ Response 200
        
        {
            ".meta": {
                "currency": "EUR"
            },
            "data": {
                "endorlight": {
                    "list": [
                        {
                            "price_new": 0.45,
                            "price_old": 2.35,
                            "price_cut": 81,
                            "url": "https:\/\/indiegamestand.com\/store\/2420\/endorlight",
                            "shop": {
                                "id": "indiegamestand",
                                "name": "IndieGameStand"
                            },
                            "drm": [
                                "steam"
                            ]
                        },
                        {
                            "price_new": 2.99,
                            "price_old": 2.99,
                            "price_cut": 0,
                            "url": "http:\/\/store.steampowered.com\/app\/428430\/?snr=1_7_7_204_150_94",
                            "shop": {
                                "id": "steam",
                                "name": "Steam"
                            },
                            "drm": [
                                "steam"
                            ]
                        }
                    ],
                    "urls": {
                        "game": "https:\/\/isthereanydeal.com\/#\/page:game\/info?plain=endorlight"
                    }
                },
                "storyofsurvivor": {
                    "list": [
                        {
                            "price_new": 3.53,
                            "price_old": 5.29,
                            "price_cut": 33,
                            "url": "https:\/\/indiegamestand.com\/store\/2385\/story-of-the-survivor",
                            "shop": {
                                "id": "indiegamestand",
                                "name": "IndieGameStand"
                            },
                            "drm": [
                                "DRM Free",
                                "steam"
                            ]
                        },
                        {
                            "price_new": 4.99,
                            "price_old": 4.99,
                            "price_cut": 0,
                            "url": "http:\/\/store.steampowered.com\/app\/440950\/?snr=1_7_7_230_150_495",
                            "shop": {
                                "id": "steam",
                                "name": "Steam"
                            },
                            "drm": [
                                "steam"
                            ]
                        },
                        {
                            "price_new": 5.99,
                            "price_old": 5.99,
                            "price_cut": 0,
                            "url": "https:\/\/www.bundlestars.com\/en\/game\/story-of-the-survivor",
                            "shop": {
                                "id": "bundlestars",
                                "name": "Bundle Stars"
                            },
                            "drm": [
                                "steam"
                            ]
                        }
                    ],
                    "urls": {
                        "game": "https:\/\/isthereanydeal.com\/#\/page:game\/info?plain=storyofsurvivor"
                    }
                }
            }
        }


## Historical low [/v01/game/lowest/{?key,plains,region,country,shops,exclude,since}]
```Version: v01 | Type: protected ```

Get historically lowest price for one or more games.

### Get historical low [GET]

+ Parameters
    + key (required) - Your API key
    + plains: `europauniversalisiv,falloutiv` (required) - List of plains separated by comma
    + region: `eu2` (optional)
    + country: `SK` (optional)
    + shops: `nuuvem,funstock,steam` (optional) - list of store ids to include in results, separated by comma
    + exclude: `amazonus,fanatical` (optional) - list of store ids to exclude from results, separated by comma
    + since: `0` (optional, unsigned) - timestamp, only get history records since this timestamp
    + until (optional, unsigned) - timestamp, only get history records until this timestamp
    + new: `1`|`0` (optional) - if true (`1`), only get deals that started after specified `since`
    
+ Response 200
        
        {
            ".meta": {
                "currency": "EUR"
            },
            "data": {
                "europauniversalisiv": {
                    "shop": {
                        "id": "nuuvem",
                        "name": "Nuuvem"
                    },
                    "price": 4.26,
                    "cut": 80,
                    "added": 1419894191,
                    "urls": {
                        "game": "https:\/\/isthereanydeal.com\/#\/page:game\/info?plain=europauniversalisiv",
                        "history": "https:\/\/isthereanydeal.com\/#\/page:game\/price?plain=europauniversalisiv"
                    }
                },
                "falloutiv": {
                    "shop": {
                        "id": "funstock",
                        "name": "FunStock Digital"
                    },
                    "price": 27.24,
                    "cut": 50,
                    "added": 1450883648,
                    "urls": {
                        "game": "https:\/\/isthereanydeal.com\/#\/page:game\/info?plain=falloutiv",
                        "history": "https:\/\/isthereanydeal.com\/#\/page:game\/price?plain=falloutiv"
                    }
                }
            }
        }


## Store low [/v01/game/storelow/{?key,plains,region,country,shops,exclude}]
```Version: v01 | Type: protected ```

Get historically lowest price for one or more games at individual stores. Max 5 games at once.

### Get store low [GET]

+ Parameters
    + key (required) - Your API key
    + plains: `europauniversalisiv,falloutiv` (required) - List of plains separated by comma
    + region: `eu2` (optional)
    + country: `SK` (optional)
    + shops: `gog,greenmangaming,steam` (optional) - list of store ids to include in results, separated by comma
    + exclude: `amazonus,fanatical` (optional) - list of store ids to exclude from results, separated by comma
    
+ Response 200
        
       {
         ".meta": {
           "currency": "EUR"
         },
         "data": {
           "europauniversalisiv": [
             {
               "shop": "greenmangaming",
               "price": 7.99
             },
             {
               "shop": "steam",
               "price": 7.99
             }
           ],
           "falloutiv": [
             {
               "shop": "greenmangaming",
               "price": 11.28
             },
             {
               "shop": "steam",
               "price": 7.49
             }
           ]
         }
       }


## Bundles [/v01/game/bundles/{?key,plains,limit,expired,sort,region}]
```Version: v01 | Type: protected ```

Provides information about how many times the game has been bundled and lists these bundles.

> If expiry of the bundle is unknown, `null` will be returned instead of timestamp

### Get info about where the game was bundled [GET]

+ Parameters
    + key (required) - Your API key
    + plains: `endorlight,storyofsurvivor` (required) - List of plains separated by comma
    + limit: `1` (int,optional) - How many bundles to list at most, `-1` for no limit
    + expired: `0` (1/0,optional) - Whether to list expired bundles
    + sort (enum[string],optional) - Sorting order of listed bundles
    
        + Values
            + `expiry`
            + `recent`
            
    + region: `us` (optional)
                
+ Response 200
        
        {
            "data": {
                "endorlight": {
                    "total": 4,
                    "list": [
                        {
                            "title": "Slash & Shoot Bundle",
                            "bundle": "One More Bundle",
                            "start": 1457895360,
                            "expiry": 1458658740,
                            "platforms": [
                                "windows"
                            ],
                            "url": "https:\/\/isthereanydeal.com/specials\/3172\/"
                        }
                    ],
                    "urls": {
                        "game": "https:\/\/isthereanydeal.com\/#\/page:game\/info?plain=endorlight",
                        "bundles": "https:\/\/isthereanydeal.com\/specials\/#\/search:endorlight;\/filter:bundle"
                    }
                },
                "storyofsurvivor": {
                    "total": 3,
                    "list": [
                        {
                            "title": "Every Monday Bundle 102",
                            "bundle": "Indie Gala",
                            "start": 1457974140,
                            "expiry": null,
                            "platforms": [
                                "windows"
                            ],
                            "url": "https:\/\/isthereanydeal.com/specials\/3176\/"
                        }
                    ],
                    "urls": {
                        "game": "https:\/\/isthereanydeal.com\#\/page:game\/info?plain=storyofsurvivor",
                        "bundles": "https:\/\/isthereanydeal.com\specials\/#\/search:storyofsurvivor;\/filter:bundle"
                    }
                }
            }
        }


## Info [/v01/game/info/{?key,plains}]
```Version: v01 | Type: protected ```

Get basic information about game.

> Since Greenlight no longer exists on Steam, `greenlight` field will be always null 

With `optional=metacritic` parameter you can also get summary for this game from Metacritic, if we found one. However,
you **have to** mention that this text is authored by Metacritic and you should link to Metacritic page - you can find
url in `urls` field. If Metacritic info is not available, it will be `null`.

### Get info about game [GET]

+ Parameters
    + key (required) - Your API key
    + plains: `europauniversalisiv,linelight` (required) - List of plains separated by comma
    + optional (enum[string], optional) - Separate multiple values with comma
        + Members
            + `metacritic`

+ Response 200
        
        {
            "data": {
                "europauniversalisiv": {
                    "title": "Europa Universalis IV",
                    "greenlight": null,
                    "is_package": false,
                    "is_dlc": false,
                    "achievements": true,
                    "trading_cards": true,
                    "early_access": false,
                    "reviews": {
                        "steam": {
                            "perc_positive": 92,
                            "total": 15712,
                            "text": "Very Positive"
                        }
                    },
                    "urls": {
                        "game": "https:\/\/isthereanydeal.com\/#\/page:game\/info?plain=europauniversalisiv",
                        "package": "https:\/\/isthereanydeal.com\/#\/page:game\/package?plain=europauniversalisiv",
                        "dlc": "https:\/\/isthereanydeal.com\/#\/page:game\/dlc?plain=europauniversalisiv"
                    }
                },
                "linelight": {
                    "title": "Linelight",
                    "greenlight": {
                        "status": "greenlit",
                        "url": "http:\/\/steamcommunity.com\/sharedfiles\/filedetails\/?id=637687580"
                    },
                    "is_package": false,
                    "is_dlc": false,
                    "achievements": false,
                    "trading_cards": false,
                    "early_access": false,
                    "reviews": null,
                    "urls": {
                        "game": "https:\/\/isthereanydeal.com\/#\/page:game\/info?plain=linelight",
                        "package": "https:\/\/isthereanydeal.com\/#\/page:game\/package?plain=linelight",
                        "dlc": "https:\/\/isthereanydeal.com\/#\/page:game\/dlc?plain=linelight"
                    }
                }
            }
        }


## Overview [/v01/game/overview/{?key,region,country,plains,shop,ids,allowed,optional}]
```Version: v01 | Type: protected ```

Originally made for Enhanced Steam addon, this endpoint returns basic information about game's prices:
best current price, historical lowest price, and bundles in which the game is included.

You can use `region` and/or `country` to specify country, restrict stores with `allowed` parameter, and optionally
request prices in local currency or with vouchers applied.

To request games, you can use either `plains` or `shop`/`ids`, where you specify store for which you will supply store
specific ids - this will be most commonly used for steam.

### Get game price overview [GET]

+ Parameters
    + key (required) - Your API key
    + region: `us` (string, optional)
    + country: `US` (string, optional)
    + plains (string, optional) - List of plains, separated by commas
    + shop: `steam` (string, optional)
    + ids: `app/460930,sub/37125,bundle/7078` (string, optional)
    + allowed: `steam,gog` (string, optional) - List of allowed stores, separated by commas
    + optional (enum[string], optional) - Separate multiple values with comma
        + Members
            + `voucher`
            + `local`

+ Response 200
        
        {
          ".meta": {
            "region": "us",
            "country": "US",
            "currency": "USD"
          },
          "data": {
            "app/460930": {
              "price": {
                "store": "Steam",
                "cut": 65,
                "price": 17.49,
                "price_formatted": "$17.49",
                "url": "https://store.steampowered.com/app/460930/",
                "drm": [
                  "Denuvo Anti-tamper",
                  "steam",
                  "Other DRM",
                  "uplay"
                ]
              },
              "lowest": {
                "store": "Steam",
                "cut": 65,
                "price": 17.49,
                "price_formatted": "$17.49",
                "url": "https://store.steampowered.com/app/460930/",
                "recorded": 1544469391,
                "recorded_formatted": "18 days ago"
              },
              "bundles": {
                "count": 0,
                "live": []
              },
              "urls": {
                "info": "https://isthereanydeal.com/game/tomclancysghostreconwildlands/info/",
                "history": "https://isthereanydeal.com/game/tomclancysghostreconwildlands/history/",
                "bundles": "https://isthereanydeal.com/specials/#/filter:search/tomclancysghostreconwildlands"
              }
            },
            "sub/37125": {
              "price": {
                "store": "Steam",
                "cut": 80,
                "price": 1.99,
                "price_formatted": "$1.99",
                "url": "https://store.steampowered.com/app/222880/",
                "drm": [
                  "steam"
                ]
              },
              "lowest": null,
              "bundles": {
                "count": 11,
                "live": [
                  {
                    "title": "Batman: Arkham Asylum Game of the Year Edition",
                    "url": "https://www.greenmangaming.com/mix-n-match/",
                    "expiry": null,
                    "expiry_rfc": null,
                    "type": "bundle",
                    "page": "GreenMan Gaming",
                    "details": "https://isthereanydeal.com/specials/#/options:pending;/filter:id/6747",
                    "tiers": [
                      {
                        "price": 5,
                        "price_formatted": "$5.00",
                        "fixed": 1,
                        "note": "",
                        "games": [
                          "Batman: Arkham Asylum Game of the Year Edition",
                          "Mount & Blade: Warband",
                          "Super Cloudbuilt ",
                          "F.E.A.R. 3",
                          "Magicka",
                          "Adrift",
                          "Dead Age",
                          "RAGE",
                          "War Tech Fighters",
                          "Killing Floor Complete Pack",
                          "The Black Death",
                          "Peregrin",
                          "Toybox Turbos",
                          "InnerSpace",
                          "Beat Cop",
                          "Mount & Blade: With Fire & Sword",
                          "This War of Mine",
                          "Frozen Synapse Prime",
                          "Mount & Blade",
                          "Ballistic Overkill",
                          "Stable Orbit",
                          "Fahrenheit: Indigo Prophecy Remastered",
                          "LEGO® Batman: The Videogame",
                          "Crusader Kings II",
                          "Quarantine",
                          "Warhammer: End Times - Vermintide",
                          "INSURGENCY",
                          "Beyond Eyes",
                          "Next Up Hero",
                          "Tower 57",
                          "Injustice: Gods Among Us - Ultimate Edition"
                        ]
                      }
                    ]
                  }
                ]
              },
              "urls": {
                "info": "https://isthereanydeal.com/game/insurgency/info/",
                "history": "https://isthereanydeal.com/game/insurgency/history/",
                "bundles": "https://isthereanydeal.com/specials/#/filter:search/insurgency"
              }
            },
            "bundle/7078": {
              "price": {
                "store": "Steam",
                "cut": 87,
                "price": 12.58,
                "price_formatted": "$12.58",
                "url": "https://store.steampowered.com/bundle/7078/",
                "drm": []
              },
              "lowest": null,
              "bundles": {
                "count": 0,
                "live": []
              },
              "urls": {
                "info": "https://isthereanydeal.com/game/gamenesissuperpack/info/",
                "history": "https://isthereanydeal.com/game/gamenesissuperpack/history/",
                "bundles": "https://isthereanydeal.com/specials/#/filter:search/gamenesissuperpack"
              }
            }
          }
        }


# Group Search

## Search [/v02/search/search/{?key,q,limit,strict}]
```Version: v02 | Type: protected ```

Search for games.

### Find games [GET]

+ Parameters
    + key (required) - Your API key
    + q: `assassins creed odyssey` (required) - Search query
    + limit: 20 (optional)
    + strict: `0` (optional) - 0 or 1; with strict mode enabled we are disabling search query fuzziness  

+ Response 200


        {
          "data": {
            "results": [
              {
                "id": 40805,
                "plain": "assassinscreedodyssey",
                "title": "Assassin's Creed Odyssey"
              },
              {
                "id": 93797,
                "plain": "assassinscreedodysseygoldedition",
                "title": "Assassin's Creed Odyssey Gold Edition"
              },
              {
                "id": 141318,
                "plain": "assassinscreedodysseyseasonpass",
                "title": "Assassin's Creed Odyssey Season Pass"
              },
              
              ... rest of response omitted for clarity ...
            ],
            "urls": {
              "search": "https:\/\/isthereanydeal.com\/search\/?q=assassins%20creed%20odyssey"
            }
          }
        }


# Group Deals

## Deals List [/v01/deals/list/{?key,offset,limit,region,country,shops}]
```Version: v01 | Type: protected ```

Provides list of deals.

> Use region and country parameters to get more accurate listing

### Get deals [GET]

+ Parameters
    + key (required) - Your API key
    + offset: 0 (optional)
    + limit: 20 (optional)
    + region: `eu2` (optional)
    + country: `CZ` (optional)
    + shops: `steam,gog` (optional) - List of store ids separated by comma
    + sort: `price:asc` (optional) - Sort parameter, key and direction, separated by colon.
                                     Possible keys are: `time` (default), `price`, `cut`, `expiry`

+ Response 200
        
        {
            ".meta": {
                "currency": "EUR"
            },
            "data": {
                "count": 7984,
                "list": [
                    {
                        "plain": "theatreofwariiafricaiixiviii",
                        "title": "Theatre of War 2 Africa 1943",
                        "price_new": 7.98,
                        "price_old": 17.74,
                        "price_cut": 55,
                        "added": 1458442019,
                        "expiry": 1571068800,
                        "shop": {
                            "id": "amazonus",
                            "name": "Amazon"
                        },
                        "drm": [
                            ""
                        ],
                        "urls": {
                            "buy": "http:\/\/www.amazon.com\/Theatre-War-Africa-1943-Download\/dp\/B004QO9UUM\/ref=sr_1_4163_twi_sof_1\/185-4887647-7992563?s=videogames-download&ie=UTF8&qid=1458442018&sr=1-4163&refinements=p_n_availability%3A1238047011%2Cp_n_operating_system_browse-bin%3A2346238011%2Cp_n_condition-type%3A2224366011%2Cp_36%3A-700&tag=isthcom0a-20",
                            "game": "https:\/\/isthereanydeal.com\/#\/page:game\/info?plain=theatreofwariiafricaiixiviii"
                        }
                    },
                    {
                        "plain": "mightandmagicheroesviideluxeedition",
                        "title": "Might & Magic Heroes VII Deluxe Edition",
                        "price_new": 56.99,
                        "price_old": 59.99,
                        "price_cut": 5,
                        "added": 1458441819,
                        "expiry": 0,
                        "shop": {
                            "id": "dlgamer",
                            "name": "DLGamer"
                        },
                        "drm": [
                            "uplay"
                        ],
                        "urls": {
                            "buy": "http:\/\/www.dlgamer.eu\/download-might-magic-heroes-7-deluxe--pc_games-p-29359.html?affil=3429886050",
                            "game": "https:\/\/isthereanydeal.com\/#\/page:game\/info?plain=mightandmagicheroesviideluxeedition"
                        }
                    },
                    
                    ... rest of response omitted for clarity ...
                    
                ],
                "urls": {
                    "deals": "https:\/\/isthereanydeal.com\/"
                }
            }
        }


# Group Waitlist

## Waitlist Check [/v01/user/wait/{?access_token,plain}]
```Version: v01 | Type: protected | Scope: wait_read```

Check whether user has a game in Waitlist. Response will be `yes` or `no`.

### Check if user has game in Waitlist [GET]

+ Parameters
    + access_token (required) - OAuth access token
    + plain: `dishonored` (required)
        
+ Response 200
        
        {"data": {"in_waitlist": "no"}}

## Waitlist [/v01/user/wait/all/{?access_token,shop,optional}]
```Version: v01 | Type: protected | Scope: wait_read```

Get list of games that the user has in Waitlist.

### Get user's Waitlist [GET]

+ Parameters
    + access_token (required) - OAuth access token
    + shop (optional) - Only used with optional `gameid`
    + optional (enum[string], optional)
        + Members:
            + `title` - default if no optional is specified
            + `gameid`
    
    
+ Response 200

        {
            "data": {
                "falloutnewvegas": {
                    "title": "Fallout: New Vegas"
                },
                "racesun": {
                    "title": "Race the Sun"
                },
                "vessel": {
                    "title": "Vessel"
                }
            }
        }


## Import via Form [/waitlist/import/]
```Latest data version: 02 | Type: public | Does not use API server```

It is possible to export Waitlist in a JSON format and then import it back to ITAD.
You can use the same import process to let the user import data from a 3rd party source.
User will be presented a form where he will be able to choose which games he wishes to import.

To get a live example of the data you may [export your Waitlist](https://isthereanydeal.com/waitlist/#/page:export/waitlist),
or check out examples.

> **When using this method of import you will send POST requests directly to IsThereAnyDeal, not to the API**

### JSON Format

Supplied JSON has to contain two fields:
 
* `version` Version string
* `data` Array of objects

Each object in `data` array has to contain one of: `title`, `plain`, or `gameid` (a [shop, id] pair).

#### Minimal Example

```json
{
    "version": "02",
    "data": [
            {"gameid": ["steam", "app/440"]},
            {"title": "Oxygen Not Included"},
            {"plain": "leftivdead"}
    ]
}
```

#### More Complete Example

```json
{
    "version": "02",
    "data": [
        {
            "plain": "oxygennotincluded",
            "title": "Oxygen Not Included",
            "cat":
            {
                "id": 13,
                "title": "Christmas Wishlist"
            },
            "shop": null,
            "price_limit": "7.99",
            "cut_limit": 25,
            "drm": ["steam", "drmfree"],
            "added": 1505868698
        }
    ]
}
```

Notes:
* Unused fields may be omitted or set to null, for omitted fields user's default settings will be used

### Waitlist Categories

You may set [Waitlist Category](https://isthereanydeal.com/settings/waitlist/categories/) either by `id`, if you know it,
or by `title` (or both).
If the category is set, an attempt will be made to match it with an existing one, but if none is found and you set `title`,
it will be created.

### Send user to Waitlist import form [POST]
 
+ Request

    The body of the request will simulate form submission. It has to contain two fields: `file` and `upload`.
    The content of `file` is `base64` encoded JSON string, the content of `upload` field is not important.

    The URL for request is `https://isthereanydeal.com/waitlist/import/`

    Example of minimal POST data from example:

    + Body  
        
            file=ew0KICAgICJ2ZXJzaW9uIjogIjAyIiwNCiAgICAiZGF0YSI6IFsNCiAgICAgICAgeyJ0aXRsZSI6ICJPeHlnZW4gTm90IEluY2x1ZGVkIn0NCiAgICBdDQp9&upload=Import+Waitlist
        
## Direct Import [/v01/waitlist/import/{?access_token}]
```Version: v01 | Type: protected | Scope: wait_write```

Alternative to import via form is to directly import data via API. This doesn't give user
as much control over the import but will be simpler. Use the flow that fits your app the best.

### Import Waitlist [POST]

+ Parameters

    + access_token (required) - OAuth access token

+ Request

    Data format for import is exactly the same as for import via Form,
    **the only difference is that body of the request is JSON string that is NOT `base64` encoded**.
    
    + Body
           
        {
            "version": "02",
            "data": [
                {
                    "plain": "oxygennotincluded",
                    "title": "Oxygen Not Included",
                    "cat":
                    {
                        "id": 13,
                        "title": "Christmas Wishlist"
                    },
                    "shop": null,
                    "price_limit": "7.99",
                    "cut_limit": 25,
                    "drm": ["steam", "drmfree"],
                    "added": 1505868698
                }
            ]
        }
        
+ Response 200

    Response will contain number of games that were added in `.meta` field.
    It is possible for request to be successful and still import 0 games,
    because games that are already in Waitlist are ignored during import  

    + Body
    
        {
            ".meta": {
                "games": 1
            }
        }
        
        


## Waitlist Remove [/v02/user/wait/remove/{?access_token,plain}]
```Version: v02 | Type: protected | Scope: wait_write```

Delete games from user's Waitlist (from all categories!).
You can either use list of plains, or ids from specific store.

### Delete games from user's Waitlist [DELETE]

+ Parameters
    + access_token (required) - OAuth access token
    + plains (string, optional) - List of plains, separated by commas
    + shop: `steam` (string, optional)
    + ids: `app/440,bundle/7078` (string, optional)
    
        
+ Response 200
        
        { 
            "data": {
                "response": "success",
                "plains": {
                    "app\/440": "teamfortressii",
                    "bundle\/7078": "gamenesissuperpack"
                }
            }
        }

        
# Group Collection

## Collection Check [/v01/user/coll/{?access_token,plain,optional}]
```Version: v01 | Type: protected | Scope: coll_read```

Check whether user has the game in Collection. Response will be `yes` or `no`.

You can supply `optional=stores` to get info about stores at which the user owns the game.
List of stores will be added to output, each store having `id` and `title`, which are the same as stores ITAD is
covering with addition of `other` and `retail`. If the `id` is numeric, it's the user's custom value.

### Check if user has game in Collection [GET]

+ Parameters
    + access_token (required) - OAuth access token
    + plain: `dishonored` (required)
    + optional: `stores` (enum[string], optional) - Separate multiple values with comma
        + Members
            + `stores`

+ Response 200

        {
            "data": {
                "in_collection": "yes",
                "stores": [
                    {
                        "id": "greenmangaming",
                        "name": "Greenmangaming"
                    },
                    {
                        "id": "steam",
                        "name": "Steam"
                    }
                ]
            }
        }
        
        
## Collection v02 [/v02/user/coll/all/{?access_token,shop,short,optional}]
```Version: v02 | Type: protected | Scope: coll_read```

Get complete user's Collection.

> Requesting full Collection, saving it for some time and comparing games to the saved list is often prefered than doing
Collection request for each game individually.

This endpoint has no formal required parameter, but you must list at least one of `plain`, `title`, `gameid` in parameter `optional`.
If you use `gameid`, you also need to specify `shop` (an ITAD shop ID), which will return games in user's Collection with
game ID at specificied store. 

You can also supply `copy_type` to get list of copy types that user owns - e.g. Steam copy, GOG copy etc.
Copy types will be returned as list of IDs and `typemap` will be included in a result, as a map of id to name.

If you only require one identifying parameter (`plain`, `title`, `gameid`) and do not require `copy_type`,
the result will be simple list, instead of list of objects. You can force list of objects with `short=0`.

### Get user's Collection [GET]

+ Parameters
    + access_token (required) - OAuth access token
    + shop (optional) - Used in conjunction with `optional=gameid` to specify which IDs should be returned 
    + short: `1` (optional) - Force result to be a list of objects
    + optional: `plain,title,copy_type` (enum[string], optional) - Separate multiple values with comma
        + Members
            + `plain`
            + `title`
            + `gameid`
            + `copy_type`

+ Request

+ Response 200

        {
          "data": {
            "games": [
              {
                "plain": "thatbastardistryingtostealourgold",
                "title": "! That Bastard Is Trying To Steal Our Gold !"
              },
              {
                "plain": "iii0viiixblockactionrpg",
                "title": "3079 -- Block Action RPG"
              },
              {
                "plain": "aiwarfleetcommand",
                "title": "AI War: Fleet Command"
              }
            ]
          }
        }


+ Request

+ Response 200

        {
          "data": {
            "games": [
              "app\/449940",
              "app\/259620",
              "app\/40400",
              "app\/3730",
            ]
          }
        }
        
+ Request

+ Response 200

        {
          "data": {
            "games": [
              {
                "gameid": "app\/952950",
                "types": [
                  "steam"
                ]
              },
              {
                "gameid": "app\/278440",
                "types": [
                  "steam"
                ]
              },
              {
                "gameid": "app\/15560",
                "types": [
                  "steam"
                ]
              }
              ...
            ],
            "typemap": {
              "steam": "Steam",
              "gog": "GOG",
              "bundlestars": "Bundle Stars",
              "other": "Other",
              "11": "Klei Account",
              "macgamestore": "MacGameStore"
            }
          }
        }

        
## Collection v01 [/v01/user/coll/all/{?access_token,optional}]
```Version: v01 | Type: protected | Scope: coll_read```

> **This endpoint is deprecated, please use `v02`**

Get user's Collection. Result is the list of collected games.

> Requesting full Collection, saving it for some time and comparing games to the saved list is often prefered than doing
Collection request for each game individually.

You can supply `optional=stores` to get info about stores at which the user owns the game.
Same as in *Collection / Single Game* endpoint. 

### Get user's Collection [GET]

+ Parameters
    + access_token (required) - OAuth access token
    + optional: `stores` (enum[string], optional) - Separate multiple values with comma
        + Members
            + `stores`

+ Response 200

        {
            "data": {
                "falloutnewvegas": {
                    "title": "Fallout: New Vegas"
                },
                "racesun": {
                    "title": "Race the Sun"
                },
                "vessel": {
                    "title": "Vessel"
                }
            }
        }

## Import via Form [/collection/import/]
```Latest data version: 02 | Type: public | Does not use API server```

It is possible to export Collection in a JSON format and then import it back to ITAD.
You can use the same import process to let the user import data from a 3rd party source.
User will be presented a form where he will be able to choose which games he wishes to import.

To get a live example of the data you may [export your Collection](https://isthereanydeal.com/collection/#/page:export/collection),
or check out examples.

> **When using this method of import you will send POST requests directly to IsThereAnyDeal, not to the API**

### JSON Format

Supplied JSON has to contain two fields:
 
* `version` Version string
* `data` Array of objects

Each object in `data` array has to contain identifier (one of: `title`, `plain`, or `gameid` (a [shop, id] pair)) and `copies` array, while each copy has at the very least contain `type`.
Type may be ID of store or any string which will create [custom value](https://isthereanydeal.com/settings/collection/lists/)
if it doesn't yet exist.

#### Minimal Example

```json
{
    "version": "02",
    "data": [
      {"gameid": ["steam", "app/440"], "copies": [{"type": "steam"}]},
      {"title": "Oxygen Not Included", "copies": [{"type": "steam"}]},
      {"plain": "leftivdead", "copies": [{"type": "steam"}]}
    ]
}
```

#### More Complete Example

```json
{
    "version": "02",
    "data": [
        {
            "plain": "0rbitalis",
            "title": "0RBITALIS",
            "group": null,
            "note": null,
            "status": "notplayed",
            "user_tags": [],
            "playtime": 0,
            "copies": [
            {
                "type": "steam",
                "platforms": ["windows", "mac"],
                "status": "redeemed",
                "price": null,
                "currency": "EUR",
                "note": null,
                "added": 1506896003,
                "owned": 0,
                "source": null
            },
            {
                "type": "Groupees.com",
                "platforms": [],
                "status": "redeemed",
                "price": null,
                "currency": "EUR",
                "note": null,
                "added": 1511669037,
                "owned": 1,
                "source":
                {
                    "type": "s",
                    "id": "steam"
                }
            }]
        }
    ]
}
```

Notes:
* Unused fields may be omitted or set to null
* If you want to use `source` (determining where the user bought the copy), you should always set `type` to `s`

### Custom Values

Some of the fields may contain [custom values](https://isthereanydeal.com/settings/collection/lists/), not defined by ITAD by default.
When custom value doesn't exist, it will be created for the user during import.
Currently you can set custom value for game's status, copy's status,
copy's type and copy's source.

### Send user to Collection import form [POST]
 
+ Request

    The body of the request will simulate form submission. It has to contain two fields: `file` and `upload`.
    The content of `file` is `base64` encoded JSON string, the content of `upload` field is not important.

    The URL for request is `https://isthereanydeal.com/collection/import/`

    Example of minimal POST data from example:

    + Body
    
            file=ew0KICAgICJ2ZXJzaW9uIjogIjAyIiwNCiAgICAiZGF0YSI6IFt7DQogICAgICAgICJ0aXRsZSI6ICJPeHlnZW4gTm90IEluY2x1ZGVkIiwNCiAgICAgICAgImNvcGllcyI6IFsNCiAgICAgICAgICAgIHsNCiAgICAgICAgICAgICAgICAidHlwZSI6ICJzdGVhbSINCiAgICAgICAgICAgIH0NCiAgICAgICAgXQ0KICAgIH1dDQp9&upload=Import+Collection


## Direct Import [/v01/collection/import/{?access_token}]
```Version: v01 | Type: protected | Scope: coll_write wait_write```

Alternative to import via form is to directly import data via API. This doesn't give user
as much control over the import but will be simpler. Use the flow that fits your app the best.

> Please note that this endpoint requires `wait_write` scope. This is because user may
> have turned on option to remove games from their Waitlist when it is added to Collection 

### Import Collection [POST]

+ Parameters

    + access_token (required) - OAuth access token

+ Request

    Data format for import is exactly the same as for import via Form,
    **the only difference is that body of the request is JSON string that is NOT `base64` encoded**.
    
    + Body
           
            {
                "version": "02",
                "data": [
                    {
                        "plain": "0rbitalis",
                        "title": "0RBITALIS",
                        "group": null,
                        "note": null,
                        "status": "notplayed",
                        "user_tags": [],
                        "playtime": 0,
                        "copies": [
                        {
                            "type": "steam",
                            "platforms": ["windows", "mac"],
                            "status": "redeemed",
                            "price": null,
                            "currency": "EUR",
                            "note": null,
                            "added": 1506896003,
                            "owned": 0,
                            "source": null
                        },
                        {
                            "type": "Groupees.com",
                            "platforms": [],
                            "status": "redeemed",
                            "price": null,
                            "currency": "EUR",
                            "note": null,
                            "added": 1511669037,
                            "owned": 1,
                            "source":
                            {
                                "type": "s",
                                "id": "steam"
                            }
                        }]
                    }
                ]
            }
        
+ Response 200

    Response will contain number of *copies* that were added in `.meta` field.
    It is possible for request to be successful and still import 0 copies.
    If copy of the same type already exists in user's Collection it is ignored during import

    + Body
    
            {
                ".meta": {
                    "copies": 2
                }
            }

# Group Custom Profiles

IsThereAnyDeal supports linking [3rd party profiles](https://isthereanydeal.com/settings/profiles/) to provide
a way to sync in remote wishlists and libraries into user's Waitlist and Collection.
In addition to built-in support for covered stores any 3rd party can create custom profile by creating
a publicly accessible profile description.
User will then be able to link this profile by [adding URL](https://isthereanydeal.com/settings/profiles/) of profile
description into IsThereAnyDeal. 

## Profile Description

Profile description has to be located at the publicly accessible URL ending with `/isthereanydeal.profile.json`,
not including query string. For example `https://example.com/isthereanydeal.profile.json?id=1234` is a valid URL.

The content is a JSON string with a following format:

```json
{
  "profile": {
    "id": "userid0123456789",
    "name": "UserName",
    "waitlist": {
      "public": "https:\/\/example.com\/profile\/waitlist\/",
      "source": "https:\/\/example.com\/profile\/waitlist.json"
    },
    "collection": {
      "public": "https:\/\/example.com\/custom\/collection\/",
      "source": "https:\/\/example.com\/custom\/collection.json"
    }
  }
}
```

`id` and `name` fields are **required**. `id` is user's (or profile's) ID and once linked can't be changed.
If `id` changes user will still be able to sync in from last known URLs but won't be able to refresh profile
to update user name or data URLs.  

`name` is user's name and serves for easier identification of the profile on IsThereAnyDeal.

## Syncing Waitlist

Waitlist sync is available if profile description contains `profile.waitlist.source` URL which links to export for Waitlist. 
`profile.waitlist.public` field is optional and may contain publicly accessible URL of this profile.

### Format

Format of Waitlist sync file is very simple and only lists games' names:

```json
{
  "version": "01",
  "data": [
    {"title": "DOOM"},
    {"title": "Cuphead"},
    {"title": "Turmoil"},
    {"title": "Songbringer"},
    {"title": "Immortal Redneck"},
    {"title": "Wizard of Legend"},
    {"title": "At Sundown"}
  ]
}
``` 

## Syncing Collection

Collection sync is available if profile description contains `profile.collection.source` URL which links to export for Collection. 
`profile.collection.public` field is optional and may contain publicly accessible URL of this profile.

### Format

Collection sync allows you to specify *status*, *playtime* and *platforms* of imported copies. `id`, `title` and `type` fields
are **required**.

`id` may contain any string and serves for identifying copy so we can track it in case there are multiple copies
of same type. You should try to prevent `id` changes, since ITAD would remove the copy and add a new one.

> Since Collection on IsThereAnyDeal determines games and their copies, sync is working with individual copies.
> This means that you can import for example both "Steam" and "DRM Free" copy for a single game.

```json
{
  "version": "01",
  "collection": [
    {
      "id": "40701",
      "title": "0RBITALIS",
      "type": "steam",
      "status": "redeemed",
      "playtime": 0,
      "platforms": ["windows", "mac", "linux"]
    },
    {
      "id": "40702",
      "title": "Oxygen Not Included",
      "type": "Klei Account",
      "status": "Custom Status",
      "playtime": 900,
      "platforms": ["windows", "mac", "linux"]
    }
  ]
}
``` 

You can use custom values for `type` and `status` or use default ones already created by ITAD.

Default type:
* any store id
* `drmfree`
* `retail`
* `other`

Default status:
* `redeemed`
* `extra`
* `giveaway`
* `trade`
* empty string for unknown/not set

## Link Profile [/v01/profile/link/{?access_token,url}]
```Version: v01 | Type: protected | Scope: profile_link```

If you don't want user to link profile manually, you can link it for him with this request.

> Please note that profile description has to be publicly available anyway due to ability to refresh profile

### Link Remote Profile [GET]

+ Parameters
    + access_token (required) - OAuth access token
    
+ Request 
    + Body

            url=https://example.com/isthereanydeal.profile.json?id=1234
        
+ Response 200
        
        {
            ".meta": {
                "linked": true
            }
        }

# Group Stats

## Waitlist Price Limits [/v01/stats/waitlist/price/{?key,plain,scale,bucket,region,currency}]
```Version: v01 | Type: private (request access)```

Get statistical info about Waitlist notification *price limits* for specific game, including:
* total number of occurences in Waitlists (if user has a game in different categories, all instances are used)
* number of occurences where user did not specify price limit 
* average notification limit
* notification limit at specified percentiles
* number of occurences divided into price buckets
 
### Get Waitlist Price Limit Stats [GET]

+ Parameters
    + key (required) - API key
    + plain (required)
    + scale: 25 (int, optional) - Scale determining what percentiles will be computed. Only values that divide 100 are valid. 
    + bucket: 5 (int, optional) - Size of price buckets,
    + region (optional) - If supplied, only Waitlists for this region will be used
    + currency (optional) - If supplied, all prices will be converted into this currency.
                            If not, and region is used, the region currency will be used.
                            Otherwise prices will be shown in USD.
    
+ Response 200
        
        {
            ".meta": {
                "currency": "USD"
            },
            "data": {
                "count": 10935,
                "anyprice": 6895,
                "average": 14.52,
                "percentiles": {
                    "0": 0.01,
                    "10": 6.45,
                    "20": 9.99,
                    "30": 9.99,
                    "40": 10.1,
                    "50": 11.57,
                    "60": 12.03,
                    "70": 14.92,
                    "80": 17.76,
                    "90": 27,
                    "100": 180
                },
                "buckets": {
                    "0": 165,
                    "5": 1119,
                    "10": 1649,
                    "15": 502,
                    "20": 148,
                    "25": 117,
                    "30": 93,
                    "35": 70,
                    "40": 84,
                    "45": 26,
                    "50": 12,
                    "55": 25,
                    "60": 7,
                    "65": 2,
                    "70": 17,
                    "75": 1,
                    "90": 2,
                    "180": 1
                }
            }
        }



## Waitlist Cut Limits [/v01/stats/waitlist/cut/{?key,plain,bucket,region}]
```Version: v01 | Type: private (request access)```

Get statistical info about Waitlist notification *price cut limits* for specific game:
* total number of occurences in Waitlists (if user has a game in different categories, all instances are used)
* average notification limit
* number of occurences divided into buckets
 
### Get Waitlist Price Cut Limit Stats [GET]

+ Parameters
    + key (required) - API key
    + plain (required)
    + bucket: 5 (int, optional) - Size of price buckets,
    + region (optional) - If supplied, only Waitlists for this region will be used
    
+ Response 200
        
        {
          "data": {
            "count": 13669,
            "average": 10,
            "buckets": {
              "0": 11267,
              "5": 6,
              "10": 38,
              "15": 13,
              "20": 60,
              "25": 101,
              "30": 91,
              "35": 17,
              "40": 83,
              "45": 24,
              "50": 704,
              "55": 13,
              "60": 181,
              "65": 115,
              "70": 191,
              "75": 530,
              "80": 138,
              "85": 32,
              "90": 47,
              "95": 9,
              "100": 9
            }
          }
        }



## Waitlist Chart [/v01/stats/waitlist/chart/{?key,offset,limit}]
```Version: v01 | Type: protected```

Returns Waitlist chart, games sorted from most Waitlisted to least Waitlisted.
 
### Get Waitlist Chart [GET]

+ Parameters
    + key (required) - API key
    + offset: 0 (int, optional) - Position at which to start
    + limit: 20 (int, optional) - Number of games to return (max 500)
    
+ Response 200
        
            {
              ".meta": {
                "range": [
                  0,
                  5
                ]
              },
              "data": [
                {
                  "position": 1,
                  "title": "Cyberpunk 2077",
                  "plain": "cyberpunkii0viivii",
                  "count": 21707
                },
                {
                  "position": 2,
                  "title": "NieR: Automata",
                  "plain": "nierautomata",
                  "count": 21085
                },
                ...
              ]
            }



## Collection Chart [/v01/stats/collection/chart/{?key,offset,limit}]
```Version: v01 | Type: protected```

Returns Collection chart, games sorted from most collected to least collected.
 
### Get Collection Chart [GET]

+ Parameters
    + key (required) - API key
    + offset: 0 (int, optional) - Position at which to start
    + limit: 20 (int, optional) - Number of games to return (max 500)
    
+ Response 200
        
            {
              ".meta": {
                "range": [
                  0,
                  20
                ]
              },
              "data": [
                {
                  "position": 1,
                  "title": "Portal 2",
                  "plain": "portalii",
                  "count": 43151
                },
                {
                  "position": 2,
                  "title": "Portal",
                  "plain": "portal",
                  "count": 41817
                },
                ...
              ]
            }



## Popularity Chart [/v01/stats/popularity/chart/{?key,offset,limit}]
```Version: v01 | Type: protected```

Returns popularity chart, games sorted from most popular to least popular, where popularity is
computed as normalized count in Waitlist + normalized count in Collection,
and rank is set based on the position in sorted list.
 
### Get Popularity Chart [GET]

+ Parameters
    + key (required) - API key
    + offset: 0 (int, optional) - Position at which to start
    + limit: 20 (int, optional) - Number of games to return (max 500)
    
+ Response 200
        
            {
              ".meta": {
                "range": [
                  0,
                  20
                ]
              },
              "data": [
                {
                  "position": 1,
                  "title": "NieR: Automata",
                  "plain": "nierautomata",
                  "rank": 1
                },
                {
                  "position": 2,
                  "title": "Stardew Valley",
                  "plain": "stardewvalley",
                  "rank": 2
                },
                ...
              ]
            }
