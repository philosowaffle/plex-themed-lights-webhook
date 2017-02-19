# plex-lifx-webhook
A webhook for Plex that changes the color of your LIFX lights to match the main colors of the poster art being played.

## Requirements
- [LIFX](http://www.lifx.com/) lights already installed and set up
- Install [Node.js](https://nodejs.org/en/)
- Clone Repository
- Currently need to use [virtualenv](http://docs.python-guide.org/en/latest/dev/virtualenvs/) for dependency management, see [Issue 1](https://github.com/philosowaffle/plex-lifx-webhook/issues/1)
- Navigate to directory and start virtualenv: `venv\Scripts\activate`

## Usage
- [Configure variables](#Configuration)
- `python plexlifx.py`

## How it works
This webhook runs a local webserver using [Flask](http://flask.pocoo.org/) which recieves requests from Plex when certain media events occur for categories `movie` and `episode`.  Based on filter criteria that can be set in the config file the webhook decides whether or not to generate an effect.  If this event matches our criteria then it grabs the thumbnail that came along with the request and uses [color-thief-py](https://github.com/fengsp/color-thief-py) to generate a color palette.  Then, using the api wrapper provided by [PIFX](https://github.com/cydrobolt/pifx), sets your LIFX lights to match the color palette.

Boilerplate code heavily based on [plex2hue-relay](https://github.com/lotooo/plex2hue-relay).

## Key Features
- When playing a `movie` or `episode` sets lights to match the color palette of the media thumbnail.
- When media is paused or stopped, lights are restored to a default "pause" theme.
- When media is resumed, lights return to the media color palette
- See the Configuration section for more information about customizing the experience.

## Configuration
Configuration is done in the `config.ini` file found at the root level.

| Configuration     | Default | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | Required | Example                                             |
|-------------------|---------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|-----------------------------------------------------|
| LogFile           |         | The file where logs will be written.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | Yes      | plex_lifx_webhook.log                               |
| FlaskPort         | 5000    | The port that flask will run on.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | No       |                                                     |
| IgnorePlayerUUIDs | none    | Specify a comma separated list of player UUIDs which will be ignored by the webhook.  Any requests sent by one of the players listed here will not trigger any events.  Comma separated, no spaces                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | No       |                                                     |
| LocalPlayerOnly   | true    | Specifies whether or not events should only be triggered for local players or all players.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | No       |                                                     |
| APIKey            |         | Your LIFX Api Key.  Go [here](https://cloud.lifx.com/settings) to get one for your account.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | Yes      |                                                     |
| Brightness        | .35     | The brightness level the lights will be dimmed to when a movie/show starts.  Any number between 0.0-1.0 where 1.0 represents 100% brightness.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | No       |                                                     |
| Duration          | 2s      | Determines how quickly the lights will transition to the them once the movie/show starts playing.  Specify any whole number in seconds.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | No       |                                                     |
| NumColors         | 4       | Determines how many colors will be chosen from the color palette.  If you specify more colors than you have light bulbs then you will be defaulted to the number of bulbs available.  For example, if you have 8 bulbs but specify 10 colors, then only 8 colors will be chosen.  If you have 8 bulbs but specify 4 colors, then 4 colors will be chosen and you will have pairs of bulbs with the same color.  This will depend on the order that bulbs are specified in the config file.  If you do not specify the bulbs explicitly then the colors will be applied in random order.                                                                             | No       |                                                     |
| ColorQuality      | 1       | How precise the color selection will be.  Can choose any whole number between 1 and 10, where 1 is the lowest quality.  Note that increasing quality will increase the time spent calculating colors and may be slower.                                                                                                                                                                                                                                                                                                                                                                                                                                             | No       |                                                     |
| DefaultPauseTheme |         | The name of the default theme that should be restored when media pauses or stops.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | Yes      | Basic                                               |
| DefaultPlayTheme  |         | The name of the default theme that should be used should the webhook fail to generate a color palette.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | Yes      | Movie Blue                                          |
| Lights            |         | Comma separated list of your lightbulb names.  Any lights excluded in this list will be ignored and left untouched by the webhook.  The order you specify the bulbs in will effect how colors get applied.  For example, assume we have a color palette of (red, blue, yellow, green), and we have specified lights "a,b,c,d,e,f,g,h,i", the the colors will be applied like so.  Lights a, b, and c will be red.  Lights d and e will be blue.  Lights f and g will be yellow.  And lights h and i will be green.  If you choose not to specify any lights then the webhook will apply the effects to all lights and will randomize which lights get which colors. | No       | Corner Lamp,Kitchen Lamp 1,Standing Lamp2,Sofa Lamp |



