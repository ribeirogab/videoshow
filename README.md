# videoshow [![Build Status](https://api.travis-ci.org/h2non/videoshow.svg?branch=master)][travis] [![Code Climate](https://codeclimate.com/github/h2non/videoshow/badges/gpa.svg)](https://codeclimate.com/github/h2non/videoshow) [![NPM](https://img.shields.io/npm/v/videoshow.svg)][npm] [![Dependency Status](https://gemnasium.com/h2non/videoshow.svg)][gemnasium]

<img src="https://github.com/h2non/videoshow/blob/master/test/fixtures/norris.gif" width="180" align="right" />

Simple utility for **node/io.js** to **create straightforward video slideshows based on images** using [ffmpeg](http://ffmpeg.org)

You can easily **create videos** with optional **audio**, **subtitles** and **fade in/out transitions**.

To getting started you can take a look to the [examples](https://github.com/h2non/videoshow/tree/master/examples), [programmatic API](#api) and [command-line](#command-line-interface) interface usage

[Here](https://www.youtube.com/watch?v=hob1kxPNXg0) you can see a real video created using videoshow

## Requirements

- **[ffmpeg](http://ffmpeg.org)** with additional compilation flags `--enable-libass --enable-libmp3lame`

## Install

```bash
npm install videoshow
```

For command-line usage, install it as global package:
```bash
npm install -g videoshow
```

## Usage

```js
var videoshow = require('videoshow')

var images = [
  'step1.jpg',
  'step2.jpg',
  'step3.jpg',
  'step4.jpg'
]

var videoOptions = {
  fps: 25,
  loop: 5, // seconds
  transition: true,
  transitionDuration: 1, // seconds
  videoBitrate: 1024,
  videoCodec: 'libx264',
  size: '640x?',
  audioBitrate: '128k',
  audioChannels: 2,
  format: 'mp4'
}

videoshow(images, videoOptions)
  .audio('song.mp3')
  .save('video.mp4')
  .on('start', function (command) {
    console.log('ffmpeg process started:', command)
  })
  .on('error', function (err, stdout, stderr) {
    console.error('Error:', err)
    console.error('ffmpeg stderr:', stderr)
  })
  .on('end', function (output) {
    console.error('Video created in:', output)
  })
```

Take a look to the [programmatic API](#api) for more details

## Command-line interface

```bash
$ videoshow --help
```

```bash
Create video slides easily from images
Usage: bin/videoshow [options]

Options:
  --help, -h       Show help
  --config, -c     File path to JSON config file [required]
  --audio, -a      Optional audio file path
  --subtitles, -s  Path to .srt subtitles file
  --input, -i      Add additional input to video
  --output, -o     Output video file path
  --size, -x       Video size resolution
  --logo, -l       Path to logo image
  --debug, -d      Enable debug mode in error case

Examples:
  bin/videoshow -c config.json video.mp4
  bin/videoshow -c config.json --audio song.mp3 video.mp4
  bin/videoshow -c config.json --audio song.mp3 --logo logo.png video.mp4
```

Example `config.json` file:

```json
{
  "output": "video.mp4",
  "options": {
    "fps": 25,
    "loop": 5,
    "transition": true,
    "transitionDuration": 1,
    "videoBitrate": 1024,
    "videoCodec": "libx264",
    "size": "640x?",
    "audioBitrate": "128k",
    "audioChannels": 2,
    "format": "mp4"
  },
  "images": [
    "./test/fixtures/step_1.png",
    "./test/fixtures/step_2.png",
    "./test/fixtures/step_3.png",
    "./test/fixtures/step_4.png",
    "./test/fixtures/step_5.png"
  ]
}
```

## API

### videoshow(images, [ options ])
Return: `Videoshow`

Videoshow constructor. You should pass an `array<string>` or `array<object>` or `array<ReadableStream>` with the desired images,
and optionally passing the video render `options` object per each image.

Image formats supported are: `jpg`, `png`, `gif`, `bmp`

```js
videoshow([ 'image1.jpg', 'image2.jpg', 'image3.jpg'])
  .save('video.mp4')
  .on('error', function () {})
  .on('end', function () {})
```

`images` param could be a collection as well:
```js
videoshow([{
    path: 'image1.jpg',
    caption: 'Hello world as video subtitle'
  }, {
    path: 'image2.jpg',
    caption: 'This is a sample subtitle',
    loop: 10 // long caption
  }])
  .save('video.mp4')
  .on('error', function () {})
  .on('end', function () {})
```

#### Video options

You can define any of the methods allowed by [fluent-ffmpeg][ffmpeg-api]

Default options are:
```js
{
  fps: 25,
  loop: 5, // seconds
  transition: true,
  transitionDuration: 1,
  videoBitrate: 1024,
  videoCodec: 'libx264',
  size: '640x?',
  audioBitrate: '128k',
  audioChannels: 2,
  format: 'mp4'
}
```

#### Image options

- **path** `string` - File path to image
- **loop** `number` - Image slide duration in **seconds**. Default to `5`
- **transition** `boolean` - Enable fade in/out transition for the current image
- **transitionDuration** `number` - Fade in/out transition duration in **seconds**. Default to `0.5`
- **transitionColor** `string` - Fade in/out transition background color. Default to `black`. See [supported colors][ffmpeg-colors]
- **disableFadeOut** `boolean` - If transition is enable, disable the fade out. Default `false`
- **disableFadeIn** `boolean` - If transition is enable, disable the fade in. Default `false`
- **caption** `string` - Caption text as subtitle. It allows a limited set of HTML tags. See [Subrip][subrip]
- **captionStart** `number` - Miliseconds to start the caption. Default to `1000`
- **captionEnd** `number` - Miliseconds to remove the caption. Default to `loop - 1000`
- **logo** `string` - Path to logo image. See `logo()` method

#### videoshow#image(image)

Push an image to the video. You can pass an `string` as path to the image,
or a plain `object` with [image options](#supported-image-options)

#### videoshow#audio(path [, params ])

Define the audio file path to use.
It supports multiple formats and codecs such as `acc`, `mp3` or `ogg`

**Supported params**:

- **delay** `number` - Delay audio start in seconds. Default to `0` seconds
- **fade** `boolean` - Enable audio fade in/out effect. Default `true`

#### videoshow#logo(path [, params ])

Add a custom image as logo in the left-upper corner.
It must be a `png` or `jpeg` image

**Supported params**:

- **start** `number` - Video second to show the logo. Default `5` seconds
- **end** `number` - Video second to remove  the logo. Default `totalLength - 5` seconds
- **xAxis** `number` - Logo `x` axis position. Default `10`
- **yAxis** `number` - Logo `y` axis position. Default `10`

#### videoshow#subtitles(path)

Define the [SubRip subtitles][subrip]
file path to load. It should be a `.srt` file

If you want to use [SubStation Alpha](http://en.wikipedia.org/wiki/SubStation_Alpha) (SSA/ASS) subtitles,
you should pass it as video input:

```js
videoshow(images)
  .input('subtitles.ass')
  .save('video.mp4')
```

#### videoshow#save(path)
Return: `EventEmitter` Alias: `render`

Render and write in disk the resultant video in the given path

Supported events for subscription:

- **start** `cmd` - Fired when ffmpeg process started
- **error** `error` - Fired when transcoding error success
- **progress** `data` - Fired with transcoding progress information
- **codecData** `codec` - Fired when input codec data is available
- **end** `videoPath` - Fired when the process finish successfully

For more information, see the [ffmpeg docs](https://github.com/fluent-ffmpeg/node-fluent-ffmpeg#setting-event-handlers)

#### videoshow#input(input)

Add input file to video. By default you don't need to call this method

#### videoshow#filter(filter)

Add a custom video filter to the video. See the [docs](https://github.com/fluent-ffmpeg/node-fluent-ffmpeg#videofiltersfilter-add-custom-video-filters)

#### videoshow#complexFilter(filter)

Add a custom complex filter to the video. See the [docs](https://github.com/fluent-ffmpeg/node-fluent-ffmpeg#complexfilterfilters-map-set-complex-filtergraph)

#### videoshow#loop(seconds)

Default image loop time in seconds. Default to `5`

#### videoshow#size(resolution)

Video size resolution. Default to `640x?`.
See the [docs](https://github.com/fluent-ffmpeg/node-fluent-ffmpeg#sizesize-set-output-frame-size)

#### videoshow#aspect(aspect)

Video aspect ration. Default autocalculated from video `size` param.
See the [docs](https://github.com/fluent-ffmpeg/node-fluent-ffmpeg#aspectaspect-set-output-frame-aspect-ratio)

#### videoshow#options(options)
Alias: `flags`

Add a set of video output options as command-line flag.
`options` argument should be an array.
See the [docs](https://github.com/fluent-ffmpeg/node-fluent-ffmpeg#outputoptionsoption-add-custom-output-options)

#### videoshow#option(argument)
Alias: `flag`

Add a custom output option as command-line flag to pass to `ffmpeg`

#### videoshow#options(arguments)
Alias: `flags`

Add multiple output options as command-line flags to pass to `ffmpeg`.
The argument must be an `array`

### videoshow.VERSION
Type: `string`

Current package semantic version

### videoshow.ffmpeg
Type: `function`

[fluent-ffmpeg](https://github.com/fluent-ffmpeg/node-fluent-ffmpeg) API constructor

## License

[MIT](http://opensource.org/licenses/MIT) © Tomas Aparicio

[travis]: http://travis-ci.org/h2non/videoshow
[gemnasium]: https://gemnasium.com/h2non/videoshow
[npm]: http://npmjs.org/package/videoshow
[subrip]: http://en.wikipedia.org/wiki/SubRip#SubRip_text_file_format
[ffmpeg-api]: https://github.com/fluent-ffmpeg/node-fluent-ffmpeg#creating-an-ffmpeg-command
[ffmpeg-colors]: https://www.ffmpeg.org/ffmpeg-utils.html#Color
