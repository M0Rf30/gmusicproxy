# GMusicProxy – Google Play Music Proxy

*"Let's stream Google Play Music using any media-player"*

 - [About](#about)
   - [Features](#features)
 - [Setup](#setup)
   - [On Arch Linux](#on-arch-linux)
   - [Requirements](#requirements)
   - [Installation](#installation)
 - [Usage](#usage)
   - [Command-line](#command-line)
   - [URL-based interface](#url-based-interface)
   - [Examples of integration](#examples-of-integration)
     - [MPD](#mpd)
     - [VLC](#vlc)
   - [Related projects](#related-projects)
 - [Support](#support)
   - [Issues](#issues)
 - [Contributors](#contributors)
 - [Changelog](#changelog)
 - [License](#license)

## About
This program permits the use of Google Play Music with All Access subscription with any music player that is able to stream MP3 files and to manage M3U playlists (e.g., MPD server, VLC, ...). It can work also with a free account without All Access extras.

This project is not supported nor endorsed by Google.

### Features
- create persistent URLs to all the tracks, albums and stations available on the Google Play Music + All Access platform
- get access to all the songs in your collection, playlists and registered stations
- search by name any artist, album or song
- request a transient (it will be not registered in your account) station based on any search (a.k.a. "Instant Mix")
- stream any songs as standard MP3 complete of IDv3 tag with all the information and album image

## Setup

### On Arch Linux
just type:
```
<your AUR helper> -S gmusicproxy
```
and follow the instructions

### Requirements
- a Google Play Music account with All Access subscription (some functionalities continue to work even with a free account)
- a Python 3 interpreter
- some python libs: *gmusicapi*, *netifaces*, *eyed3*, *python-daemon*

### Installation

- The right way to use an under-development python project makes use of `virtualenv` utility:
  - install the proxy once:

    ```bash
    sudo apt-get install python3-virtualenv virtualenv
    git clone https://github.com/M0Rf30/gmusicproxy.git
    cd gmusicproxy
    virtualenv .
    pip install -r requirements.txt
    source bin/activate

    ```
  - launch the proxy when you need it:

    ```bash
    ./GMusicProxy
    ```
  - if you need to upgrade the proxy and its dependencies:
    - use the option `--upgrade` on the `pip` installation command (e.g., `pip install --upgrade -r requirements`), or
    - Exit from the virtualenv using `deactivate`.

## Usage
With the service running on a computer on the LAN, it can be used by any others of the same network.

To launch the proxy you need the credentials of your Google account: *email* and *password*. If you are using the 2-factor authentication, you have to create an application-specific password to be used with this program. Another usefull information would be the device ID of an Android/iOS device registered in your account: you can discover it using the option `--list-devices` on the command-line. As default a fake-id, based on the mac address of the main network card of the server running the service, is used.

You can provide such necessary information, as well as other options, on the command-line of the program or using a configuration file.

### Command-line
Here a list of the supported options on the command-line:

- `--host`: host in the generated URLs [default: autodetected local ip address]
- `--bind-address`: ip address to bind to [default: 0.0.0.0=all]
- `--port`: default TCP port to use [default: 9999]
- `--oauth`: generate OAuth2 credentials (opens browser)
- `--disable-all-access`: disable All Access functionalities
- `--debug`: enable debug messages
- `--log`: log file
- `--daemon`: daemonize the program
- `--extended-m3u`: enable non-standard extended m3u headers
- `--shoutcast-metadata`: enable Shoutcast metadata protocol support (disabling IDv3 tags)
- `--disable-playcount-increment`: disable the automatic increment of playcounts upon song fetch

Before starting the proxy for the first time, run `GMusicProxy --oauth` to generate OAuth2 credentials. Follow the prompts, and paste the code into your terminal. *This requires All Access.*

### URL-based interface
The only way to use the service is to query the proxy by means of properly formatted HTTP requests over the configured TCP port. Such URLs can be used directly in music programs or in scripts or in any browser. A URL looks like this: `http://host:port/command?param_1=value&param_2=value`. I don't apply any validation to the submitted values: please, be nice with the proxy and don't exploit it! :)

Consider that any song, album, artist, playlist or station got a unique ID in Google Music API but there are many methods to discover them.

Here a list of the supported requests (with some restricted by the availability of a All Access subscription):

- `/get_collection`: reports an M3U playlist with all the songs in your personal collection; the resulting list can be shuffled and/or filtered using the rating; note that not all the rated (liked) songs belong to your collection.
  Allowed parameters:
     - `shuffle`: if the collection has to be shuffled [default: no]
     - `rating`: an integer value (typically between 1-5) to filter out low rated or unrated songs form your collection
- `/get_promoted`: reports an M3U playlist of "promoted tracks"; the resulting list can be shuffled; this is, more or less, your Thumb's Up playlist, kinda.
  Allowed parameters:
     - `shuffle`: if the playlist has to be shuffled [default: no]
- `/search_id`: reports the unique ID as result of a search for an artist, a song or an album.
  Allowed parameters:
     - `type`: search for `artist`, `album` or `song` [required]
     - `title`: a string to search in the title of the album or of the song
     - `artist`: a string to search in the name of the artist in any kind of search
     - `exact`: a `yes` implies an exact match between the query parameters `artist` and `title` and the real data of the artist/album/song [default: `yes`]
- `/get_by_search`: makes a search for artist/album/song as `/search_id` and returns the related content (an M3U list for the album or for the top songs of an artist and the MP3 file for a song); it is also possible to get the full list of matches reported by Google Music using search with `type=matches` [requires A.A.].
  Allowed parameters:
     - `type`: search for `artist`, `album`, `song` or `matches` [required]
     - `title`: a string to search in the title of the album or of the song
     - `artist`: a string to search in the name of the artist in any kind of search
     - `exact`: a `yes` implies an exact match between the query parameters `artist` and `title` and the real data of the artist/album/song; it doesn't make sense with a search for `matches` [default: `no`]
     - `num_tracks`: the number of top songs to return in a search for artist [default: 20]
- `/get_all_stations`: reports a list of registered stations as M3U playlist (with URLs to other M3U playlist) or as plain-text list (with one station per line)  [requires A.A.].
  Allowed parameters:
     - `format`: `m3u` for an M3U list or `text` for a plain-text list with lines like `Name of the Station|URL to an M3U playlist` [default: `m3u`]
     - `separator`: a separator for the plain-text lists [default: `|`]
     - `only_url`: a `yes` creates a list of just URLs in the plain-text lists (the name of the station is totally omitted) [default: `no`]
     - `exact`: a `yes` implies an exact match between the query parameters `artist` and `title` and the real data of the artist/album/song [default: `no`]
- `/get_all_playlists`: reports the playlists registered in the account as M3U playlist (with URLs to other M3U playlist) or as plain-text list (with one playlist per line).
  The allowed parameters are the same as `/get_all_stations`.
- `/get_new_station_by_search`: reports as M3U playlist the content of a new (transient or permanent) station created on the result of a search for artist/album/song (a.k.a. "Instant Mix") [requires A.A.].
  Allowed parameters:
     - `type`: search for `artist`, `album` or `song` [required]
     - `title`: a string to search in the title of the album or of the song
     - `artist`: a string to search in the name of the artist in any kind of search
     - `exact`: a `yes` implies an exact match between the query parameters `artist` and `title` and the real data of the artist/album/song [default: `no`]
     - `num_tracks`: the number of songs to extract from the new station [default: 20]
     - `transient`: a `no` creates a persistent station that will be registered into the account [default: `yes`]
     - `name`: the name of the persistent station to create [required if `transient` is `no`]
- `/get_new_station_by_id`: reports as M3U playlist the content of a new (transient or permanent) station created on a specified id of an artist/album/song [requires A.A.].
  Allowed parameters:
     - `id`: the unique identifier of the artist/album/song [required]
     - `type`: the type of id specified among `artist`, `album` and `song` [required]
     - `num_tracks`: the number of songs to extract from the new station [default: 20]
     - `transient`: a `no` creates a persistent station that will be registered into the account [default: `yes`]
     - `name`: the name of the persistent station to create [required if `transient` is `no`]
- `/get_station`: reports an M3U playlist of tracks associated to the given station  [requires A.A.].
  Allowed parameters:
     - `id`: the unique identifier of the station [required]
     - `num_tracks`: the number of tracks to extract [default: 20]
- `/get_ifl_station`: reports an M3U playlist of tracks associated to the automatic "I'm feeling lucky" station  [requires A.A.].
  Allowed parameters:
     - `num_tracks`: the number of tracks to extract [default: 20]
- `/get_listen_now`: reports Listen Now stations and albums as M3U playlist (with URLs to other M3U playlist) or as plain-text list (with one playlist per line).
  Allowed parameters:
     - `type`: search for `artist` or `album` [required]
     - `format`: `m3u` for an M3U list or `text` for a plain-text list with lines like `Name of Station|URL to an M3U playlist` [default: `m3u`]
     - `separator`: a separator for the plain-text lists [default: `|`]
     - `only_url`: a `yes` creates a list of just URLs in the plain-text lists (the name of the album is totally omitted) [default: `no`]
- `/get_situations`: reports Listen Now Situations (curated playlists) as M3U playlist (with URLs to other M3U playlist) or as plain-text list (with one playlist per line).
  Allowed parameters:
     - `format`: `m3u` for an M3U list or `text` for a plain-text list with lines like `Name of Playlist|URL to an M3U playlist` [default: `m3u`]
     - `separator`: a separator for the plain-text lists [default: `|`]
     - `only_url`: a `yes` creates a list of just URLs in the plain-text lists (the name of the album is totally omitted) [default: `no`]
- `/get_playlist`: reports the content of a registered playlist in the M3U format; the list can be also shuffled.
  Allowed parameters:
     - `id`: the unique identifier of the playlist [required]
     - `shuffle`: if the list has to be shuffled [default: no]
- `/get_album`: reports the content of an album as an M3U playlist.
  Allowed parameters:
     - `id`: the unique identifier of the album [required]
- `/get_song`: streams the content of the specified song as a standard MP3 file with IDv3 tag.
  Allowed parameters:
     - `id`: the unique identifier of the song [required]
     - `quality`: the quality of the song stream
        Allowed values are: `low`, `med` and `hi` [default: `hi`]
- `/get_top_tracks_artist`: reports an M3U playlist with the top songs of a specified artist [requires A.A.].
  Allowed parameters:
     - `id`: the unique identifier of the artist [required]
     - `type`: the type of id specified among `artist`, `album` and `song` [required]
     - `num_tracks`: the number of top songs to return [default: 20]
- `/get_discography_artist`: reports the list of available albums of a specified artist as M3U playlist (with URLs to other M3U playlist) or as plain-text list (with one album per line)  [requires A.A.].
  Allowed parameters:
     - `id`: the unique identifier of the artist [required]
     - `format`: `m3u` for an M3U list or `text` for a plain-text list with lines like `Name of Album|Year|URL to an M3U playlist` [default: `m3u`]
     - `separator`: a separator for the plain-text lists [default: `|`]
     - `only_url`: a `yes` creates a list of just URLs in the plain-text lists (the name of the album is totally omitted) [default: `no`]
- `/like_song`: reports a positive rating on the song with specified id.
  Allowed parameters:
     - `id`: the unique identifier of the song [required]
- `/dislike_song`: reports a negative rating on the song with specified id.
  Allowed parameters:
     - `id`: the unique identifier of the song [required]

### Examples of integration
#### MPD
- You can copy any M3U list generated by the proxy in the playlists registered inside MPD. MPD usually keeps the playlists inside the folder specified by `playlist_directory` in its configuration file `mpd.conf`.

  ```bash
  curl -s 'http://localhost:9999/get_by_search?type=album&artist=Queen&title=Greatest%20Hits' >
    /var/lib/mpd/playlists/queen.m3u
  mpc load queen
  mpc play
  ```

- You can also request a fresh list of songs from a station and add them to the current playlist.

  ```bash
  mpc clear
  curl -s 'http://localhost:9999/get_new_station_by_search?type=artist&artist=Queen&num_tracks=100' |
    grep -v ^# | while read url; do mpc add "$url"; done
  mpc play
  ```

#### VLC
- You can listen any generated playlist using VLC from command-line.

  ```bash
  vlc 'http://localhost:9999/get_by_search?type=album&artist=Rolling%20Stones&title=tattoo&exact=no'
  ```
- You can automatically choose at random one registered station.

  ```bash
  curl -s 'http://localhost:9999/get_all_stations?format=text&only_url=yes' | sort -R | head -n1 | vlc -
  ```

### Related projects
- Original idea: [Mario Di Raimondo](https://github.com/gmusicproxy/gmusicproxy)
- Simon's Unofficial-Google-Music-API (the great backend used by gmusicproxy): https://github.com/simon-weber/gmusicapi
- web2py-mpd-gmproxy (a web interface that uses gmusicproxy as backend): https://github.com/matclab/web2py-mpd-gmproxy
- GMusic-MPD (an helper script for GMusicProxy together with MPD): https://github.com/Illyism/GMusic-MPD
- gmproxy-scripts (helper scripts for working with GMusicProxy): https://github.com/kmac/gmproxy-scripts
- gpmplay (a bash script to easily search with GMusicproxy): https://github.com/onespaceman/gmpplay
- g-music (Emacs client for gmusicproxy and mpd): https://github.com/bodicsek/g-music
- GMusicProxyGui (a C# GUI for the GMusicProxy): https://github.com/Poket-Jony/GMusicProxyGui
- PlayFetch (a full-featured helper written in Python): https://github.com/jeffmhubbard/playfetch

## Support
### Issues
Feel free to open bug reports (complete of verbose output produced with options `--debug` and `--log`) on GitHub, to fork the project and to make pull requests for your contributions.

## [Contributors](CONTRIBUTORS.md)
## [Changelog](CHANGELOG.md)
## [License](LICENSE.md)

