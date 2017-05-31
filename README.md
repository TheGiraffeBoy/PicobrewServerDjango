# DjangoPicoServer
This is a Django version of picobrew-server based largely on the work of  https://github.com/hotzenklotz/picobrew-server


# HTTP API
The PicoBrew's built-in Ardunio uses an unencrypted HTTP communication protocol. All request are `GET` requests and are not authenticated. The following documentation is based on Firmware 1.18.

[Complete API Docs](https://github.com/hotzenklotz/picobrew-server/wiki/PicoBrew-API)

# PicoBrew HTTP API
The PicoBrew's built-in Ardunio uses an unencrypted HTTP communication protocol. All request are `GET` requests and are not authenticated. The following documentation is based on Firmware 1.18.

## Get all user recipes
This request retrieves all the user recipes from the server.

##### GET
`http://www.picobrew.com/API/SyncUser?user=3ccfxxxxxxxxxxxxxxxxxxxxxxxxxa39&machine=5xxxxxxx0000`


###### PARAMETERS
- `user`: Some user id. Haven't seen it on the Picobrew.com website yet.
- `machine` : The "Zymatic ID" number. This can be found in your Picobrew.com profile.

##### RESPONSE
All recipes are concatenated as one string seperated by the pipe symbol "|".

```
#Motueka Dark Ale/8f085361e36643ea89bfefd9d08bf60f/Heat to Temp,102,0,0,0/Dough In,102,20,1,8/Heat to Mash,152,0,0,0/Mash 1,152,30,1,8/Mash 2,154,60,1,8/Heat to MO,175,0,0,0/Mash Out,175,10,1,10/Heat to Boil,207,0,0,0/Boil Adjunct 1,207,30,2,0/Boil Adjunct 2,207,20,3,0/Boil Adjunct 3,207,10,4,5/Connect Chiller,0,0,6,0/Chill,65,10,0,10/|Pico Pale Ale/b7f69fad72494b99a1978fff6c0f9bf0/Heat Water,152,0,0,0/Mash,152,90,1,8/Heat to Boil,207,0,0,0/Boil Adjunct 1,207,45,2,0/Boil Adjunct 2,207,5,3,0/Boil Adjunct 3,207,5,4,0/Boil Adjunct 4,207,5,5,5/Connect Chiller,70,0,6,0/Chill,70,10,0,10/|Polaris Single Hop/1877ea832ea64b29860fc52454305203/Heat Water,152,0,0,0/Mash,152,90,1,8/Heat to Boil,207,0,0,0/Boil Adjunct 1,207,30,2,0/Boil Adjunct 2,207,25,3,0/Boil Adjunct 3,207,5,4,5/Connect Chiller,0,0,6,0/Chill,65,10,0,10/|#
```

##### RESPONSE SCHEMA
```
#RECIPE_NAME / HASH on Picobrew.com / STEP_NAME, TEMPERATURE, DURATION, COMPARTMENT , DRAIN / |#
```

## Get all cleaning/rinse recipes
This retrieves the cleaning / rinse recipes from the "Clean/Rinse" menu item. The recipe follow the same schema as above.

##### GET
`	http://www.picobrew.com/API/SyncUser?user=00000000000000000000000000000000&machine=5xxxxxxx0000`

##### RESPONSE
```
#Cleaning v1/7f489e3740f848519558c41a036fe2cb/Heat Water,152,0,0,0/Clean Mash,152,15,1,5/Heat to Temp,152,0,0,0/Adjunct,152,3,2,1/Adjunct,152,2,3,1/Adjunct,152,2,4,1/Adjunct,152,2,5,1/Heat to Temp,207,0,0,0/Clean Mash,207,10,1,0/Clean Mash,207,2,1,0/Clean Adjunct,207,2,2,0/Chill,120,10,0,2/|Rinse v3/0160275741134b148eff90acdd5e462f/Rinse,0,2,0,5/|#
```

The program steps contained in the recipe have the following structure:

```
/NAME, TEMPERATURE, TIME, LOCATION, DRAIN/

NAME = An arbitrary string
TEMPERATURE = in °F
TIME = Duration the program step in minutes
LOCATION = 0 : Between Compartments, 1 : Mash Compartment, 2 : Adjunct Compartment A, 3 : Adjunct B, 4 : Adjunct C, 5 : Adjunct C, 6 : Chiller(?)
DRAIN = Duration in minutes to run the pump and drain
```

###### PARAMETERS
- `user` user id
- `machine` "Zymatic" id

## Re-Sync Recipes
Issued when using the "re-sync recipe" option. Sometimes triggered by the machine itself.
##### GET
`	http://www.picobrew.com/API/checksync?user=3ccfxxxxxxxxxxxxxxxxxxxxxxxxxa`

#### RESPONSE
I have only ever seen this one response. Not sure what the request is really for.
```
"\r\n#!#"
```


## Start new Brew Session
This starts a new brew session for your brew log on the PicoBrew Website. The returned id can be used to subsequently send detailed session log entry.

##### GET
`http://www.picobrew.com/API/logSession?user=00000000000000000000000000000000&recipe=7f489e3740f848519558c41a036fe2cb&code=0&machine=5xxxxxxx0000&firm=1.1.8
`

###### PARAMETERS
- `user` user id for the official PicoBrew user
- `recipe` recipe id
- `machine` "Zymatic" id
- `firm` firmware version

##### RESPONSE
Returns a new session id

```
#4406c30c181a4f918ff3f091d64d84a3#
```


## Log temperature reading for session
After starting a new brew session the machine constantly sends feedback to Picobrew.com. This allows them to draw the pretty session graph.

##### GET
`	http://www.picobrew.com/API/logsession?session=88c7011fc51e4666b9c984311f934e07&code=1&data=Heat%20Water&state=0`

`	http://www.picobrew.com/API/LogSession?session=88c7011fc51e4666b9c984311f934e07&data=2%2F67|1%2F75|3%2F74|4%2F76&code=2&step=0/520264/520081/520081/0/0/0/1&state=0`

`http://www.picobrew.com/API/LogSession?session=857b4b3f39a74044bbaa258c6f58dc91&data=2%2F142|1%2F151|3%2F81|4%2F146&code=2&step=0/4368868/3707806/3707806/0/0/0/1&state=2`

###### PARAMETERS
- `session` Session id from the above request
- `data` Temperature reading like `2%2F67|1%2F75|3%2F74|4%2F76` or the name of the current step of the brew process like `Heat%20Water`
- `state` ???
- `step` The current state of the machine. Critical to correctly resuming a session.
- `code` 1 : Log new program step, 2 : Temperature data for current step, 3 : End Session

The temperature readings have the following structure:
```
Heat Loop °F | Wort °F | Board °F | Heat Loop 2 °F
```

##### RESPONSE
	<empty>



## Resume a Session #1
If a brew sessions is ended prematurely one can resume it from the PicoBrew's help menu. This a two-step request. 1. Request the recipe and then 2. set the correct machine parameters.

##### GET
`	http://www.picobrew.com/API/recoversession?session=857b4b3f39a74044bbaa258c6f58dc91&code=0`

###### PARAMETERS
- `session` Session id
- `code` "0" for the first part of the request

##### RESPONSE
```
#Cleaning v1/7f489e3740f848519558c41a036fe2cb/Heat Water,152,0,0,0/Clean Mash,152,15,1,5/Heat to Temp,152,0,0,0/Adjunct,152,3,2,1/Adjunct,152,2,3,1/Adjunct,152,2,4,1/Adjunct,152,2,5,1/Heat to Temp,207,0,0,0/Clean Mash,207,10,1,0/Clean Mash,207,2,1,0/Clean Adjunct,207,2,2,0/Chill,120,10,0,2/|!#
```


## Resume a Session #2
Returns the same machine parameters as were saved during the temperature logging request.

##### GET
`http://www.picobrew.com/API/recoversession?session=857b4b3f39a74044bbaa258c6f58dc91&code=1`

###### PARAMETERS

- `session` Session id
- `code` "1" for the second part of the request

##### RESPONSE
```
#0/10757813/3707806/3707806/0/0/0/1#
```


# Installation

- Install Python 3.6.1 and its requirements

```
// Install Python requirements
pip install -r requirements.txt

```

- Connect the PicoBrew machine to your computer and enable DNS spoofing. Re-route `www.picobrew.com` to your computer.
[More Details](https://github.com/hotzenklotz/picobrew-server/wiki/Install)

# Features
- Import BeerXML files
- Send all your recipes to the PicoBrew
- Send cleaning recipes to the PicoBrew
- Session Logging
- Session Recovery
- Admin Web UI



# Disclaimer
This software is provided "as is" and any expressed or implied warranties are disclaimed. This software submits recipes with temperature targets to your PicoBrew machine and will cause it to heat water. Any damage to your PicoBrew machine is at your own risk.

# License

MIT @ Ryan Burton