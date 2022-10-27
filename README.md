# Flow by //42 - Documentation

Scripting in Flow is fairly straightforward. Flow's scripting engine taps into the sim's javascript confines to give you access to data and functions you normally wouldn't have access to outside of JS. We've built our own internal APIs to make it easy for anyone with basic knowledge of javascript to create scripts.

While there is a code editor in sim, we **strongly recommend** using a full external code editor (like [Visual Studio Code](https://code.visualstudio.com/)) to write your code. You can easily copy back-forth between the Flow script editor and your external code editor. 

MSFS has a tendency to reload panels at random times and might lead you to loose some unsaved changes if you code directly in the sim.


# Base script
Since this is javascript, anything outside of the following default functions will execute as soon as it is loaded. In flow, this occurs 20 seconds after the UI is loaded or when the user triggers execution of the script.

## ``run()``
*"run"* will execute when the user clicks the tile or uses the Concierge via Flow.
 - Returning a duration (in ms) will delay the closure of the flow wheel by the specified duration. 
 - Returning ``true`` leaves flow open.
 - Returning ``false`` closes flow immediately.
 - Omitting the return will default the close duration to 500ms.
```js
run(() => {
  return 1500; 
}
```

## ``exit()``
*"exit"* will execute before the script is reloaded or deleted. For the good of everyone, please be diligent and clear your loops, intervals and timeouts here.

```js
exit(() => {
  // your exit code here
}
```

## ``state()``
*"state"* will automatically execute every 500ms and updates the content of the script's tile. MDI icons are supported here (e.g. mdi:airplane). A full list of icons is available here: https://pictogrammers.github.io/@mdi/font/6.9.96/
```js
state(() => {
  return 'mdi:lightbulb-on:Overlayed Text';
}
```

## ``info()``
*"info"* will automatically execute every 500ms and updates the content shown in the center circle when the tile is hovered. MDI icons are not supported here.
```js
state(() => {
  return 'Engine on';
}
```

## ``twitch_message(message)``
If the Twitch integration is enabled by the user in the Flow settings, chat messages will come through *"twitch_message"*. If the user specified a command via the script settings, this will only execute if the command is found within the message. Otherwise, every message will come through.
```js
twitch_message((message) => {
  let example_message = {
    "tags": {
      "badge-info": {
        "founder": "51"
      },
      "badges": {
        "moderator": "1",
        "founder": "0",
        "bits": "1000"
      },
      "color": "#65C47F",
      "display-name": "runshotgun",
      "emotes": null,
      "first-msg": "0",
      "id": "9dcf58b7-e6a8-484d-9c16-280f2d384c4f",
      "mod": "1",
      "returning-chatter": "0",
      "room-id": "91420958",
      "subscriber": "1",
      "tmi-sent-ts": "1666907326935",
      "turbo": "0",
      "user-id": "84610723",
      "user-type": "mod"
    },
    "source": {
      "nick": "runshotgun",
      "host": "runshotgun@runshotgun.tmi.twitch.tv"
    },
    "command": {
      "command": "PRIVMSG",
      "channel": "#theskyloungetv"
    },
    "parameters": "Looking good!"
  }
}
```
Note that the tags structure is where the magic happens. Tags are defined or not depending on the type of message that went through. Here are a few examples:
```js
// Bits
"bits": "10"

// Resub
"msg-id": "resub",
"msg-param-cumulative-months": "8",
"msg-param-months": "0",
"msg-param-multimonth-duration": "0",
"msg-param-multimonth-tenure": "0",
"msg-param-should-share-streak": "0",
"msg-param-sub-plan-name": "Channel\\sSubscription\\s(oohcando)",
"msg-param-sub-plan": "Prime",
"msg-param-was-gifted": "false",
"subscriber": "1",
"system-msg": "runshotgun\\ssubscribed\\swith\\sPrime.\\sThey've\\ssubscribed\\sfor\\s8\\smonths!"

// Highlighted message
"msg-id": "highlighted-message"
```
Fore more info on the Twitch integration, please refer to the official api documentation here: https://dev.twitch.tv/docs/irc/example-bot


# APIs

## Toolbar
Get the list of panels available in the sim's toolbar:
```js 
this.$api.toolbar.get_panels()
```
Get info on a specific in-game panel. The ID can be acquired using ``get_panels``:
```js 
this.$api.toolbar.get_panel(panel_id)
```
Open an in-game panel. The ID can be acquired using ``get_panels``:
```js 
this.$api.toolbar.open_panel(panel_id)
```
Close an in-game panel. The ID can be acquired using ``get_panels``:
```js 
this.$api.toolbar.close_panel(panel_id)
```

## DataStore
You can persist data (Object or Array structure) to be used in future sessions:
```js 
this.$api.datastore.export(data)
```
You can restore data that was persisted in previous sessions:
```js 
this.$api.datastore.import()
```

## Community
To get available multiplayer servers:
```js 
this.$api.community.get_servers()
```
To get the user's current multiplayer server:
```js 
this.$api.community.get_server()
```
To check if the user is currently connected to multiplayer or not:
```js 
this.$api.community.is_online()
```
To change the user's current multiplayer server:
```js 
this.$api.community.set_server(server_name)
```

## Airports
To get airport data for a specific ICAO:
```js 
this.$api.airports.find_airport_by_icao(icao, callback)
// data returns via the callback method
```
To get airports within a radius of a specified Lat/Lon:
```js 
this.$api.airports.find_airports_by_coords(uid, lon, lat, radius, limit, callback_added, callback_removed, callback_failed, init)
// data returns via the 3 callback methods
```
``uid``: A reusable unique identifier  
``lon, lat``: Coordinates  
``radius``: Radius in meters for the search  
``limit``: Limit the amount of airports to return. Farthest airports are eliminated first.  
``callback_added(Array)``: When doing a second search for a different coordinate, new airports will be returned here.  
``callback_removed(Object)``: When doing a second search for a different coordinate, removed airports will be returned here.  
``callback_failed(Object)``: If the search failed, the error will return here.  
``init``: Return all airports in the *callback_added* callback (instead of only the changes).  

## Weather
To get all available weather presets:
```js 
this.$api.weather.get_presets()
```
To set a specific preset:
```js 
this.$api.weather.set_preset(preset_index)
```
To get the METAR of a specific station:
```js 
this.$api.weather.find_metar_by_icao(icao, callback)
// data returns via the callback method
```
To get the METAR of the closest station:
```js 
this.$api.weather.find_metar_from_coords(lat, lon, callback)
// data returns via the callback method
```

## Time
To set time from a string:
```js 
this.$api.time.set_time_by_moment(moment)
// data returns via the callback method
```
``live``: Set to realtime  
``solarNoon``: The sun-time for the solar noon (sun is in the highest position)  
``nadir``: The sun-time for nadir (darkest moment of the night, sun is in the lowest position)  
``goldenHourDawnStart``: The sun-time for morning golden hour (soft light, best time for photography)  
``goldenHourDawnEnd``: The sun-time for morning golden hour (soft light, best time for photography)  
``goldenHourDuskStart``: The sun-time for evening golden hour starts  
``goldenHourDuskEnd``: The sun-time for evening golden hour starts  
``sunriseStart``: The sun-time for sunrise starts (top edge of the sun appears on the horizon)  
``sunriseEnd``: The sun-time for sunrise ends (bottom edge of the sun touches the horizon)  
``sunsetStart``: The sun-time for sunset starts (bottom edge of the sun touches the horizon)  
``sunsetEnd``: The sun-time for sunset ends (sun disappears below the horizon, evening civil twilight starts)  
``blueHourDawnStart``: The sun-time for blue Hour start (time for special photography photos starts)  
``blueHourDawnEnd``: The sun-time for blue Hour end (time for special photography photos end)  
``blueHourDuskStart``: The sun-time for blue Hour start (time for special photography photos starts)  
``blueHourDuskEnd``: The sun-time for blue Hour end (time for special photography photos end)  
``civilDawn``: The sun-time for dawn (morning nautical twilight ends, morning civil twilight starts)  
``civilDusk``: The sun-time for dusk (evening nautical twilight starts)  
``nauticalDawn``: The sun-time for nautical dawn (morning nautical twilight starts)  
``nauticalDusk``: The sun-time for nautical dusk end (evening astronomical twilight starts)  
``amateurDawn``: The sun-time for amateur astronomical dawn (sun at 12° before sunrise)  
``amateurDusk``: The sun-time for amateur astronomical dusk (sun at 12° after sunrise)  
``astronomicalDawn``: The sun-time for night ends (morning astronomical twilight starts)  
``astronomicalDusk``: The sun-time for night starts (dark enough for astronomical observations)  
\
\
To set the time of day (seconds from midnight local):
```js 
this.$api.time.set_local_time(local_time_of_day)
```
To set the time of day (seconds from midnight zulu):
```js 
this.$api.time.set_zulu_time(zulu_time_of_day)
```

## Twitch
To get if the Twitch integration is connected or not:
```js 
this.$api.twitch.is_connected()
```
To send a message in Twitch chat:
```js 
this.$api.twitch.send_message(message, prefix)
```
``message``: String message to send  
``prefix``: (Optional) Will prefix the message with the given string data. To reply to a user for example, extract its ID from the tags structure and set this as your prefix: ``@reply-parent-msg-id=${message.tags['id']}`` 


# Debugging
To debug your code in-sim, you can simply add !!! before your console message to display it in the debug window in-sim. Only .log, .warn .error .info are supported.
```js 
console.log('!!!Script has exited');
```
Alternatively, you can navigate to http://127.0.0.1:19999 in your browser or use the Coherent GT Debugger executable that comes with the MSFS SDK and select **//42 Flow**.
