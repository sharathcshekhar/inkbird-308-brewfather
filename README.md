# inkbird-308-brewfather

This is a small project to get Inkbird ITC-308 WiFi temperature data on Brewfather app. Here, I will describe how you can get your 
fermentation data or keezer temperature, from Inkbird thermostat, directly synced to Brewfather app (you will need a paid version of Brewfather).

# Intro
Inkbird has WiFi capabilities, but it does not play along well with many smart home apps. Inkbird's own app added integration to Google assistant,
which was promising. 
The temperature data from InkBird gets displayed on my Google Home App, but I could not find a way to take it further e.g., export this data to Brewfather.

To get this working, we need two compatible apps - one which understands data from inkbird, and another which plays well with popular IoT
and smart home softwares. 

I came across some posts (which I am not able to recollect and hence unable to link here) where users could successfully pair Inkbird with Tuya Smart app
(turns out Inkbird uses Tuya chip). This was great news because Tuya works well with `https://www.home-assistant.io/`, and home-assistant is hackable.

# Inkbird to Tuya

The first step is to install the Tuya Smart app and pair it with ITC-308. Install the Tuya App, reset the Inkbird device, and pair it with the Tuya App as a
`T&H > Temperature and humidity Sensor (WiFi)`
After this, you can read the values from Inkbird on Tuya. Next, create a cloud account with Tuya so your data is synced to cloud. 

# Running home-assistant

Next, you have to run home-assistant server. If you have a always-on laptop or a desktop, you can get a docker image
and start it: https://www.home-assistant.io/docs/installation/docker/. The other options are using a Raspberry Pi, or a AWS instance to run home-assistant.
To start a docker instance:
    docker run -d --name="home-assistant" -v ~/home-assistant/config:/config -v /etc/localtime:/etc/localtime:ro --net=host homeassistant/home-assistant:stable

To verify, point your browser to `http://localhost:8123/`. This should load the home-assistant console.

# Linking home-assistant and Tuya
1. Navigate to `http://localhost:8123/config/integrations`, and link your Tuya account by entering your credentials.
2. You should now be able to see some data from Inkbird in your home-assistant console.

The data home-assistant receives from InkBird has many other information (such as delay, calibration, etc.). You can get all the attributes here:
    http://localhost:8123/developer-tools/state

To monitor the fermentation/keezer temperature, you need only two of these values. `temperature` which is the set-temperature and `current_temperature` 
which is the current temperature.

However, for some reason the data home-assistant receives from Tuya looks like it is off by a factor of  approximately 10 (not exactly though).
This is probably due to some strange way of converting Fahrenheit/Celsius (e.g., double conversion at some point).

To get back the original temperatures, this is what I found works for me when InkBird is configured to output values in F.

    actual_set_temp in F = (temp_in_home_assistat - 32) / 18
    actual_current_temp in F = (temp_in_home_assistat + 288) / 10

# Pushing data to Brefather
1. To push the data from home-assistant to Brewfather, create a REST service. See `configuration.yaml`.
2. Place this file in the `config` directory of home-assistant.
3. Turn on the custom streams on Brewfather - the app will generate a custom URL.
4. Copy the custom URL generated in the previous step and replace the url on line 16 in `configuration.yaml`.

# Create Sync automation script
The REST service now has to be called through an automated service within home-assistant.
I have chosen a time-based automation script which runs every 15 mins.
See `automation.yml`. But home-assistant is flexible enough to make a trigger as per your preference.
This script also includes the steps to correct the temperature using the formula described earlier.

The automation script needs to know the device id of Inkbird. You can get that from `http://localhost:8123/config/entities`.
Grab the device-id which looks something like `climate.123456789011` and put it in place of `device-id` in lines 10 and 12.

That's it. Data from Inkbird will now be synced to Brewfather.

# Summary
1. Install Tuya on a smartphone and pair it with InkBird.
2. Run home-assistant server on your desktop, Raspberry Pi, AWS or wherever.
3. Link Tuya and home-assistant (from home-assistant). This is as straightforward as entering your Tuya username and password inside home-assistant
4. Enable custom streams on brewfather and add a hook to home assistant based on the configurations in the repo.
